# 1) Unnecessary computation in the daily management limit function
Instance:
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L485

The function:
```
function _checkTimeLimit(address _token, uint256 _amount) internal {
    uint256 dailyLimit = strategyDailyLimitRemaining[msg.sender][_token];
    if (block.timestamp - lastManaged[msg.sender][_token] >= 1 days) {
        dailyLimit = strategyDailyLimitAmount[msg.sender][_token];
        unchecked {
@>          lastManaged[msg.sender][_token] = (block.timestamp / 1 days) * 1 days;
        }
    }
    strategyDailyLimitRemaining[msg.sender][_token] = dailyLimit - _amount;
}
```
Division by 1, them multiplication by 1 adds nothing to the function, other than unnecessary gas consumption.


Recommended mitigation steps:
Consider changing this function to:
```
function _checkTimeLimit(address _token, uint256 _amount) internal {
    uint256 dailyLimit = strategyDailyLimitRemaining[msg.sender][_token];
    if (block.timestamp - lastManaged[msg.sender][_token] >= 1 days) {
        dailyLimit = strategyDailyLimitAmount[msg.sender][_token];
        unchecked {
-           lastManaged[msg.sender][_token] = (block.timestamp / 1 days) * 1 days;
+           lastManaged[msg.sender][_token] = block.timestamp;
        }
    }
    strategyDailyLimitRemaining[msg.sender][_token] = dailyLimit - _amount;
}
```
