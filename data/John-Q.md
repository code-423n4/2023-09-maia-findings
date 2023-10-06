## Invalid commnets

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L76C5-L77C86
```
    /// @notice Mapping from Underlying Address to isUnderlying (bool).
    mapping(address bridgeAgentFactory => bool isActive) public isBridgeAgentFactory;
```
    
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L79C5-L80C43
```
    /// @notice Bridge Agents deployed in root chain.
    address[] public bridgeAgentFactories;
```
## Unnecessary prameter in `toggleBranchBridgeAgentFactory` of the `CoreRootRouter` contract

The `toggleBranchBridgeAgentFactory` function in the `CoreRootRouter` contract contains an unnecessary parameter. 
To optimize the code, it is advisable to remove this extraneous parameter.
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L148-L177
```
/**
    * @notice Add or Remove a Branch Bridge Agent Factory.
    * @param _rootBridgeAgentFactory Address of the root Bridge Agent Factory.
    * @param _branchBridgeAgentFactory Address of the branch Bridge Agent Factory.
    * @param _refundee Receiver of any leftover execution gas upon reaching the destination network.
    * @param _dstChainId Chain Id of the branch chain where the new Bridge Agent will be deployed.
    * @param _gParams Gas parameters for remote execution.
    */
function toggleBranchBridgeAgentFactory(
    address _rootBridgeAgentFactory,
    address _branchBridgeAgentFactory,
    address _refundee,
    uint16 _dstChainId,
    GasParams calldata _gParams
) external payable onlyOwner {
    if (!IPort(rootPortAddress).isBridgeAgentFactory(_rootBridgeAgentFactory)) {
        revert UnrecognizedBridgeAgentFactory();
    }

    // Encode CallData
    bytes memory params = abi.encode(_branchBridgeAgentFactory);

    // Pack funcId into data
    bytes memory payload = abi.encodePacked(bytes1(0x03), params);

    //Add new global token to branch chain
    IBridgeAgent(bridgeAgentAddress).callOut{value: msg.value}(
        payable(_refundee), _refundee, _dstChainId, payload, _gParams
    );
}
```

In this function, the `_rootBridgeAgentFactory` parameter is superfluous for the operation of the function. 

Removing this parameter is essential to enhance the efficiency of the code.
