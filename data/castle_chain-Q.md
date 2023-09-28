## missed events should be emitted 
the contract should emit events with the `nonce` in the bridge functions in the Branch bridge agent contracts 
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L209-L228
```
    function callOutAndBridge(
        address payable _refundee,
        bytes calldata _params,
        DepositInput memory _dParams,
        GasParams calldata _gParams
    ) external payable override lock {
        //Cache Deposit Nonce
        uint32 _depositNonce = depositNonce;


        //Encode Data for cross-chain call.
        bytes memory payload = abi.encodePacked(
            bytes1(0x02), _depositNonce, _dParams.hToken, _dParams.token, _dParams.amount, _dParams.deposit, _params
        );


        //Create Deposit and Send Cross-Chain request
        _createDeposit(_depositNonce, _refundee, _dParams.hToken, _dParams.token, _dParams.amount, _dParams.deposit);


        //Perform Call
        _performCall(_refundee, payload, _gParams);
    }
```
## should make sure that the msg.value is greater than or equal the gas parameters 
when the user specify gas parameters the protocol should make sure that the msg.value is greater than or equal the gas parameters 
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L209-L228

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L195-L200
