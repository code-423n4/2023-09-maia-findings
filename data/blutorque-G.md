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


## [Gas-02] Re-writing storage slot with the same factory address
*File: BranchPort.sol*
```solidity
    address[] public bridgeAgentFactories;
```
The bridgeAgentFactories holds the bridge agent address added to the branch port, we only want to write this array when adding the new bridge agent. This could be done by `addBridgeAgentFactory()` call, its push that address to above array and marks the address as an active bridge agent factory.

The `addBridgeAgentFactory` is only called by the router `_toggleBranchBridgeAgentFactory()` function. The toggle function checks if the address marks as active bridge agent, if not it will add the bridge agent address via the `addBridgeAgentFactory`,

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/CoreBranchRouter.sol#L244-L256
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/CoreBranchRouter.sol#L284-L296

The problem here is if we are toggling a already deactivated bridge agent factory address, it will re-add the address to the same array. And because writing an array cost gas, we don't want the gas to be wasted adding the same element twice. 

So first, 
`_toggleBranchBridgeAgentFactory` adds `_newBridgeAgentFactoryAddress`, which push a new value to `bridgeAGentFactories[]`
Second, 
`_toggleBranchBridgeAgentFactory` get called to disable it as bridge agent address
Third, 
`_toggleBranchBridgeAgentFactory` get called again for same addres in 1st step, Hence wasted gas to write the same value in `bridgeAGentFactories[]`


Note: Similar pattern can be notice in `_manageStrategyToken()`

**Recommendation**
Durring the toggle off, the removal process of bridge agent factory address from an array doesn't going to change anything in term of gas usage (as more steps will be added). So I think its better to remove if/else check in `_toggleBranchBridgeAgentFactory()`.

**File: CoreBranchRouter.sol**
```solidity
    function _toggleBranchBridgeAgentFactory(address _newBridgeAgentFactoryAddress) internal {
        // Save Port Address to memory
        address _localPortAddress = localPortAddress;
        IPort(_localPortAddress).toggleBridgeAgentFactory(_newBridgeAgentFactoryAddress);
    }
```

