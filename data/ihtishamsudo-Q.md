## [L-01] VirtualAccount#withdrawNative should use ```forceSafeTransferETH``` for DoS Protection 

#### Proof of Concept 
It is stated in ```safeTransferLib``` by solady : 
``` 
/// @dev Note:
/// - For ETH transfers, please use `forceSafeTransferETH` for DoS protection.
```
But this recommendation is not implemented in ```VirtualAccount#withdrawNative```.
``` solidity 
function withdrawNative(uint256 _amount) external override requiresApprovedCaller {
        msg.sender.safeTransferETH(_amount);    //@audit-issue not used forceSafeTransferEth for DoS Protection
    }
```
#### Recommended Mitigation 
```forceSafeTransferEth``` should be used instead of safeTransferEth for native tokens.
Mitigated Code : 
``` solidity 
function withdrawNative(uint256 _amount) external override requiresApprovedCaller {
        -msg.sender.safeTransferETH(_amount); 
        +msg.sender.forceSafeTransferETH(_amount);
    }
```
#### Relevant Github Link 
[VirtualAccount.sol#L52](https://github.com/code-423n4/2023-09-maia/blob/c0dc3550e0754571b82d7bfd8f0282ac8fa5e42f/src/VirtualAccount.sol#L52)

## [NC-01] BranchPort#_checkTimeLimit lastmanage could be simplified 
#### Proof of Concept 
```solidity 
function _checkTimeLimit(address _token, uint256 _amount) internal {
        uint256 dailyLimit = strategyDailyLimitRemaining[msg.sender][_token];
        if (block.timestamp - lastManaged[msg.sender][_token] >= 1 days) {
            dailyLimit = strategyDailyLimitAmount[msg.sender][_token];
            unchecked {
                lastManaged[msg.sender][_token] = (block.timestamp / 1 days) * 1 days;
            }
        }
        strategyDailyLimitRemaining[msg.sender][_token] = dailyLimit - _amount;
    }
```
Above function has lastManaged = block.timestamp which is dividing and multiplying by 1 days. Dividing by 1 day and then Multiplying by 1 day does not have any effect on the value, as it cancels out and leaves the timestamp unchanged. So, this line of code could be simplified to :
``` lastManaged[msg.sender][_token] = block.timestamp; ```
  