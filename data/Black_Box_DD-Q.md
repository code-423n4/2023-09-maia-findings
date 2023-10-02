## No duplicate check while adding tokens to the strategyTokens

***Permalink:*** https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L362

In the BranchPort.sol, the function `addStrategyToken()` lacks duplicate check while adding token into strategyTokens Array

Duplicate token Addresses can pushed multiple times to the `strategyTokens` Array.

## Recommendation:
Add Duplicate check if token Address already added to the Array.
```solidity
 if(isStrategyToken[_token]) revert ("AlreadyAddedStrategyToken");
```

```solidity
    function addStrategyToken(address _token, uint256 _minimumReservesRatio) external override requiresCoreRouter {
        if (_minimumReservesRatio >= DIVISIONER || _minimumReservesRatio < MIN_RESERVE_RATIO) {
            revert InvalidMinimumReservesRatio();
        }

    +   if(isStrategyToken[_token]) revert AlreadyAddedStrategyToken();
        strategyTokens.push(_token);
        getMinimumTokenReserveRatio[_token] = _minimumReservesRatio;
        isStrategyToken[_token] = true;

        emit StrategyTokenAdded(_token, _minimumReservesRatio);
    }
```

