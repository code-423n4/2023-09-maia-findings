# [QA-01] Lack of following LayerZero Integration Checklist by hardcoding `zroPaymentAddress`

The [LayerZero Integration Checklist](https://layerzero.gitbook.io/docs/evm-guides/layerzero-integration-checklist) states:

* `Do not hardcode address zero (address(0)) as zroPaymentAddress when estimating fees and sending messages. Pass it as a parameter instead.`

The issue affects below instances:
[File: BranchBridgeAgent.sol](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L770-L778)
```
        ILayerZeroEndpoint(lzEndpointAddress).send{value: msg.value}(
            rootChainId,
            rootBridgeAgentPath,
            _payload,
            payable(_refundee),
            address(0),
            abi.encodePacked(uint16(2), _gParams.gasLimit, _gParams.remoteBranchExecutionGas, rootBridgeAgentAddress)
        );
    }
```

[File: BranchBridgeAgent.sol](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L787-L794)
```
       ILayerZeroEndpoint(lzEndpointAddress).send{value: address(this).balance}(
            rootChainId,
            rootBridgeAgentPath,
            abi.encodePacked(bytes1(0x09), _settlementNonce),
            _refundee,
            address(0),
            ""
        );
```

[File: RootBridgeAgent.sol](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L823-L830)
```
            ILayerZeroEndpoint(lzEndpointAddress).send{value: msg.value}(
                _dstChainId,
                getBranchBridgeAgentPath[_dstChainId],
                _payload,
                _refundee,
                address(0),
                abi.encodePacked(uint16(2), _gParams.gasLimit, _gParams.remoteBranchExecutionGas, callee)
            );
```

[File: RootBridgeAgent.sol](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L915-L922)
```
            ILayerZeroEndpoint(lzEndpointAddress).send{value: _value}(
                _dstChainId,
                getBranchBridgeAgentPath[_dstChainId],
                payload,
                payable(_refundee),
                address(0),
                abi.encodePacked(uint16(2), _gParams.gasLimit, _gParams.remoteBranchExecutionGas, callee)
            );
```

[File: RootBridgeAgent.sol](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L940-L9947)
```
        ILayerZeroEndpoint(lzEndpointAddress).send{value: address(this).balance}(
            _dstChainId,
            getBranchBridgeAgentPath[_dstChainId],
            abi.encodePacked(bytes1(0x04), _depositNonce),
            payable(_refundee),
            address(0),
            ""
        );
```

According to `ILayerZeroEndpoint` interface, the 5th parameter of `send` function is `_zroPaymentAddress`, 

[File: ILayerZeroEndpoint.sol](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/interfaces/ILayerZeroEndpoint.sol#L15-L22)
```
    function send(
        uint16 _dstChainId,
        bytes calldata _destination,
        bytes calldata _payload,
        address payable _refundAddress,
        address _zroPaymentAddress,
        bytes calldata _adapterParams
    ) external payable;
```

which implies that this [LayerZero Integration Checklist](https://layerzero.gitbook.io/docs/evm-guides/layerzero-integration-checklist) requirement is not fullfilled, because both `RootBridgeAgent.sol` and `BranchBridgeAgent.sol` hardcode `_zroPaymentAddress` to `address(0)`, instead of passing it as parameter.

# [QA-02] Invalid comment describing `GasParams`

Issue occurs in: [File: BridgeAgentStructs.sol](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/interfaces/BridgeAgentStructs.sol#L10)

The `GasParam` is being used for crafting the `_adapterParams` for `ILayerZeroEndpoint.send()` call.

However, this comment: ```gas allocated for remote branch execution. Must be lower than `gasLimit` ``` is incorrect and seems to be a leftover from the previous architecture. Especially, that even LayerZero documentation mentions that the native gas value is bigger than the gasLimit - https://layerzero.gitbook.io/docs/evm-guides/advanced/relayer-adapter-parameters#airdrop


# [QA-03] Avoid using toggle functions

It's good security practice to avoid toggle functions in the blockchain, because their result may not lead to the expected result - if the first transaction you sent was not correctly submitted (but it's just pending for a very long period of time). This will lead to double-toggling the state (which basically means - setting the initial value again).

Protocol implements multiple of toggle functions:

* File: `BranchPort.sol`

```
    function toggleBridgeAgentFactory(address _newBridgeAgentFactory) external override requiresCoreRouter {
        isBridgeAgentFactory[_newBridgeAgentFactory] = !isBridgeAgentFactory[_newBridgeAgentFactory];

        emit BridgeAgentFactoryToggled(_newBridgeAgentFactory);
    }

    /// @inheritdoc IBranchPort
    function toggleBridgeAgent(address _bridgeAgent) external override requiresCoreRouter {
        isBridgeAgent[_bridgeAgent] = !isBridgeAgent[_bridgeAgent];

        emit BridgeAgentToggled(_bridgeAgent);
    }


    /// @inheritdoc IBranchPort
    function toggleStrategyToken(address _token) external override requiresCoreRouter {
        isStrategyToken[_token] = !isStrategyToken[_token];

        emit StrategyTokenToggled(_token);
    }

    function togglePortStrategy(address _portStrategy, address _token) external override requiresCoreRouter {
        isPortStrategy[_portStrategy][_token] = !isPortStrategy[_portStrategy][_token];

        emit PortStrategyToggled(_portStrategy, _token);
    }
```

* File: `RootPort.sol`

```
  /// @inheritdoc IRootPort
    function toggleVirtualAccountApproved(VirtualAccount _userAccount, address _router)
        external
        override
        requiresBridgeAgent
    {
        isRouterApproved[_userAccount][_router] = !isRouterApproved[_userAccount][_router];
    }

     function toggleBridgeAgent(address _bridgeAgent) external override onlyOwner {
        isBridgeAgent[_bridgeAgent] = !isBridgeAgent[_bridgeAgent];

        emit BridgeAgentToggled(_bridgeAgent);
    }

     function toggleBridgeAgentFactory(address _bridgeAgentFactory) external override onlyOwner {
        isBridgeAgentFactory[_bridgeAgentFactory] = !isBridgeAgentFactory[_bridgeAgentFactory];

        emit BridgeAgentFactoryToggled(_bridgeAgentFactory);
    }
```


Similar findings have been evaluated as low-risk in the previous Code4rena contests:
* https://code4rena.com/reports/2021-06-realitycards#l-24-dangerous-toggle-functions

Instead of creating a toggle-like functions - create two separate functions - first one to enable the state, and the 2nd one - to disable it.

# [QA-04] Lack of following LayerZero Integration Checklist by hardcoding `useZro`

The [LayerZero Integration Checklist](https://layerzero.gitbook.io/docs/evm-guides/layerzero-integration-checklist) states:

```
Do not hardcode useZro to false when estimating fees and sending messages. Pass it as a parameter instead.
```

In function `getFeeEstimate()` we're calling `ILayerZeroEndpoint(lzEndpointAddress).estimateFees`, which hardcodes `useZro` to false, instead of passing it as parameter:

[File: RootBridgeAgent.sol](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L146-L152)
```
 (_fee,) = ILayerZeroEndpoint(lzEndpointAddress).estimateFees(
            _dstChainId,
            address(this),
            _payload,
            false,
            abi.encodePacked(uint16(2), _gasLimit, _remoteBranchExecutionGas, getBranchBridgeAgent[_dstChainId])
        );
```

[File: BranchBridgeAgent.sol](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L166-L172)
```
        (_fee,) = ILayerZeroEndpoint(lzEndpointAddress).estimateFees(
            rootChainId,
            address(this),
            _payload,
            false,
            abi.encodePacked(uint16(2), _gasLimit, _remoteBranchExecutionGas, rootBridgeAgentAddress)
        );
```

This seem to intended way of calculating fees - since the protocol does not use ZRO to pay for the messages and does not pose any security threat. Nonetheless, it violates the LayerZero Integration Checklist, thus is had to be mentioned in the QA Report.

# [N-01] Risk of delivery messages across chains

Protocol uses LayerZero to transport messages across different chains. In case of LayerZero being compromised, the worst case scenario might include forges of these messages. This may be causing havoc and possibility of funds loss.
Even though, using the LayerZero is the conscious choice for the developers - this risk should also be taken into a consideration and mentioned in the report.
For the [reference](https://github.com/solodit/solodit_content/blob/main/reports/Trust%20Security/2023-05-23-Mozaic%20Archimedes.md#delivery-of-messages-across-chains), similar issues has been reported as Informational - in other web3 security report. 

