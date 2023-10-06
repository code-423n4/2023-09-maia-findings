# QA Report
## Summary
- [01] Should check if the `_deposit` is 0 before performing transfer in `ArbitrumBranchPort.sol#depositToPort()`
- [02] Missing `requiresApprovedCaller()` modifier in `VirtualAccount.sol#payableCall()`
- [03] Consider use `PARAMS_ADDRESS_SIZE` rather than `20` in `BranchBridgeAgent.sol#_requiresEndpoint()`
- [04] Argument `_nonce` is not used in `RootBridgeAgent.sol#lzReceive()` and `BranchBridgeAgent.sol#lzReceive()`
- [05] `RootBridgeAgent.sol#lzReceive()` should be override
- [06] `BranchBridgeAgent.sol#getFeeEstimate()` and `RootBridgeAgent.sol#getFeeEstimate()` are not used
- [07] `getBranchBridgeAgentPath[_dstChainId]` should be packed into 40 bytes before executing `endpoint.send()`
### Total 07 instances
## [01] Should check if the `_deposit` is 0 before performing transfer in `ArbitrumBranchPort.sol#depositToPort()`
If the `_deposit` is 0, the code below woud revert : [here](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/ArbitrumBranchPort.sol#L66C72-L66C80)
```
File: ArbitrumBranchPort.sol
50:    function depositToPort(address _depositor, address _recipient, address _underlyingAddress, uint256 _deposit)
---SNIP---
66:        _underlyingAddress.safeTransferFrom(_depositor, address(this), _deposit);
```
### Recommendation
The protocol should check if `_deposit` is 0 before performing transfer.

## [02] Missing `requiresApprovedCaller()` modifier in `VirtualAccount.sol#payableCall()`
- A Virtual Account allows users to manage assets and perform interactions remotely. In `VirtualAccount.sol` there is a `requireApprovedCaller()` modifier to verify whether the sender of the message is approved to use the virtual account or not. Either the owner or an approved router.
- The `requiresApprovedCaller()` modifier is used in most functions in the `VirtualAccount.sol` contract: `withdrawNative()`, `withdrawERC20()`, `withdrawERC721()`, `call()`. But is missing in `payableCall()`.
### Recommendation
Consider add `requiresApprovedCaller()` modifier to `payableCall()`.

## [03] Consider use `PARAMS_ADDRESS_SIZE` rather than `20` in `BranchBridgeAgent.sol#_requiresEndpoint()`
[LINK TO CODE](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L936-L944)
```
File: BranchBridgeAgent.sol
936:     function _requiresEndpoint(address _endpoint, bytes calldata _srcAddress) internal view virtual {
---SNIP---
943:        if (rootBridgeAgentAddress != address(uint160(bytes20(_srcAddress[20:])))) revert LayerZeroUnauthorizedCaller();
```
[LINK TO CODE](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/interfaces/BridgeAgentConstants.sol#L37)
```
File: BridgeAgentConstants.sol
37:    uint256 internal constant PARAMS_ADDRESS_SIZE = 20;
```
### Recommendation
Consider change to
```
        if (rootBridgeAgentAddress != address(uint160(bytes20(_srcAddress[PARAMS_ADDRESS_SIZE:])))) revert LayerZeroUnauthorizedCaller();
```

## [04] Argument `_nonce` is not used in `RootBridgeAgent.sol#lzReceive()` and `BranchBridgeAgent.sol#lzReceive()`
- According to `ILayerZeroReceiver.sol` , the third argument is `uint64 _nonce`, however ` it is not used in `RootBridgeAgent.sol#lzReceive()` and `BranchBridgeAgent.sol#lzReceive()`  : [here](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/interfaces/ILayerZeroReceiver.sol#L11)
```
    function lzReceive(uint16 _srcChainId, bytes calldata _srcAddress, uint64 _nonce, bytes calldata _payload) external;
```

- `BranchBridgeAgent.sol#lzReceive()` : [here](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L578-L584)
```
    function lzReceive(uint16, bytes calldata _srcAddress, uint64, bytes calldata _payload) public override {
        address(this).excessivelySafeCall(
            gasleft(),
            150,
            abi.encodeWithSelector(this.lzReceiveNonBlocking.selector, msg.sender, _srcAddress, _payload)
        );
    }
```
- `RootBridgeAgent.sol#lzReceive()` : [here](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L423-L431)
```
    function lzReceive(uint16 _srcChainId, bytes calldata _srcAddress, uint64, bytes calldata _payload) public {
        (bool success,) = address(this).excessivelySafeCall(
            gasleft(),
            150,
            abi.encodeWithSelector(this.lzReceiveNonBlocking.selector, msg.sender, _srcChainId, _srcAddress, _payload)
        );

        if (!success) if (msg.sender == getBranchBridgeAgent[localChainId]) revert ExecutionFailure();
    }
```

## [05] `RootBridgeAgent.sol#lzReceive()` should be override
- `RootBridgeAgent.sol#lzReceive()` : [here](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L423-L431)
```
    function lzReceive(uint16 _srcChainId, bytes calldata _srcAddress, uint64, bytes calldata _payload) public {
        (bool success,) = address(this).excessivelySafeCall(
            gasleft(),
            150,
            abi.encodeWithSelector(this.lzReceiveNonBlocking.selector, msg.sender, _srcChainId, _srcAddress, _payload)
        );

        if (!success) if (msg.sender == getBranchBridgeAgent[localChainId]) revert ExecutionFailure();
    }
```
### Recommendation
Add override to `RootBridgeAgent.sol#lzReceive()` 

## [06] `BranchBridgeAgent.sol#getFeeEstimate()` and `RootBridgeAgent.sol#getFeeEstimate()` are not used
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L140-L153
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L161C14-L173
`BranchBridgeAgent.sol#getFeeEstimate()` and `RootBridgeAgent.sol#getFeeEstimate()` are implemented but not used so consider removing unused functions

## [07] `getBranchBridgeAgentPath[_dstChainId]` should be packed into 40 bytes before executing `endpoint.send()`
There are 03 instances of this issue:
[_performCall()](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L808C-L825) , [_performRetrySettlementCall()](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L856C-L917) , [_performFallbackCall()](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L938-L942)
```
File: RootBridgeAgent.sol
64:    /// @notice Message Path for each connected Branch Bridge Agent as bytes for Layzer Zero interaction = localAddress + destinationAddress abi.encodePacked()
65:    mapping(uint256 chainId => bytes branchBridgeAgentPath) public getBranchBridgeAgentPath;

---
808:    function _performCall(
---SNIP---
823:            ILayerZeroEndpoint(lzEndpointAddress).send{value: msg.value}(
824:                _dstChainId,
825:                getBranchBridgeAgentPath[_dstChainId],

---
856:    function _performRetrySettlementCall(
---SNIP---
915:            ILayerZeroEndpoint(lzEndpointAddress).send{value: _value}(
916:                _dstChainId,
917:                getBranchBridgeAgentPath[_dstChainId],

---
938:    function _performFallbackCall(address payable _refundee, uint32 _depositNonce, uint16 _dstChainId) internal {
939        //Sends message to LayerZero messaging layer
940:        ILayerZeroEndpoint(lzEndpointAddress).send{value: address(this).balance}(
941:            _dstChainId,
942:            getBranchBridgeAgentPath[_dstChainId],
```
According to [docs](https://layerzero.gitbook.io/docs/evm-guides/master/how-to-send-a-message) : `remote address concated with local address packed into 40 bytes` then it is  passed into the `endpoint.send` function.
## Recommendation
`getBranchBridgeAgentPath[_dstChainId]` should be packed into 40 bytes before executing `endpoint.send()`
Eg:
```
getBranchBridgeAgentPath[_dstChainId] = abi.encodePacked(getBranchBridgeAgent[_dstChainId], address(this));
```
#### NOTE: In ` _performCall()` and `_performRetrySettlementCall()` : 
```
        address callee = getBranchBridgeAgent[_dstChainId];
```