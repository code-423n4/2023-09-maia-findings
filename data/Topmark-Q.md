### Report 1:
Absence of comma "," in comment description causes misinterpretation of code from IRootBridgeAgent.sol. As corrected in the code snippet below 
"has not been executed yet, triggering" is different from "has not been executed, yet triggering", 
not having a comma makes it even more complicated as this comment description is relevant to this code description:  https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L711-L718 .
The comment Description: https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/IRootBridgeAgent.sol#L54 .
```solidity
54. --- *       0x08 | Call to `retrieveDeposit()´. (clears a deposit that has not been executed yet triggering `_fallback`)
54. +++ *       0x08 | Call to `retrieveDeposit()´. (clears a deposit that has not been executed yet, triggering `_fallback`)
```
###  Report 2:
A better way to write the two conditions from the code snippet below is to use the "||" operator, it makes it more cleaner, efficient and readable
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1060-L1061
```solidity
---        if (_globalAddresses.length != _amounts.length) revert InvalidInputParamsLength();
---        if (_amounts.length != _deposits.length) revert InvalidInputParamsLength();
+++        if (_globalAddresses.length != _amounts.length || _amounts.length != _deposits.length) revert InvalidInputParamsLength();
```