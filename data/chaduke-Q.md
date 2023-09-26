QA1. ``addStrategyToken()`` fails to check whether the strategyToken to be added has already been added. 

[https://github.com/code-423n4/2023-09-maia/blob/c0dc3550e0754571b82d7bfd8f0282ac8fa5e42f/src/BranchPort.sol#L362-L372](https://github.com/code-423n4/2023-09-maia/blob/c0dc3550e0754571b82d7bfd8f0282ac8fa5e42f/src/BranchPort.sol#L362-L372)

Mitigation: we need to check whether the strategyToken to be added has already been added. 

```diff
 function addStrategyToken(address _token, uint256 _minimumReservesRatio) external override requiresCoreRouter {

+       if(isStrategyToken[_token])  revert AlreadyAddedStrategyToken();

        if (_minimumReservesRatio >= DIVISIONER || _minimumReservesRatio < MIN_RESERVE_RATIO) {
            revert InvalidMinimumReservesRatio();
        }

        strategyTokens.push(_token);
        getMinimumTokenReserveRatio[_token] = _minimumReservesRatio;
        isStrategyToken[_token] = true;

        emit StrategyTokenAdded(_token, _minimumReservesRatio);
    }
```

QA2. ``updatePortStrategy()`` fails to check whether the input PortStrategy has been added or not:

[https://github.com/code-423n4/2023-09-maia/blob/c0dc3550e0754571b82d7bfd8f0282ac8fa5e42f/src/BranchPort.sol#L403-L411](https://github.com/code-423n4/2023-09-maia/blob/c0dc3550e0754571b82d7bfd8f0282ac8fa5e42f/src/BranchPort.sol#L403-L411)

Mitigation: we need to check whether the input PortStrategy has been added or not. 

