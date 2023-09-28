## [Gas-01] Redundant `if` check can be removed in `addBridgeAgentFactory()` function

`BranchPort#addBridgeAgentFactory()` called by the `_toggleBranchBridgeAgentFactory()` in CoreBranchRouter.sol, the _toggleBranchBridgeAgentFactory() call disable bridgeAgentFactoryAddress if the address passes `IPort(_localPortAddress).isBridgeAgentFactory(_newBridgeAgentFactoryAddress)` check. And if it is not, the toggle fn performs addBridgeAgentFactory() call to enable/add the bridgeAgentFactoryAddress. 


The `Branchport#addBridgeAgentFactory()` call could not add existing bridgeAgentFactoryAddress, if it adds it revert with `AlreadyAddedBridgeAgentFactory()` error. However, this check is not required, bc `_toggleBranchBridgeAgentFactory()` only calling addBridgeAgentFactory() address when there is not existing bridgeAgentFactoryAddress. Avoiding this check can reduce the gas consumption.

```solidity
    function _toggleBranchBridgeAgentFactory(address _newBridgeAgentFactoryAddress) internal {
        // Save Port Address to memory
        address _localPortAddress = localPortAddress;

        // Check if BridgeAgentFactory is active
        if (IPort(_localPortAddress).isBridgeAgentFactory(_newBridgeAgentFactoryAddress)) {
            // If so, disable it.
            IPort(_localPortAddress).toggleBridgeAgentFactory(_newBridgeAgentFactoryAddress);
        } else {
            // If not, add it.
            IPort(_localPortAddress).addBridgeAgentFactory(_newBridgeAgentFactoryAddress);
        }
    }
```

```solidity
    function addBridgeAgentFactory(address _newBridgeAgentFactory) external override requiresCoreRouter {
        if (isBridgeAgentFactory[_newBridgeAgentFactory]) revert AlreadyAddedBridgeAgentFactory(); // <- REDUNDANT CHECK

        isBridgeAgentFactory[_newBridgeAgentFactory] = true;
        bridgeAgentFactories.push(_newBridgeAgentFactory);

        emit BridgeAgentFactoryAdded(_newBridgeAgentFactory);
    }
```

NOTE; This good practice is also followed by similar function e.g. BranchPort#addStrategyToken()