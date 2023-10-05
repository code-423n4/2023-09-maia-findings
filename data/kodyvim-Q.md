# QA Report
## _checkTimelimit should also round down `block.timestamp` to the nearest days
`checkTimeLimit` checks if a Port Strategy has reached its daily management limit
While `lastManaged` is rounded down to the nearest day then substracted from `block.timestamp`, `block.timestamp` is not rounded down to the nearest day.
This would results in a situation where the daily limit could be reset earlier than expected.
Round `block.timestamp` before substraction.
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L487
```solidity
function _checkTimeLimit(address _token, uint256 _amount) internal {
        uint256 dailyLimit = strategyDailyLimitRemaining[msg.sender][_token];
@>        if (block.timestamp - lastManaged[msg.sender][_token] >= 1 days) {//@audit round block.timestamp to the nearest day as well
            dailyLimit = strategyDailyLimitAmount[msg.sender][_token];
            unchecked {
@>              lastManaged[msg.sender][_token] = (block.timestamp / 1 days) * 1 days;
            }
        }
        strategyDailyLimitRemaining[msg.sender][_token] = dailyLimit - _amount;
    }
```

## Redundant implementation should be removed
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L434
```solidity
function redeemDeposit(uint32 _depositNonce) external override lock {
        // Get storage reference
        Deposit storage deposit = getDeposit[_depositNonce];

        // Check Deposit
        if (deposit.status == STATUS_SUCCESS) revert DepositRedeemUnavailable();
        if (deposit.owner == address(0)) revert DepositRedeemUnavailable();
        if (deposit.owner != msg.sender) revert NotDepositOwner();

        // Zero out owner
@>      deposit.owner = address(0);

        // Transfer token to depositor / user
        for (uint256 i = 0; i < deposit.tokens.length;) {
            _clearToken(msg.sender, deposit.hTokens[i], deposit.tokens[i], deposit.amounts[i], deposit.deposits[i]);

            unchecked {
                ++i;
            }
        }

        // Delete Failed Deposit Token Info
 @>       delete getDeposit[_depositNonce];
    }
```
* `deposit.owner` would be zero-ed out after deletion of the getDeposit mapping
## Allow users to specify token `recipient`
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchBridgeAgent.sol#L69
```solidity
function depositToPort(address underlyingAddress, uint256 amount) external payable lock {
        IArbPort(localPortAddress).depositToPort(msg.sender, msg.sender, underlyingAddress, amount);
    }
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchBridgeAgent.sol#L79
```
solidity
function withdrawFromPort(address localAddress, uint256 amount) external payable lock {
        IArbPort(localPortAddress).withdrawFromPort(msg.sender, msg.sender, localAddress, amount);
    }
```
hardcoding recipient as `msg.sender` may not be desirable in most cases for instance the current account might have been blocklisted by the receiving token.