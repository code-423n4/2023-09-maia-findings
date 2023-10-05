# LOW

## [1] May enable a non-existent BridgeAgent/BridgeAgentFactory

In RootPort,there are functions to toggle the stage of BridgeAgent and BridgeAgentFactory.But there seems to be a missing logical judgment here.A BridgeAgent/BridgeAgentFactory should have been added before it can be toggled.  
  
Without this logical judgment,the Port owner may enable a non-existent BridgeAgent/BridgeAgentFactory。  
  
In BranchPort,these functions will be called by router.Router will do judgment before it be toggled.So there's no need to worry about this issue.

GitHub:[414](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L414)


    File:src/RootPort.sol

    414        function toggleBridgeAgent(address _bridgeAgent) external override onlyOwner {
    415            isBridgeAgent[_bridgeAgent] = !isBridgeAgent[_bridgeAgent];
    416
    417            emit BridgeAgentToggled(_bridgeAgent);
    418        }

  
GitHub:[431](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L431)


    File:src/RootPort.sol

    431        function toggleBridgeAgentFactory(address _bridgeAgentFactory) external override onlyOwner {
    432            isBridgeAgentFactory[_bridgeAgentFactory] = !isBridgeAgentFactory[_bridgeAgentFactory];
    433
    434            emit BridgeAgentFactoryToggled(_bridgeAgentFactory);
    435        }


# NOT-CRITAL

## [1] Note: Function @notice is not match
There is a Function @notice does not match the documentation and function functionality.The document description and function implementation indicate that this function is seems to be used to add a new bridgeAgent.  
  
This will not cause safety issues, but it may cause confusion and affect the reading of the contract.  
  
Changing it to synchronize with the document will improve standardization.
  
GitHub:[188](https://github.com/code-423n4/2023-09-maia/blob/6c21f6b739252367c5fe770fed3ff76eb3ca2c86/src/CoreBranchRouter.sol#L188)
    
    File: src/CoreBranchRouter.sol

    187        /**
    188            * @notice Function to deploy/add a token already active in the global environment in the Root Chain.
    189            *         Must be called from another chain.
    190            *    @param _newBranchRouter the address of the new branch router.
    191            *    @param _branchBridgeAgentFactory the address of the branch bridge agent factory.
    192            *    @param _rootBridgeAgent the address of the root bridge agent.
    193            *    @param _rootBridgeAgentFactory the address of the root bridge agent factory.
    194            *    @param _refundee the address of the excess gas receiver.
    195            *    @param _gParams Gas parameters for remote execution.
    196            *    @dev FUNC ID: 2
    197            *    @dev all hTokens have 18 decimals.
    198            */
    199            function _receiveAddBridgeAgent(
    200                address _newBranchRouter,
    201                address _branchBridgeAgentFactory,
    202                address _rootBridgeAgent,
    203                address _rootBridgeAgentFactory,
    204                address _refundee,
    205                GasParams memory _gParams
    204            ) internal virtual


There is another place here.Although its Function @notice matches the document, the functional description is not clear enough.  
  
Its function is to add portStrategy, update daily limit, or disable portStrategy for an underlyingToken instead of adding an underlyingToken.  
  
Clearer description of functions will ease of reading and maintenance.  


GitHub[299](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/CoreBranchRouter.sol#L299C1-L299C1)

    File: src/CoreBranchRouter.sol

    298        /**
    299        * @notice Function to deploy/add a token already active in the global environment in the Root Chain.
    300        *         Must be called from another chain.
    301        *  @param _portStrategy the address of the port strategy.
    302        *  @param _underlyingToken the address of the underlying token.
    303        *  @param _dailyManagementLimit the daily management limit.
    304        *  @param _isUpdateDailyLimit if the daily limit is being updated.
    305        *  @dev FUNC ID: 6
    306        */
    307        function _managePortStrategy(
    308            address _portStrategy,
    309            address _underlyingToken,
    310            uint256 _dailyManagementLimit,
    311            bool _isUpdateDailyLimit
    312        ) internal {
## [2] Consider check deposit greater than 0
If the deposit parameter of callOutAndBridge or callOutAndBridgeMultiple is zero, the function will still continue to execute and create a deposit in storage and make a cross-chain call to root.So consider check deposit greater than zero.


GitHub[208](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L208C1-L208C1)

    File: src/CoreBranchRouter.sol

    208        /// @inheritdoc IBranchBridgeAgent
    209        function callOutAndBridge(
    210            address payable _refundee,
    211            bytes calldata _params,
    212            DepositInput memory _dParams,
    213            GasParams calldata _gParams
    214        ) external payable override lock {
    215            //Cache Deposit Nonce
    216            uint32 _depositNonce = depositNonce;
    217
    218            //Encode Data for cross-chain call.
    219            bytes memory payload = abi.encodePacked(
    220                bytes1(0x02), _depositNonce, _dParams.hToken, _dParams.token, _dParams.amount, _dParams.deposit, _params
    221            );
    222
    223            //Create Deposit and Send Cross-Chain request
    224            _createDeposit(_depositNonce, _refundee, _dParams.hToken, _dParams.token, _dParams.amount, _dParams.deposit);
    225
    226            //Perform Call
    227            _performCall(_refundee, payload, _gParams);
    228        }

