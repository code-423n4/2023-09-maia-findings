## Gas Optimizations

| Number | Issue | Instances | 
|--------|-------|-----------|
|[G-01]| Use Modifiers Instead of Functions To Save Gas |  4  | 
|[G-02]| Repeatedly doing the same operation (addition) involving a state variable should be avoided   |  1  | 
|[G-03]| Use the cached value instead of fetching a storage value   |  3  | 
|[G-04]| Unused initialization of variable inside they function    |  2  | 
|[G-05]| Optimizing the check order for cost efficient function execution |  3  |
|[G-06]| Do not cache immutable values |  2  | 
|[G-07]| Use != 0 instead of > 0 for unsigned integer comparison   |  18  | 
|[G-08]| Can Make The Variable Outside The Loop To Save Gas   |  6  | 
|[G-09]| Gas saving is achieved by removing the delete keyword (~60k)   |  2  | 
|[G-10]| With assembly, .call (bool success) transfer can be done gas-optimized  |  4  | 
|[G-11]| Duplicated require()/if() checks should be refactored to a modifier or function  |  14  | 
|[G-12]| Use a hardcoded address instead of address(this) |  39  |  
|[G-13]| Use assembly to write address storage values   |  6  | 
|[G-14]| Use do while loops instead of for loops   |  9  |  
|[G-15]| Use assembly to make more efficient back-to-back calls   |  4  | 
|[G-16]| Expensive operation inside a for loop   |  3  |  
|[G-17]| Change public function visibility to external   |  6  |  
|[G-18]| Create DepositMultipleInput and GasParams variable  immutable variable to avoid redundant external calls   |  5  |  
|[G-19]| Use assembly in place of abi.decode to extract calldata values more efficiently   |  29  |  
|[G-20]| Use assembly to validate msg.sender   |  25  | 
|[G-21]| Cache external calls outside of loop to avoid re-calling function on each iteration   |  3  | 
|[G-22]| A modifier used only once and not being inherited should be inlined to save gas   |  9  | 
|[G-23]| Not using the named return variable when a function returns, wastes deployment gas only in view function |  3  |
|[G-24]| Uncheck arithmetics operations that can’t underflow/overflow  |  14  |

## [G-01] Use Modifiers Instead of Functions To Save Gas

Example of two contracts with modifiers and internal view function:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.9;
contract Inlined {
    function isNotExpired(bool _true) internal view {
        require(_true == true, "Exchange: EXPIRED");
    }
function foo(bool _test) public returns(uint){
            isNotExpired(_test);
            return 1;
    }
}
// SPDX-License-Identifier: MIT
pragma solidity 0.8.9;
contract Modifier {
modifier isNotExpired(bool _true) {
        require(_true == true, "Exchange: EXPIRED");
        _;
    }
function foo(bool _test) public isNotExpired(_test)returns(uint){
        return 1;
    }
}

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L518-520

```solidity
file:  src/MulticallRootRouter.sol

518   function _decode(bytes calldata data) internal pure virtual returns (bytes memory) {
        return data;
    }

```



https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L604-606

```solidity
file:  src/MulticallRootRouter.sol

604   function _requiresExecutor() internal view {
        if (msg.sender != bridgeAgentExecutorAddress) revert UnrecognizedBridgeAgentExecutor();
    }

```


https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgentExecutor.sol#L53-L56

```solidity
file:  src/BranchBridgeAgentExecutor.sol

53    function executeNoSettlement(address _router, bytes calldata _payload) external payable onlyOwner {
        // Execute Calldata if there is code in the destination router
        IRouter(_router).executeNoSettlement{value: msg.value}(_payload[PARAMS_TKN_START_SIGNED:]);
    }

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L243-L246

```solidity
file:  src/RootBridgeAgentExecutor.sol

243     function _bridgeIn(address _recipient, DepositParams memory _dParams, uint16 _srcChainId) internal {
        //Request assets for decoded request.
        RootBridgeAgent(payable(msg.sender)).bridgeIn(_recipient, _dParams, _srcChainId);
    }

```

## [G-02] Repeatedly doing the same operation (addition) involving a state variable should be avoided

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L201-L235

Gas saved on average: 548 

```solidity
file:    src/RootBridgeAgentExecutor.sol

201       function executeSignedWithDepositMultiple(
        address _account,
        address _router,
        bytes calldata _payload,
        uint16 _srcChainId
    ) external payable onlyOwner {
        //Bridge In Assets
        DepositMultipleParams memory dParams = _bridgeInMultiple(
            _account,
            _payload[
                PARAMS_START_SIGNED:
                    PARAMS_END_SIGNED_OFFSET
                        + uint256(uint8(bytes1(_payload[PARAMS_START_SIGNED]))) * PARAMS_TKN_SET_SIZE_MULTIPLE
            ],
            _srcChainId
        );

        // Check if there is additional calldata in the payload
        if (
            _payload.length
                > PARAMS_END_SIGNED_OFFSET
                    + uint256(uint8(bytes1(_payload[PARAMS_START_SIGNED]))) * PARAMS_TKN_SET_SIZE_MULTIPLE
        ) {
            //Execute remote request
            IRouter(_router).executeSignedDepositMultiple{value: msg.value}(
                _payload[
                    PARAMS_END_SIGNED_OFFSET
                        + uint256(uint8(bytes1(_payload[PARAMS_START_SIGNED]))) * PARAMS_TKN_SET_SIZE_MULTIPLE:
                ],
                dParams,
                _account,
                _srcChainId
            );
        }
    }

```


In the function above, the operation  PARAMS_START_SIGNED:
                    PARAMS_END_SIGNED_OFFSET + uint256(uint8(bytes1(_payload[PARAMS_START_SIGNED]))) * PARAMS_TKN_SET_SIZE_MULTIPLE is being done repeatedly. the amount of gas becomes too much. We should do the operation and cache the result in a memory variable.


## [G-03] Use the cached value instead of fetching a storage value

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L144-L157

### BranchPort.sol.manage(): getStrategyTokenDebt[_token] has already been cached and thus the cached value should be used. Saves ~200 gas (2 SLOADS)

```solidity
file:   src/BranchPort.sol

144      function manage(address _token, uint256 _amount) external override requiresPortStrategy(_token) {
        // Cache Strategy Token Global Debt
        uint256 _strategyTokenDebt = getStrategyTokenDebt[_token];

        // Check if request would surpass the tokens minimum reserves
        if (_amount > _excessReserves(_strategyTokenDebt, _token)) revert InsufficientReserves();

        // Check if request would surpass the Port Strategy's daily limit
        _checkTimeLimit(_token, _amount);

        // Update Strategy Token Global Debt
        getStrategyTokenDebt[_token] = _strategyTokenDebt + _amount;
                // Update Port Strategy Token Debt
        getPortStrategyTokenDebt[msg.sender][_token] += _amount;

188        function replenishReserves(address _strategy, address _token) external override lock {
        // Cache Strategy Token Global Debt
        uint256 strategyTokenDebt = getStrategyTokenDebt[_token];

        // Get current balance of _token
        uint256 currBalance = ERC20(_token).balanceOf(address(this));

        // Get reserves lacking
        uint256 reservesLacking = _reservesLacking(strategyTokenDebt, _token, currBalance);

        // Cache Port Strategy Token Debt
        uint256 portStrategyTokenDebt = getPortStrategyTokenDebt[_strategy][_token];

        // Calculate amount to withdraw. The lesser of reserves lacking or Strategy Token Global Debt.
        uint256 amountToWithdraw = portStrategyTokenDebt < reservesLacking ? portStrategyTokenDebt : reservesLacking;

        // Update Port Strategy Token Debt
        getPortStrategyTokenDebt[_strategy][_token] = portStrategyTokenDebt - amountToWithdraw;
        // Update Strategy Token Global Debt
        getStrategyTokenDebt[_token] = strategyTokenDebt - amountToWithdraw;


```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L485-L494

### BranchPort.sol._checkTimeLimit(): strategyDailyLimitRemaining[msg.sender][_token]; has already been cached and thus the cached value should be used. Saves ~200 gas (2 SLOADS)

```solidity
file: src/BranchPort.sol

485   function _checkTimeLimit(address _token, uint256 _amount) internal {
        uint256 dailyLimit = strategyDailyLimitRemaining[msg.sender][_token];
        if (block.timestamp - lastManaged[msg.sender][_token] >= 1 days) {
            dailyLimit = strategyDailyLimitAmount[msg.sender][_token];
            unchecked {
                lastManaged[msg.sender][_token] = (block.timestamp / 1 days) * 1 days;
            }
        }
        strategyDailyLimitRemaining[msg.sender][_token] = dailyLimit - _amount;
    }

```

## [G-04] Unused initialization of variable inside they function 

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1045-L1114

### they initialization of settlementNonce = _settlementNonce + 1; is not used by they function Remove this initialization to save gas 

```solidity
file:   src/RootBridgeAgent.sol

1045      function _createSettlementMultiple(
        uint32 _settlementNonce,
        address payable _refundee,
        address _recipient,
        uint16 _dstChainId,
        address[] memory _globalAddresses,
        uint256[] memory _amounts,
        uint256[] memory _deposits,
        bytes memory _params,
        bool _hasFallbackToggled
    ) internal returns (bytes memory _payload) {
        // Check if valid length
        if (_globalAddresses.length > MAX_TOKENS_LENGTH) revert InvalidInputParamsLength();

        // Check if valid length
        if (_globalAddresses.length != _amounts.length) revert InvalidInputParamsLength();
        if (_amounts.length != _deposits.length) revert InvalidInputParamsLength();

        //Update Settlement Nonce
        settlementNonce = _settlementNonce + 1;

        // Create Arrays
        address[] memory hTokens = new address[](_globalAddresses.length);
        address[] memory tokens = new address[](_globalAddresses.length);

        for (uint256 i = 0; i < hTokens.length;) {
            // Populate Addresses for Settlement
            hTokens[i] = IPort(localPortAddress).getLocalTokenFromGlobal(_globalAddresses[i], _dstChainId);
            tokens[i] = IPort(localPortAddress).getUnderlyingTokenFromLocal(hTokens[i], _dstChainId);

            // Avoid stack too deep
            uint16 destChainId = _dstChainId;

            // Update State to reflect bridgeOut
            _updateStateOnBridgeOut(
                msg.sender, _globalAddresses[i], hTokens[i], tokens[i], _amounts[i], _deposits[i], destChainId
            );

            unchecked {
                ++i;
            }
        }

        // Prepare data for call with settlement of multiple assets
        _payload = abi.encodePacked(
            _hasFallbackToggled ? bytes1(0x02) & 0x0F : bytes1(0x02),
            _recipient,
            uint8(hTokens.length),
            _settlementNonce,
            hTokens,
            tokens,
            _amounts,
            _deposits,
            _params
        );

        // Create and Save Settlement
        // Get storage reference
        Settlement storage settlement = getSettlement[_settlementNonce];

        // Update Setttlement
        settlement.owner = _refundee;
        settlement.recipient = _recipient;
        settlement.hTokens = hTokens;
        settlement.tokens = tokens;
        settlement.amounts = _amounts;
        settlement.deposits = _deposits;
        settlement.dstChainId = _dstChainId;
        settlement.status = STATUS_SUCCESS;
    }

966       function _createSettlement(
        uint32 _settlementNonce,
        address payable _refundee,
        address _recipient,
        uint16 _dstChainId,
        bytes memory _params,
        address _globalAddress,
        uint256 _amount,
        uint256 _deposit,
        bool _hasFallbackToggled
    ) internal returns (bytes memory _payload) {
        // Update Settlement Nonce
        settlementNonce = _settlementNonce + 1;

        // Get Local Branch Token Address from Root Port
        address localAddress = IPort(localPortAddress).getLocalTokenFromGlobal(_globalAddress, _dstChainId);

        // Get Underlying Token Address from Root Port
        address underlyingAddress = IPort(localPortAddress).getUnderlyingTokenFromLocal(localAddress, _dstChainId);

        //Update State to reflect bridgeOut
        _updateStateOnBridgeOut(
            msg.sender, _globalAddress, localAddress, underlyingAddress, _amount, _deposit, _dstChainId
        );

        // Prepare data for call
        _payload = abi.encodePacked(
            _hasFallbackToggled ? bytes1(0x81) : bytes1(0x01),
            _recipient,
            _settlementNonce,
            localAddress,
            underlyingAddress,
            _amount,
            _deposit,
            _params
        );

        // Avoid stack too deep
        uint32 cachedSettlementNonce = _settlementNonce;

        // Get Auxiliary Dynamic Arrays
        address[] memory addressArray = new address[](1);
        uint256[] memory uintArray = new uint256[](1);

        // Get storage reference for new Settlement
        Settlement storage settlement = getSettlement[cachedSettlementNonce];

        // Update Setttlement
        settlement.owner = _refundee;
        settlement.recipient = _recipient;

        addressArray[0] = localAddress;
        settlement.hTokens = addressArray;

        addressArray[0] = underlyingAddress;
        settlement.tokens = addressArray;

        uintArray[0] = _amount;
        settlement.amounts = uintArray;

        uintArray[0] = _deposit;
        settlement.deposits = uintArray;

        settlement.dstChainId = _dstChainId;
        settlement.status = STATUS_SUCCESS;
    }
```


## [G-05] Optimizing the check order for cost efficient function execution

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas) in a function that may ultimately revert, in the unhappy case.

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L434-L439

### first check they if (deposit.status == STATUS_SUCCESS) revert DepositRedeemUnavailable() to  Validate parameters before doing any sstores (more than 2200 gas could be saved in case of a revert)

```solidity
file:   src/BranchBridgeAgent.sol

434       function redeemDeposit(uint32 _depositNonce) external override lock {
        // Get storage reference
        Deposit storage deposit = getDeposit[_depositNonce];

        // Check Deposit
        if (deposit.status == STATUS_SUCCESS) revert DepositRedeemUnavailable();

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L587-L646

### first check they  if (executionState[nonce] != STATUS_READY) revert AlreadyExecutedTransaction(); to Validate parameters before doing any sstores (more than 2200 gas could be saved in case of a revert)

```solidity
file:   src/BranchBridgeAgent.sol

587         function lzReceiveNonBlocking(address _endpoint, bytes calldata _srcAddress, bytes calldata _payload)
        public
        override
        requiresEndpoint(_endpoint, _srcAddress)
    {
        //Save Action Flag
        bytes1 flag = _payload[0] & 0x7F;

        // Save settlement nonce
        uint32 nonce;

        // DEPOSIT FLAG: 0 (No settlement)
        if (flag == 0x00) {
            // Get Settlement Nonce
            nonce = uint32(bytes4(_payload[PARAMS_START_SIGNED:PARAMS_TKN_START_SIGNED]));

            //Check if tx has already been executed
            if (executionState[nonce] != STATUS_READY) revert AlreadyExecutedTransaction();

            //Try to execute the remote request
            //Flag 0 - BranchBridgeAgentExecutor(bridgeAgentExecutorAddress).executeNoSettlement(localRouterAddress, _payload)
            _execute(
                nonce,
                abi.encodeWithSelector(
                    BranchBridgeAgentExecutor.executeNoSettlement.selector, localRouterAddress, _payload
                )
            );

            // DEPOSIT FLAG: 1 (Single Asset Settlement)
        } else if (flag == 0x01) {
            // Parse recipient
            address payable recipient = payable(address(uint160(bytes20(_payload[PARAMS_START:PARAMS_START_SIGNED]))));

            // Parse Settlement Nonce
            nonce = uint32(bytes4(_payload[PARAMS_START_SIGNED:PARAMS_TKN_START_SIGNED]));

            //Check if tx has already been executed
            if (executionState[nonce] != STATUS_READY) revert AlreadyExecutedTransaction();

            //Try to execute the remote request
            //Flag 1 - BranchBridgeAgentExecutor(bridgeAgentExecutorAddress).executeWithSettlement(recipient, localRouterAddress, _payload)
            _execute(
                _payload[0] == 0x81,
                nonce,
                recipient,
                abi.encodeWithSelector(
                    BranchBridgeAgentExecutor.executeWithSettlement.selector, recipient, localRouterAddress, _payload
                )
            );

            // DEPOSIT FLAG: 2 (Multiple Settlement)
        } else if (flag == 0x02) {
            // Parse recipient
            address payable recipient = payable(address(uint160(bytes20(_payload[PARAMS_START:PARAMS_START_SIGNED]))));

            // Parse deposit nonce
            nonce = uint32(bytes4(_payload[22:26]));

            //Check if tx has already been executed
            if (executionState[nonce] != STATUS_READY) revert AlreadyExecutedTransaction();

```


https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L299-L307

### first check they  if (settlement.status == STATUS_SUCCESS) revert SettlementRedeemUnavailable(); to Validate parameters before doing any sstores (more than 2200 gas could be saved in case of a revert)

```solidity
file:   src/RootBridgeAgent.sol

299       function redeemSettlement(uint32 _settlementNonce) external override lock {
        // Get setttlement storage reference
        Settlement storage settlement = getSettlement[_settlementNonce];

        // Get deposit owner.
        address settlementOwner = settlement.owner;

        // Check if Settlement is redeemable.
        if (settlement.status == STATUS_SUCCESS) revert SettlementRedeemUnavailable();

```

## [G-06] Do not cache immutable values

https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol#L50-57

```solidity
file:   src/ArbitrumBranchPort.sol

50      function depositToPort(address _depositor, address _recipient, address _underlyingAddress, uint256 _deposit)
        external
        override
        lock
        requiresBridgeAgent
    {
        // Save root port address to memory
        address _rootPortAddress = rootPortAddress;

73      function withdrawFromPort(address _depositor, address _recipient, address _globalAddress, uint256 _amount)
        external
        override
        lock
        requiresBridgeAgent
    {
        // Save root port address to memory
        address _rootPortAddress = rootPortAddress;

```

## [G-07] Use != 0 instead of > 0 for unsigned integer comparison

It’s generally more gas-efficient to use != 0 instead of > 0 when comparing unsigned integer types.

This is because the Solidity compiler can optimize the != 0 comparison to a simple bitwise operation, while the > 0 comparison requires an additional subtraction operation. As a result, using != 0 can be more gas-efficient and can help to reduce the overall cost of your contract.

https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol#L127

```solidity
File: src/ArbitrumBranchPort.sol

127:         if (_deposit > 0) {

132:         if (_amount - _deposit > 0) {

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L165

```solidity
File: src/BaseBranchRouter.sol

165:         if (_amount - _deposit > 0) {

173:         if (_deposit > 0) {


```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L906

```solidity

File: src/BranchBridgeAgent.sol

906:         if (_amount - _deposit > 0) {

912:         if (_deposit > 0) {

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L259

```solidity
File: src/BranchPort.sol

259:             if (_amounts[i] - _deposits[i] > 0) {

266:             if (_deposits[i] > 0) {

525:         if (_hTokenAmount > 0) {

531:         if (_deposit > 0) {


```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L363

```solidity
File: src/RootBridgeAgent.sol

363:         if (_dParams.amount > 0) {

370:         if (_dParams.deposit > 0) {

1146:        if (_underlyingAddress == address(0)) if (_deposit > 0) revert UnrecognizedUnderlyingAddress();

1149:        if (_amount - _deposit > 0) {

1156:        if (_deposit > 0) {



```
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L284

```solidity
File: src/RootPort.sol

284:         if (_amount - _deposit > 0) {

290:         if (_deposit > 0) if (!ERC20hTokenRoot(_hToken).mint(_recipient, _deposit, _srcChainId)) revert UnableToMint();


```
https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L152

```solidity
File: src/VirtualAccount.sol

152:         return size > 0;

```

## [G-08] Can Make The Variable Outside The Loop To Save Gas



When you declare a variable inside a loop, Solidity creates a new instance of the variable for each iteration of the loop. This can lead to unnecessary gas costs; especially if the loop is executed frequently or iterates over a large number of elements.

By declaring the variable outside of the loop, you can avoid the creation of multiple instances of the variable and reduce the gas cost of your contract.

Here’s an example:

```solidity
contract MyContract {
    function sum(uint256[] memory values) public pure returns (uint256) {
        uint256 total = 0;
        
        for (uint256 i = 0; i < values.length; i++) {
            total += values[i];
        }
        
        return total;
    }
}

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L318-L340

```solidity
file:   src/RootBridgeAgent.sol

318     for (uint256 i = 0; i < settlement.hTokens.length;) {
            // Save to memory
            address _hToken = settlement.hTokens[i];

            // Check if asset
            if (_hToken != address(0)) {
                // Save to memory
                uint24 _dstChainId = settlement.dstChainId;

                // Move hTokens from Branch to Root + Mint Sufficient hTokens to match new port deposit
                IPort(localPortAddress).bridgeToRoot(
                    msg.sender,
                    IPort(localPortAddress).getGlobalTokenFromLocal(_hToken, _dstChainId),
                    settlement.amounts[i],
                    settlement.deposits[i],
                    _dstChainId
                );
            }

            unchecked {
                ++i;
            }
        }

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1070-1086

```solidity
file:  src/RootBridgeAgent.sol

1070    for (uint256 i = 0; i < hTokens.length;) {
            // Populate Addresses for Settlement
            hTokens[i] = IPort(localPortAddress).getLocalTokenFromGlobal(_globalAddresses[i], _dstChainId);
            tokens[i] = IPort(localPortAddress).getUnderlyingTokenFromLocal(hTokens[i], _dstChainId);

            // Avoid stack too deep
            uint16 destChainId = _dstChainId;

            // Update State to reflect bridgeOut
            _updateStateOnBridgeOut(
                msg.sender, _globalAddresses[i], hTokens[i], tokens[i], _amounts[i], _deposits[i], destChainId
            );

            unchecked {
                ++i;
            }
        }

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L513-L529

```solidity
file:   src/BranchBridgeAgent.sol

513       for (uint256 i = 0; i < numOfAssets;) {
            // Cache common offset
            uint256 currentIterationOffset = PARAMS_START + i;

            // Parse Params
            _hTokens[i] = address(
                uint160(
                    bytes20(
                        bytes32(
                            _sParams[
                                PARAMS_TKN_START + (PARAMS_ENTRY_SIZE * i) + ADDRESS_END_OFFSET:
                                    PARAMS_TKN_START + (PARAMS_ENTRY_SIZE * currentIterationOffset)
                            ]
                        )
                    )
                )
            );

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L70-L81

```solidity
file:   src/VirtualAccount.sol

70    for (uint256 i = 0; i < length;) {
            bool success;
            Call calldata _call = calls[i];

            if (isContract(_call.target)) (success, returnData[i]) = _call.target.call(_call.callData);

            if (!success) revert CallFailed();

            unchecked {
                ++i;
            }
        }

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L90-L108

```solidity
file:   src/VirtualAccount.sol

90    for (uint256 i = 0; i < length;) {
            _call = calls[i];
            uint256 val = _call.value;
            // Humanity will be a Type V Kardashev Civilization before this overflows - andreas
            // ~ 10^25 Wei in existence << ~ 10^76 size uint fits in a uint256
            unchecked {
                valAccumulator += val;
            }

            bool success;

            if (isContract(_call.target)) (success, returnData[i]) = _call.target.call{value: val}(_call.callData);

            if (!success) revert CallFailed();

            unchecked {
                ++i;
            }
        }

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L218-L296

```solidity
file:  src/RootBridgeAgentExecutor.sol

218          for (uint256 i = 0; i < uint256(uint8(numOfAssets));) {
            // Cache offset
            uint256 currentIterationOffset = PARAMS_START + i;

            // Parse Params
            hTokens[i] = address(
                uint160(
                    bytes20(
                        bytes32(
                            _dParams[
                                PARAMS_TKN_START + (PARAMS_ENTRY_SIZE * i) + ADDRESS_END_OFFSET:
                                    PARAMS_TKN_START + (PARAMS_ENTRY_SIZE * currentIterationOffset)
                            ]
                        )
                    )
                )

```

## [G-09] Gas saving is achieved by removing the delete keyword (~60k)

30k gas savings were made by removing the delete keyword. The reason for using the delete keyword here is to reset the struct values (set to default value) in every operation. However, the struct values do not need to be zero each time the function is run. Therefore, the delete keyword is unnecessary. If it is removed, around 30k gas savings will be achieved.

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L456

```solidity
File: src/BranchBridgeAgent.sol

456    delete getDeposit[_depositNonce];

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L343

```solidity
File: src/RootBridgeAgent.sol

343    delete getSettlement[_settlementNonce];


```

## [G-10] With assembly, .call (bool success) transfer can be done gas-optimized

Return data (bool success,) has to be stored due to EVM architecture. But in a usage like below, ‘out’ and ‘outsize’ values are given (0,0); this storage disappears and gas optimization is provided.

(bool success,) = dest.call{value:amount}(""); bool success; assembly {
success := call(gas(), dest, amount, 0, 0) }

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L717

```solidity
file:   src/BranchBridgeAgent.sol

717   (bool success,) = bridgeAgentExecutorAddress.call{value: address(this).balance}(_calldata);

773   (bool success,) = bridgeAgentExecutorAddress.call{value: address(this).balance}(_calldata);

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L754

```solidity
file:   src/RootBridgeAgent.sol

754    (bool success,) = bridgeAgentExecutorAddress.call{value: address(this).balance}(_calldata);

779    (bool success,) = bridgeAgentExecutorAddress.call{value: address(this).balance}(_calldata);
```

## [G-11] Duplicated require()/if() checks should be refactored to a modifier or function

Sign modifiers or functions can make your code more gas-efficient by reducing the overall number of operations that need to be executed. For example, if you have a complex validation check that involves multiple operations and you refactor it into a function, then the function can be executed with a single opcode, rather than having to execute each operation separately in multiple locations.

Recommendation: You can consider adding a modifier like below:

```solidity
 modifer check (address checkToAddress) {    
     require(checkToAddress != address(0) && checkToAddress != SENTINEL_MODULES, "BSA101");  
      _; 
 }

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L354

```solidity
file:  src/BranchBridgeAgent.sol

354   if (deposit.owner != msg.sender) revert NotDepositOwner();

441   if (deposit.owner != msg.sender) revert NotDepositOwner();



604   if (executionState[nonce] != STATUS_READY) revert AlreadyExecutedTransaction();

624   if (executionState[nonce] != STATUS_READY) revert AlreadyExecutedTransaction();

646   if (executionState[nonce] != STATUS_READY) revert AlreadyExecutedTransaction();

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L387

```solidity
file:  src/BranchPort.sol

387    if (!isStrategyToken[_token]) revert UnrecognizedStrategyToken();

560    if (!isStrategyToken[_token]) revert UnrecognizedStrategyToken();

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L142

```solidity
file:   src/MulticallRootRouter.sol

142    if (funcId == 0x01) 

234    if (funcId == 0x01)

323    if (funcId == 0x01) 

411    if (funcId == 0x01)
```

```solidity
file:

244    if (settlementReference.owner == address(0)) revert SettlementRetryUnavailable();

670    if (settlementReference.owner == address(0)) revert SettlementRetryUnavailable();



282    if (settlementOwner == address(0)) revert SettlementRetrieveUnavailable();

308    if (settlementOwner == address(0)) revert SettlementRedeemUnavailable();



285    if (msg.sender != settlementOwner) {

311    if (msg.sender != settlementOwner) {

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L449

```solidity
file:  src/RootBridgeAgent.sol

449   if (executionState[_srcChainId][nonce] != STATUS_READY) {

472   if (executionState[_srcChainId][nonce] != STATUS_READY) {

495   if (executionState[_srcChainId][nonce] != STATUS_READY) {

518   if (executionState[_srcChainId][nonce] != STATUS_READY) {

544   if (executionState[_srcChainId][nonce] != STATUS_READY) {

582   if (executionState[_srcChainId][nonce] != STATUS_READY) {

622   if (executionState[_srcChainId][nonce] != STATUS_READY) {



818   if (callee == address(0)) revert UnrecognizedBridgeAgent();

910   if (callee == address(0)) revert UnrecognizedBridgeAgent();



821   if (_dstChainId != localChainId)

913   if (_dstChainId != localChainId)

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L246

```solidity
file:  src/RootPort.sol
 
246   if (_localAddress == address(0)) revert InvalidLocalAddress();

264   if (_localAddress == address(0)) revert InvalidLocalAddress();



282   if (!isGlobalAddress[_hToken]) revert UnrecognizedToken();

299   if (!isGlobalAddress[_hToken]) revert UnrecognizedToken();

309   if (!isGlobalAddress[_hToken]) revert UnrecognizedToken();

320   if (!isGlobalAddress[_hToken]) revert UnrecognizedToken();

330   if (!isGlobalAddress[_hToken]) revert UnrecognizedToken();

341   if (!isGlobalAddress[_hToken]) revert UnrecognizedToken();



528   if (_coreBranchRouter == address(0)) revert InvalidCoreBranchRouter();

544   if (_coreBranchRouter == address(0)) revert InvalidCoreBranchRouter();



529   if (_coreBranchBridgeAgent == address(0)) revert InvalidCoreBrancBridgeAgent();

545   if (_coreBranchBridgeAgent == address(0)) revert InvalidCoreBrancBridgeAgent();


```

## [G-12] Use a hardcoded address instead of address(this)

It can be more gas-efficient to use a hardcoded address instead of the address(this) expression, especially if you need to use the same address multiple times in your contract.

The reason for this is that using address(this) requires an additional EXTCODESIZE operation to retrieve the contract’s address from its bytecode, which can increase the gas cost of your contract. By pre-calculating and using a hardcoded address, you can avoid this additional operation and reduce the overall gas cost of your contract.

Here’s an example of how you can use a hardcoded address instead of address(this):

```solidity
contract MyContract {
    address public myAddress = 0x1234567890123456789012345678901234567890;
    
    function doSomething() public {
        // Use myAddress instead of address(this)
        require(msg.sender == myAddress, "Caller is not authorized");
        
        // Do something
    }
}

```
In the above example, we have a contract, MyContract, with a public address variable myAddress. Instead of using address(this) to retrieve the contract’s address, we have pre-calculated and hardcoded the address in the variable. This can help to reduce the gas cost of our contract and make our code more efficient.

https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchBridgeAgent.sol#L126

```solidity
file: src/ArbitrumBranchBridgeAgent.sol

126   if (msg.sender != address(this)) revert LayerZeroUnauthorizedEndpoint();

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol#L66

```solidity
file:  src/ArbitrumBranchPort.sol

66   _underlyingAddress.safeTransferFrom(_depositor, address(this), _deposit);

128  _underlyingAddress.safeTransferFrom(_depositor, address(this), _deposit);

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L167 

```solidity
file:  src/BaseBranchRouter.sol

167   _hToken.safeTransferFrom(msg.sender, address(this), _amount - _deposit);

174   _token.safeTransferFrom(msg.sender, address(this), _deposit);

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L131

```solidity
file:   src/BranchBridgeAgent.sol

131    require(_localPortAddress != address(0), "Local Port Address cannot be the zero address.");

142    rootBridgeAgentPath = abi.encodePacked(_rootBridgeAgentAddress, address(this));

168    address(this),

579    address(this).excessivelySafeCall(

717    (bool success,) = bridgeAgentExecutorAddress.call{value: address(this).balance}(_calldata);

737    (bool success,) = bridgeAgentExecutorAddress.call{value: address(this).balance}(_calldata);

787    ILayerZeroEndpoint(lzEndpointAddress).send{value: address(this).balance}(

938    if (msg.sender != address(this)) revert LayerZeroUnauthorizedEndpoint();

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgentExecutor.sol#L92

```solidity
file:  src/BranchBridgeAgentExecutor.sol


92    _recipient.safeTransferETH(address(this).balance);

126   _recipient.safeTransferETH(address(this).balance);

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L175

```solidity
file:  src/BranchPort.sol

175    uint256 currBalance = ERC20(_token).balanceOf(address(this));

178    IPortStrategy(msg.sender).withdraw(address(this), _token, _amount);

181    require(ERC20(_token).balanceOf(address(this)) - currBalance == _amount, "Port Strategy Withdraw Failed");

193    uint256 currBalance = ERC20(_token).balanceOf(address(this));

210    IPortStrategy(_strategy).withdraw(address(this), _token, amountToWithdraw);

214    ERC20(_token).balanceOf(address(this)) - currBalance == amountToWithdraw, "Port Strategy Withdraw Failed"

436    uint256 currBalance = ERC20(_token).balanceOf(address(this));

526    _localAddress.safeTransferFrom(_depositor, address(this), _hTokenAmount);

532    _underlyingAddress.safeTransferFrom(_depositor, address(this), _deposit);

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L112

```solidity
file:  src/RootBridgeAgent.sol

112    require(_localPortAddress != address(0), "Port Address cannot be zero address");

120    bridgeAgentExecutorAddress = DeployRootBridgeAgentExecutor.deploy(address(this));

148    address(this),

424    (bool success,) = address(this).excessivelySafeCall(

695    address(this).balance

754    (bool success,) = bridgeAgentExecutorAddress.call{value: address(this).balance}(_calldata);

779    (bool success,) = bridgeAgentExecutorAddress.call{value: address(this).balance}(_calldata);

940    ILayerZeroEndpoint(lzEndpointAddress).send{value: address(this).balance}(

1182   getBranchBridgeAgentPath[_branchChainId] = abi.encodePacked(_newBranchBridgeAgent, address(this));

1205   if (msg.sender != address(this)) revert LayerZeroUnauthorizedEndpoint();

1233   if (msg.sender != IPort(localPortAddress).getBridgeAgentManager(address(this))) {

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L301

```solidity
file:   src/RootPort.sol

301    _hToken.safeTransferFrom(_from, address(this), _amount);

362    newAccount = new VirtualAccount{salt: keccak256(abi.encode(_user))}(_user, address(this));

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L62

```solidity
file:  src/VirtualAccount.sol

62     ERC721(_token).transferFrom(address(this), msg.sender, _tokenId);

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenRootFactory.sol#L83

```solidity
file:  src/factories/ERC20hTokenRootFactory.sol

83     address(this),

```

## [G‑13] Use assembly to write address storage values


https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L62

```solidity
file:  src/BaseBranchRouter.sol

62    localBridgeAgentAddress = _localBridgeAgentAddress;

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L129

```solidity
file:  src/BranchPort.sol

129     coreBranchRouterAddress = _coreBranchRouter;

334     coreBranchRouterAddress = _newCoreRouter;

419     coreBranchRouterAddress = _coreBranchRouter;

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L88

```solidity
file:  src/CoreRootRouter.sol

88    hTokenFactoryAddress = _hTokenFactory;

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenBranchFactory.sol#L74

```solidity
file:  src/factories/ERC20hTokenBranchFactory.sol

74    localCoreRouterAddress = _coreRouter;

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenRootFactory.sol#L51

```solidity
file:  src/factories/ERC20hTokenRootFactory.sol

51    coreRootRouterAddress = _coreRouter;

```

## [G‑14] Use do while loops instead of for loops

A do while loop will cost less gas since the condition is not being checked for the first iteration.

https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L192-L196

```solidity
file:   src/BaseBranchRouter.sol

192    for (uint256 i = 0; i < _hTokens.length;) {
            _transferAndApproveToken(_hTokens[i], _tokens[i], _amounts[i], _deposits[i]);

            unchecked {
                ++i;
            }
        }

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L447-L451

```solidity
file:  src/BranchBridgeAgent.sol

447    for (uint256 i = 0; i < deposit.tokens.length;) {
            _clearToken(msg.sender, deposit.hTokens[i], deposit.tokens[i], deposit.amounts[i], deposit.deposits[i]);

            unchecked {
                ++i;
            }
        }

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L257-L273

```solidity
file:

257   for (uint256 i = 0; i < length;) {
            // Check if hTokens are being bridged in
            if (_amounts[i] - _deposits[i] > 0) {
                unchecked {
                    _bridgeIn(_recipient, _localAddresses[i], _amounts[i] - _deposits[i]);
                }
            }

            // Check if underlying tokens are being cleared
            if (_deposits[i] > 0) {
                withdraw(_recipient, _underlyingAddresses[i], _deposits[i]);
            }

            unchecked {
                ++i;
            }
        }

305   for (uint256 i = 0; i < length;) {
            _bridgeOut(_depositor, _localAddresses[i], _underlyingAddresses[i], _amounts[i], _deposits[i]);

            unchecked {
                i++;
            }
        }

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L278-283

```solidity
file:   src/MulticallRootRouter.sol
  
278     for (uint256 i = 0; i < outputParams.outputTokens.length;) {
                IVirtualAccount(userAccount).withdrawERC20(outputParams.outputTokens[i], outputParams.amountsOut[i]);

                unchecked {
                    ++i;
                }
            }

367     for (uint256 i = 0; i < outputParams.outputTokens.length;) {
                IVirtualAccount(userAccount).withdrawERC20(outputParams.outputTokens[i], outputParams.amountsOut[i]);

                unchecked {
                    ++i;
                }
            }

455    for (uint256 i = 0; i < outputParams.outputTokens.length;) {
                IVirtualAccount(userAccount).withdrawERC20(outputParams.outputTokens[i], outputParams.amountsOut[i]);

                unchecked {
                    ++i;
                }
            }

557    for (uint256 i = 0; i < outputTokens.length;) {
            // Approve Root Port to spend output hTokens.
            outputTokens[i].safeApprove(_bridgeAgentAddress, amountsOut[i]);
            unchecked {
                ++i;
            }
        }

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L70-L81

```solidity
file:  src/VirtualAccount.sol

70     for (uint256 i = 0; i < length;) {
            bool success;
            Call calldata _call = calls[i];

            if (isContract(_call.target)) (success, returnData[i]) = _call.target.call(_call.callData);

            if (!success) revert CallFailed();

            unchecked {
                ++i;
            }
        }

```

## [G-15] Use assembly to make more efficient back-to-back calls

In the instance below, two external calls, both of which take two function parameters, are performed. We can potentially avoid memory expansion costs by using assembly to utilize scratch space + free memory pointer memory offsets for the function calls. We will use the same memory space for both function calls.

Note: In order to do this optimization safely we will cache the free memory pointer value and restore it once we are done with our function calls.

https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L63-L64

### Use assembly for IBridgeAgent back to back call

```solidity
file:    src/BaseBranchRouter.sol

63        localPortAddress = IBridgeAgent(_localBridgeAgentAddress).localPortAddress();
64        bridgeAgentExecutorAddress = IBridgeAgent(_localBridgeAgentAddress).bridgeAgentExecutorAddress();

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L249-L255

### Use assembly for IPort(_localPortAddress) back to back call

```solidity
file:    src/CoreBranchRouter.sol

249     if (IPort(_localPortAddress).isBridgeAgentFactory(_newBridgeAgentFactoryAddress)) {
            // If so, disable it.
            IPort(_localPortAddress).toggleBridgeAgentFactory(_newBridgeAgentFactoryAddress);
        } else {
            // If not, add it.
            IPort(_localPortAddress).addBridgeAgentFactory(_newBridgeAgentFactoryAddress);
        }

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L426-L429

### Use assembly for ERC20(_globalAddress) and IPort(rootPortAddress) back to back call

```solidity
file:   src/CoreRootRouter.sol

426     ERC20(_globalAddress).name(),
        ERC20(_globalAddress).symbol(),
        ERC20(_globalAddress).decimals(),

461     if (IPort(rootPortAddress).isGlobalAddress(_underlyingAddress)) revert TokenAlreadyAdded();
        if (IPort(rootPortAddress).isLocalToken(_underlyingAddress, _srcChainId)) revert TokenAlreadyAdded();
        if (IPort(rootPortAddress).isUnderlyingToken(_underlyingAddress, _srcChainId)) revert TokenAlreadyAdded();

```

## [G-16] Expensive operation inside a for loop

https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L192-L198

```solidity
file:  src/BaseBranchRouter.sol

192    for (uint256 i = 0; i < _hTokens.length;) {
            _transferAndApproveToken(_hTokens[i], _tokens[i], _amounts[i], _deposits[i]);

            unchecked {
                ++i;
            }
        }

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/IBranchBridgeAgent.sol#L447-L453

```solidity
file:  src/interfaces/IBranchBridgeAgent.sol

447   for (uint256 i = 0; i < deposit.tokens.length;) {
            _clearToken(msg.sender, deposit.hTokens[i], deposit.tokens[i], deposit.amounts[i], deposit.deposits[i]);

            unchecked {
                ++i;
            }
        }

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L257-L273

```solidity
file:  src/BranchPort.sol

257    for (uint256 i = 0; i < length;) {
            // Check if hTokens are being bridged in
            if (_amounts[i] - _deposits[i] > 0) {
                unchecked {
                    _bridgeIn(_recipient, _localAddresses[i], _amounts[i] - _deposits[i]);
                }
            }

            // Check if underlying tokens are being cleared
            if (_deposits[i] > 0) {
                withdraw(_recipient, _underlyingAddresses[i], _deposits[i]);
            }

            unchecked {
                ++i;
            }
        }

```

## [G-17] Change public function visibility to external

Since the following public functions are not called from within the contract, their visibility can be made external for gas optimization.

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L578

```solidity
File: src/BranchBridgeAgent.sol

578      function lzReceive(uint16, bytes calldata _srcAddress, uint64, bytes calldata _payload) public override {

587      function lzReceiveNonBlocking(address _endpoint, bytes calldata _srcAddress, bytes calldata _payload)
            public
            override
          requiresEndpoint(_endpoint, _srcAddress)

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L423

```solidity
File: src/RootBridgeAgent.solChange public function visibility to external

423      function lzReceive(uint16 _srcChainId, bytes calldata _srcAddress, uint64, bytes calldata _payload) public {

434       function lzReceiveNonBlocking(
             address _endpoint,
             uint16 _srcChainId,
             bytes calldata _srcAddress,
             bytes calldata _payload
         ) public override requiresEndpoint(_endpoint, _srcChainId, _srcAddress) {

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L85

```solidity
File: src/VirtualAccount.sol

85       function payableCall(PayableCall[] calldata calls) public payable returns (bytes[] memory returnData) {

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/token/ERC20hTokenBranch.sol#L35

```solidity
File: src/token/ERC20hTokenBranch.sol

35       function burn(uint256 amount) public override onlyOwner {

```

## [G-18] Create DepositMultipleInput and GasParams variable  immutable variable to avoid redundant external calls

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L306-L312

```solidity
file:  src/BranchBridgeAgent.sol

306      function callOutSignedAndBridgeMultiple(
        address payable _refundee,
        bytes calldata _params,
        DepositMultipleInput memory _dParams,
        GasParams calldata _gParams,
        bool _hasFallbackToggled
    ) external payable override lock {

343    function retryDeposit(
        bool _isSigned,
        uint32 _depositNonce,
        bytes calldata _params,
        GasParams calldata _gParams,
        bool _hasFallbackToggled
    ) external payable override lock {

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L175

### Create ERC20hToken variable immutable to avoid redundant external calls

```solidity
file:  src/CoreBranchRouter.sol

175    ERC20hToken newToken = ITokenFactory(hTokenFactoryAddress).createToken(_name, _symbol, _decimals, false);

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L202-L210

### Create SettlementMultipleInput, GasParams and Settlement variables immutable to avoid redundant external calls

```solidity
file:   src/RootBridgeAgent.sol

202     function callOutAndBridgeMultiple(
        address payable _refundee,
        address _recipient,
        uint16 _dstChainId,
        bytes calldata _params,
        SettlementMultipleInput calldata _sParams,
        GasParams calldata _gParams,
        bool _hasFallbackToggled
    ) external payable override lock requiresRouter {

301  Settlement storage settlement = getSettlement[_settlementNonce];

```

## [G-19] Use assembly in place of abi.decode to extract calldata values more efficiently

Instead of using abi.decode, we can use assembly to decode our desired calldata values directly. This will allow us to avoid decoding calldata values that we will not use.

https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumCoreBranchRouter.sol#L134

```solidity
file:  src/ArbitrumCoreBranchRouter.sol

134    ) = abi.decode(_data[1:], (address, address, address, address, address, GasParams));

147    (address bridgeAgentFactoryAddress) = abi.decode(_data[1:], (address));

153    (address branchBridgeAgent) = abi.decode(_data[1:], (address));

158    (address underlyingToken, uint256 minimumReservesRatio) = abi.decode(_data[1:], (address, uint256));

164    abi.decode(_data[1:], (address, address, uint256, bool));

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L96

```solidity
file:  src/CoreBranchRouter.sol

96     ) = abi.decode(_params[1:], (address, string, string, uint8, address, GasParams));

108    ) = abi.decode(_params[1:], (address, address, address, address, address, GasParams));

116    (address bridgeAgentFactoryAddress) = abi.decode(_params[1:], (address));

122    (address branchBridgeAgent) = abi.decode(_params[1:], (address));

128    (address underlyingToken, uint256 minimumReservesRatio) = abi.decode(_params[1:], (address, uint256));

135    abi.decode(_params[1:], (address, address, uint256, bool));

141    (address coreBranchRouter, address coreBranchBridgeAgent) = abi.decode(_params[1:], (address, address));

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L309

```solidity
file:  src/CoreRootRouter.sol

309   = abi.decode(_encodedData[1:], (address, address, string, string, uint8));

315   (address globalAddress, address localAddress) = abi.decode(_encodedData[1:], (address, address));

321   (address newBranchBridgeAgent, address rootBridgeAgent) = abi.decode(_encodedData[1:], (address, address));

339   abi.decode(_encodedData[1:], (address, address, uint16, GasParams[2]));

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L144

```solidity
file:   src/MulticallRootRouter.sol
 
144    (IMulticall.Call[] memory callData) = abi.decode(_decode(encodedData[1:]), (IMulticall.Call[]));

157    ) = abi.decode(_decode(encodedData[1:]), (IMulticall.Call[], OutputParams, uint16, GasParams));

181    ) = abi.decode(_decode(encodedData[1:]), (IMulticall.Call[], OutputMultipleParams, uint16, GasParams));

236    Call[] memory calls = abi.decode(_decode(encodedData[1:]), (Call[]));

245    abi.decode(_decode(encodedData[1:]), (Call[], OutputParams, uint16, GasParams));

272    ) = abi.decode(_decode(encodedData[1:]), (Call[], OutputMultipleParams, uint16, GasParams));

325    Call[] memory calls = abi.decode(_decode(encodedData[1:]), (Call[]));

334    abi.decode(_decode(encodedData[1:]), (Call[], OutputParams, uint16, GasParams));

361    ) = abi.decode(_decode(encodedData[1:]), (Call[], OutputMultipleParams, uint16, GasParams));

413    Call[] memory calls = abi.decode(_decode(encodedData[1:]), (Call[]));

422    abi.decode(_decode(encodedData[1:]), (Call[], OutputParams, uint16, GasParams));

449    ) = abi.decode(_decode(encodedData[1:]), (Call[], OutputMultipleParams, uint16, GasParams));

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L664

```solidity
file:  src/RootBridgeAgent.sol

664    (nonce, owner, params, gParams) = abi.decode(_payload[PARAMS_START:], (uint32, address, bytes, GasParams));

```

## [G-20] Use assembly to validate msg.sender

We can use assembly to efficiently validate msg.sender for the didPay and uniswapV3SwapCallback functions with the least amount of opcodes necessary. Additionally, we can use xor() instead of iszero(eq()), saving 3 gas. We can also potentially save gas on the unhappy path by using scratch space to store the error selector, potentially avoiding memory expansion costs.

https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchBridgeAgent.sol#L126

```solidity
file:  src/ArbitrumBranchBridgeAgent.sol

126    if (msg.sender != address(this)) revert LayerZeroUnauthorizedEndpoint();

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L207

```solidity
file:  src/BaseBranchRouter.sol

207   if (msg.sender != bridgeAgentExecutorAddress) revert UnrecognizedBridgeAgentExecutor();

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L247

```solidity
file:  src/RootBridgeAgent.sol

247   if (msg.sender != settlementReference.owner) {

248   if (msg.sender != address(IPort(localPortAddress).getUserAccount(settlementReference.owner))) {

285   if (msg.sender != settlementOwner) {

286   if (msg.sender != address(IPort(localPortAddress).getUserAccount(settlementOwner))) {

311   if (msg.sender != settlementOwner) {

312   if (msg.sender != address(IPort(localPortAddress).getUserAccount(settlementOwner))) {

1199  if (msg.sender != localRouterAddress) revert UnrecognizedRouter();

1205  if (msg.sender != address(this)) revert LayerZeroUnauthorizedEndpoint();

1221  if (msg.sender != bridgeAgentExecutorAddress) revert UnrecognizedExecutor();

1227  if (msg.sender != localPortAddress) revert UnrecognizedPort();

1233  if (msg.sender != IPort(localPortAddress).getBridgeAgentManager(address(this))) {
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L354

```solidity
file:   src/BranchBridgeAgent.sol

354    if (deposit.owner != msg.sender) revert NotDepositOwner();

424    if (getDeposit[_depositNonce].owner != msg.sender) revert NotDepositOwner();

938    if (msg.sender != address(this)) revert LayerZeroUnauthorizedEndpoint();

948    if (msg.sender != localRouterAddress) revert UnrecognizedRouter();

954    if (msg.sender != bridgeAgentExecutorAddress) revert UnrecognizedBridgeAgentExecutor();
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L112

```solidity
file:   src/CoreRootRouter.sol

112    if (msg.sender != IPort(rootPortAddress).getBridgeAgentManager(_rootBridgeAgent)) {

278    require(msg.sender == rootPortAddress, "Only root port can call");

512    if (msg.sender != bridgeAgentExecutorAddress) revert UnrecognizedBridgeAgentExecutor();

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L605

```solidity
file:  src/MulticallRootRouter.sol

605   if (msg.sender != bridgeAgentExecutorAddress) revert UnrecognizedBridgeAgentExecutor();

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L570

```solidity
file: src/RootPort.sol

570   if (msg.sender != coreRootRouterAddress) revert UnrecognizedCoreRootRouter();

576   if (msg.sender != localBranchPortAddress) revert UnrecognizedLocalBranchPort();

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L162

```solidity
file:   src/VirtualAccount.sol

162     if (msg.sender != userAddress) {

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L278-L284

## [G-21] Cache external calls outside of loop to avoid re-calling function on each iteration

Performing STATICCALLs that do not depend on variables incremented in loops should always try to be avoided within the loop. In the following instances, we are able to cache the external calls outside of the loop to save a STATICCALL (100 gas) per loop iteration.

```solidity
file:  src/MulticallRootRouter.sol

278    for (uint256 i = 0; i < outputParams.outputTokens.length;) {
                IVirtualAccount(userAccount).withdrawERC20(outputParams.outputTokens[i], outputParams.amountsOut[i]);

                unchecked {
                    ++i;
                }
            }

367    for (uint256 i = 0; i < outputParams.outputTokens.length;) {
                IVirtualAccount(userAccount).withdrawERC20(outputParams.outputTokens[i], outputParams.amountsOut[i]);

                unchecked {
                    ++i;
                }
            }

455    for (uint256 i = 0; i < outputParams.outputTokens.length;) {
                IVirtualAccount(userAccount).withdrawERC20(outputParams.outputTokens[i], outputParams.amountsOut[i]);

                unchecked {
                    ++i;
                }
            }
    
```

## [G-22] A modifier used only once and not being inherited should be inlined to save gas


https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L553-L556

```solidity
file:  src/BranchPort.sol

553    modifier requiresBridgeAgentFactory() {
        if (!isBridgeAgentFactory[msg.sender]) revert UnrecognizedBridgeAgentFactory();
        _;
    }

559    modifier requiresPortStrategy(address _token) {
        if (!isStrategyToken[_token]) revert UnrecognizedStrategyToken();
        if (!isPortStrategy[msg.sender][_token]) revert UnrecognizedPortStrategy();
        _;
    }

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1226-L1229

```solidity
file:  src/RootBridgeAgent.sol

1226   modifier requiresPort() {
        if (msg.sender != localPortAddress) revert UnrecognizedPort();
        _;
    }

1232   modifier requiresManager() {
        if (msg.sender != IPort(localPortAddress).getBridgeAgentManager(address(this))) {
            revert UnrecognizedBridgeAgentManager();
        }
        _;
    }

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L557-L560

```solidity
file:  src/RootPort.sol

557   modifier requiresBridgeAgentFactory() {
        if (!isBridgeAgentFactory[msg.sender]) revert UnrecognizedBridgeAgentFactory();
        _;
    }


```

https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenBranchFactory.sol#L112-L115

```solidity
file:  src/factories/ERC20hTokenBranchFactory.sol

112    modifier requiresCoreRouter() {
        if (msg.sender != localCoreRouterAddress) revert UnrecognizedCoreRouter();
        _;
    }

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenRootFactory.sol#L96-L103

```solidity
file:  src/factories/ERC20hTokenRootFactory.sol

96    modifier requiresCoreRouterOrPort() {
        if (msg.sender != coreRootRouterAddress) {
            if (msg.sender != rootPortAddress) {
                revert UnrecognizedCoreRouterOrPort();
            }
        }
        _;
    }

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L930-L933

```solidity
file:  src/BranchBridgeAgent.sol

930    modifier requiresEndpoint(address _endpoint, bytes calldata _srcAddress) {
        _requiresEndpoint(_endpoint, _srcAddress);
        _;
    }

947    modifier requiresRouter() {
        if (msg.sender != localRouterAddress) revert UnrecognizedRouter();
        _;
    }

```

## [G-23] Not using the named return variable when a function returns, wastes deployment gas only in view function 


https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L145

```solidity
file:  src/RootBridgeAgent.sol

145    ) external view returns (uint256 _fee) {

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/IRootBridgeAgent.sol#L104

```solidity
file:  src/interfaces/IRootBridgeAgent.sol

104   function settlementNonce() external view returns (uint32 nonce);

154   ) external view returns (uint256 _fee);

```

## [G-24] Uncheck arithmetics operations that can’t underflow/overflow

Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block

#### Replace this:

```solidity
file:

 uint256 value = yield(a, b, c - totalFee(c), address(this));

```

#### with this:

```solidity

 unchecked {uint256 value = yield(a, b, c - totalFee(c), address(this));}

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L169

```solidity
file:  src/BranchPort.sol

169    getPortStrategyTokenDebt[msg.sender][_token] -= _amount;

172    getStrategyTokenDebt[_token] -= _amount;

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/token/ERC20hTokenRoot.sol#L70

```solidity
file:  src/token/ERC20hTokenRoot.sol

70     getTokenBalance[chainId] -= amount;

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L157

```solidity
file:  src/BranchPort.sol

157    getPortStrategyTokenDebt[msg.sender][_token] += _amount;

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/token/ERC20hTokenRoot.sol#L58

```solidity
file:  src/token/ERC20hTokenRoot.sol

58    getTokenBalance[chainId] += amount;

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L205

```solidity
file:   src/BranchPort.sol

205     getPortStrategyTokenDebt[_strategy][_token] = portStrategyTokenDebt - amountToWithdraw;

207     getStrategyTokenDebt[_token] = strategyTokenDebt - amountToWithdraw;

493     strategyDailyLimitRemaining[msg.sender][_token] = dailyLimit - _amount; 

522     uint256 _hTokenAmount = _amount - _deposit;

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L515

```solidity
file:  src/BranchBridgeAgent.sol

515    uint256 currentIterationOffset = PARAMS_START + i;

821    depositNonce = _depositNonce + 1;

875    depositNonce = _depositNonce + 1;

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgentExecutor.sol#L111

```solidity
file:  src/BranchBridgeAgentExecutor.sol

111    uint256 settlementEndOffset = PARAMS_START_SIGNED + PARAMS_TKN_START + assetsOffset;

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L155

```solidity
file:  src/BranchPort.sol

155    getStrategyTokenDebt[_token] = _strategyTokenDebt + _amount;

```