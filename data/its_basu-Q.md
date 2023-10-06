# Ensure a check for new local token that is being added in `CoreBranchRouter.addLocalToken()`

## Details 

- To add a new local token for an underlying token `_underlyingAddress` the protocol has `CoreBranchRouter.addLocalToken()` which uses `ERC20hTokenBranchFactory.createToken()` to deploy a `newToken` which in turn appends `hTokens` array of `ERC20hTokenBranchFactory` with address of `newToken`. 

  ```solidity 
   function addLocalToken(address _underlyingAddress, GasParams calldata _gParams) external payable virtual {
        //Get Token Info
        uint8 decimals = ERC20(_underlyingAddress).decimals();
        
        
        //Create Token
        ERC20hToken newToken = ITokenFactory(hTokenFactoryAddress).createToken(
            ERC20(_underlyingAddress).name(), ERC20(_underlyingAddress).symbol(), ERC20(_underlyingAddress).decimals(), true
        );

        //Encode Data
        bytes memory params = abi.encode(_underlyingAddress, newToken, newToken.name(), newToken.symbol(), decimals);

        // Pack FuncId
        bytes memory payload = abi.encodePacked(bytes1(0x02), params);

        
        //Send Cross-Chain request (System Response/Request)
        IBridgeAgent(localBridgeAgentAddress).callOutSystem{value: msg.value}(payable(msg.sender), payload, _gParams);
    }
  ```
- It then sends the information to root chain using `BridgeAgent.callOutSystem` packed with `bytes1(0x02)` at the beginning.  

- the logic of `BridgeAgent.callOutSystem` performs a layer zero 'send' to the root chain with `0x00` at the start of the data. 

```solidity
function callOutSystem(address payable _refundee, bytes calldata _params, GasParams calldata _gParams)
        external
        payable
        override
        lock
        requiresRouter
    {
        //Encode Data for cross-chain call.
        bytes memory payload = abi.encodePacked(bytes1(0x00), depositNonce++, _params);

        //Perform Call
        _performCall(_refundee, payload, _gParams);
    }
```

```solidity
function _performCall(address payable _refundee, bytes memory _payload, GasParams calldata _gParams)
        internal
        virtual
    {
        // Sends message to LayerZero messaging layer
        // audit - check whether the address(0) implementation is correct or not
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

- On the Root chain side, the logic for `0x00` message implementation is this 

```solidity 
 function lzReceiveNonBlocking(address _endpoint, uint16 _srcChainId, bytes calldata _srcAddress, bytes calldata _payload
    ) public override requiresEndpoint(_endpoint, _srcChainId, _srcAddress) {
        // Deposit Nonce
        uint32 nonce;

        // DEPOSIT FLAG: 0 (System request / response)
        if (_payload[0] == 0x00) {
            // Parse deposit nonce
            nonce = uint32(bytes4(_payload[PARAMS_START:PARAMS_TKN_START]));

            // Check if tx has already been executed
            if (executionState[_srcChainId][nonce] != STATUS_READY) {
                revert AlreadyExecutedTransaction();
            }

            // Avoid stack too deep
            uint16 srcChainId = _srcChainId;

            // Try to execute remote request
            // Flag 0 - RootBridgeAgentExecutor(bridgeAgentExecutorAddress).executeSystemRequest(_localRouterAddress, _payload, _srcChainId)
            _execute(
                nonce,
                abi.encodeWithSelector(
                    RootBridgeAgentExecutor.executeSystemRequest.selector, localRouterAddress, _payload, srcChainId
                ),
                srcChainId
            );
```

- call to `RootBridgeAgentExecutor.executeSystemRequest` which calls to router 

```solidity
function executeSystemRequest(address _router, bytes calldata _payload, uint16 _srcChainId)
        external
        payable
        onlyOwner
    {
        //Try to execute remote request
        IRouter(_router).executeResponse(_payload[PARAMS_TKN_START:], _srcChainId);
    }
```

- finally Router calls an internal function `_addLocalToken()`

```solidty
function executeResponse(bytes calldata _encodedData, uint16 _srcChainId)
        external
        payable
        override
        requiresExecutor
    {
        // Parse funcId
        bytes1 funcId = _encodedData[0];

        ///  FUNC ID: 2 (_addLocalToken)
        if (funcId == 0x02) {
            (address underlyingAddress, address localAddress, string memory name, string memory symbol, uint8 decimals)
            = abi.decode(_encodedData[1:], (address, address, string, string, uint8));

            _addLocalToken(underlyingAddress, localAddress, name, symbol, decimals, _srcChainId);

            /// FUNC ID: 3 (_setLocalToken)
        } 
      // .... rest of the code 
```

- Finally the logic for `_addLocalToken` is as follows 

```solidity
function _addLocalToken(
        address _underlyingAddress,
        address _localAddress,
        string memory _name,
        string memory _symbol,
        uint8 _decimals,
        uint16 _srcChainId
    ) internal {
        // Verify if the underlying address is already known by the branch or root chain
        if (IPort(rootPortAddress).isGlobalAddress(_underlyingAddress)) revert TokenAlreadyAdded();
        if (IPort(rootPortAddress).isLocalToken(_underlyingAddress, _srcChainId)) revert TokenAlreadyAdded();
        if (IPort(rootPortAddress).isUnderlyingToken(_underlyingAddress, _srcChainId)) revert TokenAlreadyAdded();

        //Create a new global token
        address newToken = address(IFactory(hTokenFactoryAddress).createToken(_name, _symbol, _decimals));

        // Update Registry
        IPort(rootPortAddress).setAddresses(
            newToken, (_srcChainId == rootChainId) ? newToken : _localAddress, _underlyingAddress, _srcChainId
        );
    }

```
## Result

**Now, as it is clearly visible that this logic is checking whether the `localToken` for a particular `_underlyingAddress` on a specific `_srcChainId` is already set and rightfully reverts if that's the case**  

**My point is that the situation is reverted on the "root chain" side but nothing is done for reverting it on the "branch chain side"** 

**What I mean is that nothing is done to remove `newToken` address from `hToken` array of `ERC20hTokenBranchFactory`.** 

## Impact 

- This can create unnecessary confusion on the "branch side" with `hToken` array containing addresses that aren't even recognized by the "root chain" 

- There are unnecessary hTokens with which users can interact and that will result in unexpected outcomes for them. 

## Recommendation 

- Have a record of unerlying token to their htokens on the "branch" chain side as well with a simple mapping 
  ```solidity
  mapping(address underLyingToken => address hToken) underLyingTokenToHtokens
  ``` 
- Have it on the `BranchPort`
- And Finally implement this check everytime a new local token is being created.  