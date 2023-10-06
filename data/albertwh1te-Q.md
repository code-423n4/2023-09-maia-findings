# Unused Arrays:

The arrays listed below are not utilized:

    BranchPort.strategyTokens
    BranchPort.portStrategies
    BranchPort.bridgeAgentFactories

# forget to update array when removed 
The `strategyTokens`are added via addStrategyToken. However, when toggleStrategyToken (removing or adding) is executed, strategyTokens are not updated.
```
    function addStrategyToken(address _token, uint256 _minimumReservesRatio) external override requiresCoreRouter {
        if (_minimumReservesRatio >= DIVISIONER || _minimumReservesRatio < MIN_RESERVE_RATIO) {
            revert InvalidMinimumReservesRatio();
        }

        strategyTokens.push(_token);
        // @info minimal reserve is set when add strategy token
        getMinimumTokenReserveRatio[_token] = _minimumReservesRatio;
        isStrategyToken[_token] = true;

        emit StrategyTokenAdded(_token, _minimumReservesRatio);
    }
    /// @inheritdoc IBranchPort
    function toggleStrategyToken(address _token) external override requiresCoreRouter {
        isStrategyToken[_token] = !isStrategyToken[_token];

        emit StrategyTokenToggled(_token);
    }
```
Same issue can be found at here
    BranchPort.portStrategies
    BranchPort.bridgeAgentFactories