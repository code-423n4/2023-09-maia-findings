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
## Impact:
The impact of this issue is that it can lead to unnecessary transaction failures and user inconvenience. When a user attempts to bridge tokens with a valid deposit, the transaction will fail due to this unnecessary condition, even though the deposit is legitimate. This can result in confusion and frustration for users of the smart contract.
## Recommended Mitigation Steps:
To address this issue, the unnecessary condition `if (_deposit > 0)` should be removed from the code. Here's the corrected code snippet:
```solidity
if (_underlyingAddress == address(0)) {
    revert UnrecognizedUnderlyingAddress();
}
```
## D. Duplicate Strategy Tokens in addStrategyToken Function:
[Link](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L362-L372)
The `addStrategyToken` function is designed to add a strategy token to a list called `strategyTokens`. However, it lacks a check to prevent the addition of duplicate tokens. Without this check, the same token address can be added multiple times to the list, leading to inefficiencies and potential issues when interacting with the contract.
```solidity
  /// @inheritdoc IBranchPort
    function addStrategyToken(address _token, uint256 _minimumReservesRatio) external override requiresCoreRouter {
        if (_minimumReservesRatio >= DIVISIONER || _minimumReservesRatio < MIN_RESERVE_RATIO) {
            revert InvalidMinimumReservesRatio();
        }

        strategyTokens.push(_token);
        getMinimumTokenReserveRatio[_token] = _minimumReservesRatio;
        isStrategyToken[_token] = true;

        emit StrategyTokenAdded(_token, _minimumReservesRatio);// @audit-ok duplicate stratergy tokens additions
    }
```
## Impact:
The impact of allowing duplicate strategy tokens in the strategyTokens list is that it can lead to inefficient use of resources and potential confusion when querying or interacting with the contract. It can also disrupt the intended functionality of the contract if it relies on distinct strategy tokens.
## Mitigation:
```solidity
/// @inheritdoc IBranchPort
function addStrategyToken(address _token, uint256 _minimumReservesRatio) external override requiresCoreRouter {
    if (_minimumReservesRatio >= DIVISIONER || _minimumReservesRatio < MIN_RESERVE_RATIO) {
        revert InvalidMinimumReservesRatio();
    }

    // Check if the token is already in the list
    for (uint256 i = 0; i < strategyTokens.length; i++) {
        if (strategyTokens[i] == _token) {
            revert DuplicateStrategyToken();
        }
    }

    strategyTokens.push(_token);
    getMinimumTokenReserveRatio[_token] = _minimumReservesRatio;
    isStrategyToken[_token] = true;

    emit StrategyTokenAdded(_token, _minimumReservesRatio);
}

```
## E. Precision Issue in Daily Limit Calculation Function:
[Link](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L485-L494)
In the `_checkTimeLimit` function, there is a code snippet that calculates the start of the current day based on the current timestamp. However, the calculation is prone to a loss of precision due to the order of operations. Here is the code snippet in question:
```solidity
lastManaged[msg.sender][_token] = (block.timestamp / 1 days) * 1 days;
```
The issue arises because of the division operation (`/`) before multiplication (`*`). This code divides the `block.timestamp` by the number of seconds in a day (86400 seconds) to obtain the number of days since the Unix epoch, effectively rounding down to the nearest day. Then, it multiplies the result by `1 days` to calculate the timestamp at the start of the current day. However, since division occurs before multiplication, any fractional part of the division is discarded. This means that if a management operation occurs at any time during a day, the `lastManaged` timestamp will be set to the start of that same day, causing a loss of precision in tracking the daily limit window.
## Impact:
The lack of precision in the daily limit calculation can result in the incorrect enforcement of daily limits for Port Strategy management operations. In scenarios where multiple management operations occur within the same day, the strategy may inadvertently allow more operations than intended, potentially leading to overuse of resources or other unintended consequences.
## Mitigation:
To address the lack of precision and ensure accurate daily limit enforcement, the code should use integer division to round down to the nearest day when calculating the start of the current day. Here is the corrected code snippet:
```solidity
lastManaged[msg.sender][_token] = (block.timestamp / 1 days) * 1 days;
```