## A. Duplicate New Bridge Agents
[Link](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/ArbitrumCoreBranchRouter.sol#L126-L143)
This code creates a new Bridge Agent every time it is called, without checking if a Bridge Agent with the same address already exists. Consequently, it is possible to add duplicate Bridge Agents to the system.
## Impact:
The impact of allowing duplicate Bridge Agents is that it can lead to confusion and unintended behavior within the system. It may also result in increased gas costs due to the unnecessary creation of duplicate Bridge Agents.
## Mitigation:
To mitigate this issue, consider implementing logic in the `_receiveAddBridgeAgent` function that checks whether a Bridge Agent with the same address already exists before creating a new one. 
## B. Array Length Mismatch
[Link](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L318-L344)
In the `redeemSettlement` function, there is a loop that iterates over elements in the `settlement.hTokens` and `settlement.amounts` arrays without explicitly checking if the arrays have the same length. Here's the relevant part of the code:
```solidity
for (uint256 i = 0; i < settlement.hTokens.length;) {
    // Save to memory
    address _hToken = settlement.hTokens[i];

    // ...

    unchecked {
        ++i;
    }
}
```
The issue arises if `settlement.hTokens` and `settlement.amounts` have different lengths. If the lengths do not match:

- If `settlement.hTokens` is longer than `settlement.amounts`, the loop will access elements in `settlement.amounts` beyond its bounds, leading to potential out-of-bounds errors.
- If `settlement.amounts` is longer than `settlement.hTokens`, the loop may not process all elements in `settlement.amounts`, potentially leaving some elements unprocessed.
## Impact:
The impact of this issue could range from unexpected runtime errors, such as "index out of bounds," to more subtle issues where the function may not perform as intended due to missing or extra iterations. This can affect the correctness and reliability of the settlement redemption process.
## Mitigation:
```solidity
if (settlement.hTokens.length != settlement.amounts.length) {
    revert("Array length mismatch");
}
```
## C. Users will experience a transaction failure when bridging tokens with a valid deposit
[Link](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L1131-L1146)
The `_updateStateOnBridgeOut` function is designed to facilitate the bridging of tokens from the root omnichain environment to a branch chain. It performs several checks and transfers tokens accordingly. However, a specific section of the code has an issue:
```solidity
if (_underlyingAddress == address(0)) {
    if (_deposit > 0) {
        revert UnrecognizedUnderlyingAddress();
    }
}

```
The vulnerability lies in the inner `if (_deposit > 0)` condition. This condition should not be present here because it causes a revert when `_deposit` is greater than zero, regardless of whether the `_underlyingAddress` is valid or not. This behavior is not aligned with the intended purpose of the code, where a valid deposit should not trigger a revert.
## Impact
The impact of this issue is that it can lead to unnecessary transaction failures and user inconvenience. When a user attempts to bridge tokens with a valid deposit, the transaction will fail due to this unnecessary condition, even though the deposit is legitimate. This can result in confusion and frustration for users of the smart contract.
## Recommended Mitigation Steps
To address this issue, the unnecessary condition `if (_deposit > 0)` should be removed from the code. Here's the corrected code snippet:
```solidity
if (_underlyingAddress == address(0)) {
    revert UnrecognizedUnderlyingAddress();
}
```