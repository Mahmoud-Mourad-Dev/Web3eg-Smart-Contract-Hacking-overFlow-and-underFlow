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
