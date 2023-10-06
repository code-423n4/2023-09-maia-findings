## 1. `_refundee` VARIABLE HAS REDUNDANT DECLARATION AS `payable`

The `BranchBridgeAgent._performCall` function is used to perform calls to LayerZero messaging layer Endpoint for cross-chain messaging. The `_refundee` is a payable address which is used to refund excess gas to. The `_refundee` is declared as a `payable` address when passed in as an input parameter to the `_performCall` function. The issue is `_refundee` is again recasted to a `payable` address inside the call to the layerzero endpoint which is not required as shown below:

        ILayerZeroEndpoint(lzEndpointAddress).send{value: msg.value}( //@audit-info - uses layer zero messaging layer endpoint for cross-chain messaging
            rootChainId,
            rootBridgeAgentPath,
            _payload,
            payable(_refundee),
            address(0),
            abi.encodePacked(uint16(2), _gParams.gasLimit, _gParams.remoteBranchExecutionGas, rootBridgeAgentAddress)
        );

Hence it is recommneded to remove the redundant recasting of the `_refundee` address to a `payable` address inside the call to the `layerzero` endpoint for cross chain messaging.

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L774
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L919

## 2. LACK OF INPUT VALIDATION CHECK ON `address(0)` COULD LEAD TO LOSS OF FUNDS

The `BranchBridgeAgent._clearToken` is an internal function which is used to request balance clearance from a Port to a given user. The `_clearToken` is called by the `BranchBridgeAgent.clearToken` function by passing in the `_recipient` address to whom the balance clearance should be sent to. 

The issue is during the execution flow of the `BranchBridgeAgent.clearToken` function there is no check to verifty whether the `_recipient != address(0)`. Hence if the address(0) is passed in as the `_recipient` address the balance clearance will be sent to the address(0) which is loss of funds.

The `_clearToken` function calls the `IPort.bridgeIn` function and `IPort.withdraw` function. Inside the `bridgeIn` function and `withdraw` function `ERC20` `_mint` and `safeTransfer` functions are used. But event inside these functions no `address(0)` check is performed on the passed in `_recipient` address.

Hence it is recommended to perform the input validation check inside the `BranchBridgeAgent._clearToken` function to ensure the passed in `_recipient` address is not equal to `address(0)`. It is recommended to add the following line of code at the beginning of the `_clearToken` function.

    require( _recipient != address(0), "_recipient can not be address(0)");

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L906-L914
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L494-L571

## 3. `_recipient` ADDRESS IS NOT DECLARED AS A `payable` ADDRESS IN THE `BranchBridgeAgentExecutor.executeWithSettlement` FUNCTION

The `BranchBridgeAgentExecutor.executeWithSettlement` function is used to execute a cross-chain request with a single settlement. The function accepts the `_recipient` address which is the the recipient of the settlement as shown below:

    function executeWithSettlement(address _recipient, address _router, bytes calldata _payload)

But the issue is `_recipient` is not declared as a `payable` address. Since `_recipient` is expected to receive the `native ETH` it is recommneded to declare the `_recipient` address as a payable address in the `executeWithSettlement` function signature. The modified line of code is shown below:

    function executeWithSettlement(address payable _recipient, address _router, bytes calldata _payload)

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgentExecutor.sol#L66

## 4. `BranchPort._bridgeOut` TRANSACTION COULD REVERT IF THE `_amount < _deposit`

The `BranchPort._bridgeOut` is an internal function used to bridge out hTokens and underlying tokens. Within the function implementation the `_hTokenAmount` to bridge out is calculated as follows:

        uint256 _hTokenAmount = _amount - _deposit; 

The issue here is that if the `_amount < _deposit` the arithmetic operation will revert due to `underflow` error but the cause for the revert will not be clear. Hence it is recommended to add the below conditional check to verify (_amount > _deposit) before calculating the _hTokenAmount.

        require(_amount > _deposit, "Arithmetic Underflow");
        uint256 _hTokenAmount = _amount - _deposit; 

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L522

## 5. THE `settlement.status` IS BY DEFAULT SET TO THE `STATUS_SUCCESS` STATE SINCE THE `STATUS_SUCCESS == 0`

The `RootBridgeAgent.redeemSettlement` function is used to enable an user to redeem thier settlement. Here initially the `settlement.status` is checked to ensure whether the settlement is redeemable as shown below:

        if (settlement.status == STATUS_SUCCESS) revert SettlementRedeemUnavailable();

The `STATUS_SUCCESS` is declared in the `BridgeAgentConstant.sol` as shown below:

    uint8 internal constant STATUS_SUCCESS = 0;

At the end of the `RootBridgeAgent.redeemSettlement` function the settlement corresponding to the `_settlementNonce` is deleted as shown below:

        delete getSettlement[_settlementNonce]; 

Due to the above deletion the `settlement.status` will be equal to `0` which is the default value. Hence as a result this is still equal to the `STATUS_SUCCESS` state. Hence even for any non-existing settlement and for a settlement that has already being deleted the `settlement.status` will be equal to the `STATUS_SUCCESS` state. 

Hence it is recommended to update the constant `STATUS_SUCCESS = 2` because in that case the default value of the `settlement.status` will not be equal to the `STATUS_SUCCESS` state by default.

https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/BridgeAgentConstants.sol#L23
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L307
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L343

## 6. LACK OF INPUT VALIDATION CHECK OF `address(0)` FOR THE `_recipient` ADDRESS IN THE `RootBridgeAgent.bridgeIn` FUNCTION

The `RootBridgeAgent.bridgeIn` function is used to bridge in `hTokens` from branch to root. The `bridgeIn` function calls the `IPort.bridgeToRoot` function by passing in the `_recipient` address to whom the `_amount - _deposit` amount of `_hTokens` will be transferred to and the `_deposit` amount of `_hTokens` will be minted to inside the `bridgeToRoot` function.

But the issue here is that in none of the `bridgeIn`, `bridgeToRoot` or the `mint` functions the `_recipient` address is not checked for `address(0)`. Hence if the `address(0)` is passed in as the `_recipient` address to the `RootBridgeAgent.bridgeIn` function (address(0) is the default value), the `_hTokens` transferred to the `_recipient address (address(0))` will be permenantly lost.

Hence it is recommneded to perform the `address(0)` check inside the ``IPort.bridgeToRoot` function as an input validation check.

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L351
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L377-L378
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L277-L291

## 7. KEY DATA TYPE AND NAMING MISMATCH IN THE MAPPING DECLARATION OF THE `RootPort` CONTRACT

In the `RootPort.sol` contract there are four mappings declared on `hTokens`. They are as follows:

    /// @notice ChainId -> Local Address -> Global Address
    mapping(address chainId => mapping(uint256 localAddress => address globalAddress)) public getGlobalTokenFromLocal;

    /// @notice ChainId -> Global Address -> Local Address
    mapping(address chainId => mapping(uint256 globalAddress => address localAddress)) public getLocalTokenFromGlobal;

    /// @notice ChainId -> Underlying Address -> Local Address
    mapping(address chainId => mapping(uint256 underlyingAddress => address localAddress)) public
        getLocalTokenFromUnderlying;

    /// @notice Mapping from Local Address to Underlying Address.
    mapping(address chainId => mapping(uint256 localAddress => address underlyingAddress)) public
        getUnderlyingTokenFromLocal;

As it is evident in each of the above mappings there is a mismatch between `chainId` and `localAddress` names used. 

These mappings declare that the first key as an `address` (which is named `chainId` but it's actually an address), and the second key as a `uint256` (which is named `localAddress`, which is misleading because it's a `uint256`, `not an address`). This can be confusing to both developers and auditors and can lead to serious vulenarabilities if the naming mismatch is not understood properly.

Hence it is recommended to correct the above key data type and naming mismatch, and update the mapping declaration as shown below:

    /// @notice Local Address -> ChainId -> Global Address
    mapping(address localAddress => mapping(uint256 chainId => address globalAddress)) public getGlobalTokenFromLocal;

    /// @notice Global Address -> ChainId -> Local Address
    mapping(address globalAddress => mapping(uint256 chainId => address localAddress)) public getLocalTokenFromGlobal;

    /// @notice Underlying Address -> ChainId -> Local Address
    mapping(address underlyingAddress => mapping(uint256 chainId => address localAddress)) public
        getLocalTokenFromUnderlying;

    /// @notice Mapping from Local Address to Underlying Address.
    mapping(address localAddress => mapping(uint256 chainId => address underlyingAddress)) public
        getUnderlyingTokenFromLocal;

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L89-L101

## 8. `ERC20hTokenRootFactory` CONTRACT DOES NOT HAVE FUNCTIONALITY TO REMOVE UNWANTED `hTokens` IF THE NEED ARISES IN THE FUTURE

The `ERC20hTokenRootFactory` contract is a factory contract used to create `new hTokens`. The new hTokens are create using the `createToken` function and the newly created `hTokens` are added to the `hTokens` array as shown below:

        newToken = new ERC20hTokenRoot(
            localChainId,
            address(this),
            rootPortAddress,
            _name,
            _symbol,
            _decimals
        );
        hTokens.push(newToken);

The issue here is that even if the `hTokens` are added to the `hTokens` array there is no mechanism to remove the `hTokens` if the need arises in the future. As a result the `hTokens` array will keep on growing. 

Since there is a `ERC20hTokenRootFactory.getHTokens` function to retrieve the `hTokens` array, the retreiver could be using the `hTokens` array to loop through the array to execute a different logic. If the `hTokens` array is extremely large then the retriever will get transaction out-of-gas exception. Hence the `hTokens` array returned by the `ERC20hTokenRootFactory.getHTokens` function will not be useful for any other logic execution.

Hence it is recommended to add a functionality remove the unwanted global `hTokens` from the `hTokens` array in the `ERC20hTokenRootFactory` contract and update the corresponding token mappings (Eg : getGlobalTokenFromLocal) in the `RootPort` contract. 

https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenRootFactory.sol#L81-L89
https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenRootFactory.sol#L63-L65
https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenBranchFactory.sol#L72
https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenBranchFactory.sol#L87-L89

## 9. `address(0)` CHECK IS NOT PERFORMED ON THE `_newBranchBridgeAgent` ADDRESS IN THE `RootPort.syncBranchBridgeAgentWithRoot` FUNCTION

The `RootPort.syncBranchBridgeAgentWithRoot` function is used to sync the branch bridge agent for a given `_branchChainId` with the root bridge agent. The function calls the `IBridgeAgent.syncBranchBridgeAgent` function to perform the `_newBranchBridgeAgent` update.

But the issue is there is no `address(0)` check performed on the `_newBranchBridgeAgent` address for which the `branch bridge agent` will be set to. 

Hence it is recommended to perform the address(0) check on the `_newBranchBridgeAgent` value inside the `RootPort.syncBranchBridgeAgentWithRoot` to ensure `address(0)` is not as the branch bridge agent address for the given `_branchChainId` in the respective `_rootBridgeAgent`.

Following line of code can be added to the `RootPort.syncBranchBridgeAgentWithRoot` function before the `BridgeAgent.syncBranchBridgeAgent` is called.

    require(_newBranchBridgeAgent != address(0), "Can not be address(0)");

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L404
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1181-L1182

## 10. RETURN BOOLEAN VALUE OF `approve` TRANSACTION IS NOT CHECKED FOR SUCCESS IN THE `BaseBranchRouter._transferAndApproveToken` FUNCTION

The `BaseBranchRouter._transferAndApproveToken` is an internal function used to transfer tokens to the `Branch Router` during `callOutAndBridge` transaction. During the `_transferAndApproveToken` function execution both the local branch tokens (`_hToken`) and underlying tokens (`_token`) are transferred to the `BaseBranchRouter` contract. 

Then both the `_hToken` and the `_token` are approved by the `BaseBranchRouter` contract to the `_localPortAddress` contract to be stored in the `port`. It is shown below:

            ERC20(_hToken).approve(_localPortAddress, _amount - _deposit);

            ERC20(_token).approve(_localPortAddress, _deposit);              

This issue here is certain ERC20 tokens communicate the `approve` transaction failure by returning a `boolean false` value. Since the above lines of code does not check the return boolean value of the `approve` function it could lead to failed transactions being accepted as succesful without approving anything.

Hence it is recommneded to check the return boolean value of the `approve` transaction and revert if the `success` value of the returned boolean is `false`.

Note : In the automated report one instance (BaseBranchRouter.sol#L175) is mentioned. But There is another instance of this issue in `BaseBranchRouter.sol#L168`. As a result included this as a quality assurance vulnerabilitiy.

https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L168

## 11. THE `_refundee` CAN UNFAIRLY PROFIT FROM THE REFUNDED GAS SINCE `address(this).balance` IS USED TO CALCULATE THE BALANCE GAS AMOUNT TO BE SENT TO THE `_refundee` 

The `RootBridgeAgent` contract and `BranchBridgeAgent` contracts use `address(this).balance` to calculate the existing balance of the contract to pass as the gas amount during execution of the `RootBridgeAgent._execute`, `RootBridgeAgent._performFallbackCall` functions and while calling the `_performRetrySettlementCall` function. 

The issue is, if there is `native Eth` transferred to `RootBridgeAgent` or `BranchBridgeAgent` by another contract or an `EOA` by mistake those funds will also be transferred to the `_refundee` address of the respective transaction. Hence the `_refundee` gets an undue advantage and will profit with extra `native Eth`  from the mistake of another individual. 

Hence it is recommended to calculate the `eth balance` before and after the function call which transfers `native eth` to either `RootBridgeAgent` or `BranchBridgeAgent` contracts and then use the actually transferred `eth amount` to execute the cross-chain transaction. This can be done by declaring a state variable to keep track of the current eth balance of the contract at all times. And then use that variable to calculate the actually transferred eth amount to the contract. This will refund the correct amount of `gas` to the `_refundee` address if excessive gas remained after the transaction execution.

Once above change is made it is recommneded to add a `withdraw` function to withdraw the locked `eth` amount from the `RootBridgeAgent` contract and `BranchBridgeAgent` contracts. This will ensure fair operation of the system for all users.

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L695
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L754
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L779
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L940
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L717
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L737
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L787

## 12. `BranchPort.initialize` FUNCTION CAN BE CALLED MULTIPLE TIMES BY THE `owner` THUS MAKING THE CONTRACT VULNERABLE TO CENTRALIZATION AND SINGLE POINT OF FAILURE

The `BranchPort.initialize` function is `onlyOwner` controlled function which is used to initialize the `coreBranchRouterAddress` and `isBridgeAgentFactory[_bridgeAgentFactory]` state variables as shown below:

        coreBranchRouterAddress = _coreBranchRouter;
        isBridgeAgentFactory[_bridgeAgentFactory] = true;
        bridgeAgentFactories.push(_bridgeAgentFactory);

The issue here is that this function can be called by the owner any number of times thus changing the `coreBranchRouterAddress` address as he wants. This is not the expected behaviour of the `initialize` function which is expected to be called only once to initialze the states of the smart contract. Due to the above behaviour of the `initialize` function the `owner` is getting a greater control over the `BranchPort` contract which makes the contract vulnerable to single point of failure.

Hence it is recommended to `import` the `openzeppelin Initializable.sol` contract and add the `initializer` modifier to the `BranchPort.initialize` function so it can be called only once by the owner to initialize the smart contract.

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L122

## 13. `BranchPort._checkTimeLimit` TRANSACTION REVERT DUE TO ARITHMETIC UNDERFLOW IS NOT CORRECTLY HANDLED

The `BranchPort._checkTimeLimit` function is used to check if a Port Strategy has reached its daily management limit. The remaining daily limit is updated at the end of the `_checkTimeLimit` function as shown below:

        strategyDailyLimitRemaining[msg.sender][_token] = dailyLimit - _amount;

But there is no check performed to ensure `dailyLimit > _amount`. Hence if the `dailyLimit < _amount` transaction could revert silently. Hence it is recommended to add the following `require` statement before the `strategyDailyLimitRemaining[msg.sender][_token]` is updated.

        require(dailyLimit > _amount, "Daily limit has exceeded");
        strategyDailyLimitRemaining[msg.sender][_token] = dailyLimit - _amount;

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L493

## 14. DISCREPENCY BETWEEN `Natspect` COMMENTS AND ACTUAL LOGIC IMPLEMENTATION IN THE `hToken Address` PARSING IN THE `RootBridgeAgentExecutor._bridgeInMultiple` FUNCTION

The `RootBridgeAgentExecutor._bridgeInMultiple` function parses the input bytes array `_dParams`  into the respective token details. The `Natspect` comment on the parsing of the `hToken address` is given as below:

     *         1. PARAMS_TKN_START [hToken address has no offset from token information start]

In the `Natspec` comment is says there is no offset from the token information start. But as it is evident from the logic implementation there is an offset of `ADDRESS_END_OFFSET` byte amount as shown below:

        _dParams[
            PARAMS_TKN_START + (PARAMS_ENTRY_SIZE * i) + ADDRESS_END_OFFSET:
            PARAMS_TKN_START + (PARAMS_ENTRY_SIZE * currentIterationOffset)
        ]

This discrepency between the `Natspec` comment and actual logic implementation could be confusing to both developers and auditors.        

Hence the `Natspec` comment for the `hToken address` should be corrected to mention there is an offset of `ADDRESS_END_OFFSET` from token information start. 

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L262
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L286-L297

## 15. ERRORNEOUS NATSPEC COMMENT IN THE `BranchBridgeAgent._performCall` FUNCTION

The `BranchBridgeAgent._performCall` function is used to perform calls to LayerZero messaging layer Endpoint for cross-chain messaging. `_gParams` is passed in as a `GasParams` struct which includes the `LayerZero gas information`. The `Natspec` comments for the `_gParams` input parameter is given as shown below:

     *   @param _gParams LayerZero gas information. (_gasLimit,_remoteBranchExecutionGas,_nativeTokenRecipientOnDstChain)

But the `_gParams` is a variable of type `GasParams` struct and `GasParams` struct does not has a parameter named `_nativeTokenRecipientOnDstChain` within its scope. Hence above `Natspec` comments is errorneous and should be corrected as follows:

     *   @param _gParams LayerZero gas information. (_gasLimit,_remoteBranchExecutionGas)

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#763

## 16. TYPOS IN THE NATSPEC COMMENTS SHOULD BE CORRECTED

     *         `Naive` assets are deposited and hTokens are bridgedOut.

The above Natspec comment should be corrected as follows:

     *         `Native` assets are deposited and hTokens are bridgedOut.

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L803

    /// @notice `Deposit` nonce used for identifying transaction.

The above Natspec comment should be corrected as follows:

    /// @notice `Settlement` nonce used for identifying transaction.

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L74

     *     @param _localRouterAddress Local `Port` Address. 

The above Natspec comment should be corrected as follows:

     *     @param _localRouterAddress Local `Router` Address. 

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L103

     * @notice Internal function performs call to Layerzero `Enpoint` Contract for cross-chain messaging.

The above Natspec comment should be corrected as follows:

     * @notice Internal function performs call to Layerzero `Endpoint` Contract for cross-chain messaging.

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L934

    /// @notice `Deposit` nonce used for identifying transaction.

The above Natspec comment should be corrected as follows:

    /// @notice `Settlement` nonce used for identifying transaction.

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L74