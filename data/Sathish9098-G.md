# GAS OPTIMIZATIONS

## [G-] State variables can be packed into fewer storage slots	

If variables occupying the same slot are both written the same function or by the constructor, avoids a separate Gsset (20000 gas). Reads of the variables can also be cheaper

### ``_unlocked`` , ``bridgeAgentExecutorAddress`` can be packed same SLOT : Saves ``2000 GAS`` , ``1 SLOT``

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/MulticallRootRouter.sol#L73-L80

``_unlocked`` variable only stores the values ``1`` and ``2`` . So this can be declared as uint96 instead of uint256 to save one storage slot .


```diff
FILE: 2023-09-maia/src/MulticallRootRouter.sol

73:   /// @notice Bridge Agent to manage communications and cross-chain assets.
74:    address payable public bridgeAgentAddress;
75:
76:    /// @notice Bridge Agent Executor Address.
77:    address public bridgeAgentExecutorAddress;
78:
79:    /// @notice Re-entrancy lock modifier state.
+ 80:    uint96 internal _unlocked = 1;
- 80:    uint256 internal _unlocked = 1;

```	

### ``settlementNonce`` and ``_unlocked `` can be packed same SLOT : Saves ``2000 GAS`` , ``1 SLOT``

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L75-L92

``_unlocked`` variable only stores the values ``1`` and ``2`` . So this can be declared as uint96 instead of uint256 to save one storage slot .

```diff
FILE: 2023-09-maia/src/RootBridgeAgent.sol

  /// @notice Deposit nonce used for identifying transaction.
-    uint32 public settlementNonce;

    /// @notice Mapping from Settlement nonce to Settlement Struct.
    mapping(uint256 nonce => Settlement settlementInfo) public getSettlement;

    /*///////////////////////////////////////////////////////////////
                            EXECUTOR STATE
    //////////////////////////////////////////////////////////////*/

    /// @notice If true, bridge agent has already served a request with this nonce from  a given chain. Chain -> Nonce -> Bool
    mapping(uint256 chainId => mapping(uint256 nonce => uint256 state)) public executionState;

    /*///////////////////////////////////////////////////////////////
                            REENTRANCY STATE
    //////////////////////////////////////////////////////////////*/

    /// @notice Re-entrancy lock modifier state.
+    uint32 public settlementNonce;
-    uint256 internal _unlocked = 1;
+    uint96 internal _unlocked = 1;

```	

### ``coreBranchRouterAddress`` , ``_unlocked `` can be packed with same SLOT : Saves ``2000 GAS`` , ``1 SLOT``

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L91

``_unlocked`` variable only stores the values ``1`` and ``2`` . So this can be declared as uint96 instead of uint256 to save one storage slot .

Some of the lines trimmed for better undertandings

```diff
FILE: Breadcrumbs2023-09-maia/src/BranchPort.sol

/// @notice Local Core Branch Router Address.
25:     address public coreBranchRouterAddress;
+ 91:     uint96 internal _unlocked = 1;
/// @notice Mapping returns the amount of a Strategy Token a given Port Strategy can manage.
    mapping(address strategy => mapping(address token => uint256 dailyLimitRemaining)) public
        strategyDailyLimitRemaining;

    /*///////////////////////////////////////////////////////////////
                            REENTRANCY STATE
    //////////////////////////////////////////////////////////////*/

    /// @notice Reentrancy lock guard state.
- 91:     uint256 internal _unlocked = 1;

```

### ``depositNonce`` and ``_unlocked``  should be packed same SLOT : Saves ``2000 GAS`` , ``1 SLOT``

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L84-L101

``_unlocked`` variable only stores the values ``1`` and ``2`` . So this can be declared as uint96 instead of uint256 to save one storage slot .

```diff
FILE: 2023-09-maia/src/BranchBridgeAgent.sol

 /// @notice Deposit nonce used for identifying the transaction.
- 84:    uint32 public depositNonce;

    /// @notice Mapping from Pending deposits hash to Deposit Struct.
87:    mapping(uint256 depositNonce => Deposit depositInfo) public getDeposit;

    /*///////////////////////////////////////////////////////////////
                        SETTLEMENT EXECUTION STATE
    //////////////////////////////////////////////////////////////*/

    /// @notice If true, the bridge agent has already served a request with this nonce from a given chain.
94:    mapping(uint256 settlementNonce => uint256 state) public executionState;

    /*///////////////////////////////////////////////////////////////
                           REENTRANCY STATE
    //////////////////////////////////////////////////////////////*/

    /// @notice Re-entrancy lock modifier state.
+ 84:    uint32 public depositNonce;
- 101:    uint256 internal _unlocked = 1;
+ 101:    uint96 internal _unlocked = 1;

```

###  localPortAddress and _unlocked  can be packed with same SLOT : Saves ``2000 GAS`` , ``1 SLOT``

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BaseBranchRouter.sol#L32-L42

``_unlocked`` variable only stores the values ``1`` and ``2`` . So this can be declared as uint96 instead of uint256 to save one storage slot .

```diff
FILE: Breadcrumbs2023-09-maia/src/BaseBranchRouter.sol

 /// @inheritdoc IBranchRouter
- 33:    address public localPortAddress;

    /// @inheritdoc IBranchRouter
36:    address public override localBridgeAgentAddress;

    /// @inheritdoc IBranchRouter
39:    address public override bridgeAgentExecutorAddress;

    /// @notice Re-entrancy lock modifier state.
+ 33:    address public localPortAddress;
- 42:    uint256 internal _unlocked = 1;
+ 42:    uint96 internal _unlocked = 1;

```

##

## [G-2] Function _minimumReserves() can be more optimized

Saves 800 GAS as per tests . Gas increases as per complexity 

The case where the ``getMinimumTokenReserveRatio[_token]`` is not checked can be optimized to save gas. If the getMinimumTokenReserveRatio[_token] is 0, then the expression (_currBalance + _strategyTokenDebt) * getMinimumTokenReserveRatio[_token]) / DIVISIONER will always return 0.

To avoid this unnecessary computation, we can check the value of getMinimumTokenReserveRatio[_token] before performing the calculation. If the getMinimumTokenReserveRatio[_token] is 0, then we can simply return 0

```diff
FILE: 2023-09-maia/src/BranchPort.sol

468: function _minimumReserves(uint256 _strategyTokenDebt, uint256 _currBalance, address _token)
468:        internal
469:        view
470:        returns (uint256)
+      if(getMinimumTokenReserveRatio[_token] == 0){
+       return 0;
+       }

471:    {
472:        return ((_currBalance + _strategyTokenDebt) * getMinimumTokenReserveRatio[_token]) / DIVISIONER;
473:    }

```

##

## [G-] Removing Redundant Cache for strategyDailyLimitRemaining[msg.sender][_token] saves gas 

Saves 100 Gas

The ``dailyLimit`` value is not utilized within the ``_checkTimeLimit`` ``IF`` block. The cache strategyDailyLimitRemaining[msg.sender][_token] is redundant.

```diff
FILE: Breadcrumbs2023-09-maia/src/BranchPort.sol

 function _checkTimeLimit(address _token, uint256 _amount) internal {
        uint256 dailyLimit = strategyDailyLimitRemaining[msg.sender][_token];
        if (block.timestamp - lastManaged[msg.sender][_token] >= 1 days) {
-            dailyLimit = strategyDailyLimitAmount[msg.sender][_token];
            unchecked {
                lastManaged[msg.sender][_token] = (block.timestamp / 1 days) * 1 days;
            }
        }
        strategyDailyLimitRemaining[msg.sender][_token] = dailyLimit - _amount;
    }

```
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L485-L494

##

## [G-] Storage variables should be cached in stack variables rather than re-reading them from storage

The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

### ``deposit.hTokens.length``,``deposit.owner``,``executionState[nonce]`` should be cached  : Saves ``700 GAS`` ,``7 SLOD``

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L359

```diff
FILE: 2023-09-maia/src/BranchBridgeAgent.sol

 //Encode Data for cross-chain call.
        bytes memory payload;
+  uint256 length_ = deposit.hTokens.length ;
-        if (uint8(deposit.hTokens.length) == 1) {
+        if (uint8(length_) == 1) {
            if (_isSigned) {
                //Pack new Data
                payload = abi.encodePacked(
                    _hasFallbackToggled ? bytes1(0x85) : bytes1(0x05),
                    msg.sender,
                    _depositNonce,
                    deposit.hTokens[0],
                    deposit.tokens[0],
                    deposit.amounts[0],
                    deposit.deposits[0],
                    _params
                );
            } else {
                payload = abi.encodePacked(
                    bytes1(0x02),
                    _depositNonce,
                    deposit.hTokens[0],
                    deposit.tokens[0],
                    deposit.amounts[0],
                    deposit.deposits[0],
                    _params
                );
            }
-        } else if (uint8(deposit.hTokens.length) > 1) {
+        } else if (uint8(length_) > 1) {
            if (_isSigned) {
                //Pack new Data
                payload = abi.encodePacked(
                    _hasFallbackToggled ? bytes1(0x86) : bytes1(0x06),
                    msg.sender,
-                    uint8(deposit.hTokens.length),
+                    uint8(length_),
                    _depositNonce,
                    deposit.hTokens,
                    deposit.tokens,
                    deposit.amounts,
                    deposit.deposits,
                    _params
                );
            } else {
                payload = abi.encodePacked(
                    bytes1(0x03),
-                    uint8(deposit.hTokens.length),
+                    uint8(length_),
                    _depositNonce,
                    deposit.hTokens,
                    deposit.tokens,
                    deposit.amounts,
                    deposit.deposits,
                    _params
                );
            }
        }


439: if (deposit.status == STATUS_SUCCESS) revert DepositRedeemUnavailable();
+ address _owner = deposit.owner ;
- 440 if (deposit.owner == address(0)) revert DepositRedeemUnavailable();
+ 440 if (_owner  == address(0)) revert DepositRedeemUnavailable();
- 441:        if (deposit.owner != msg.sender) revert NotDepositOwner();
+ 441:        if (_owner != msg.sender) revert NotDepositOwner();


+   uint256 nonce_ = executionState[nonce] ;
// DEPOSIT FLAG: 0 (No settlement)
        if (flag == 0x00) {
            // Get Settlement Nonce
            nonce = uint32(bytes4(_payload[PARAMS_START_SIGNED:PARAMS_TKN_START_SIGNED]));

            //Check if tx has already been executed
-            if (executionState[nonce] != STATUS_READY) revert AlreadyExecutedTransaction();
+            if (nonce_ != STATUS_READY) revert AlreadyExecutedTransaction();

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
-            if (executionState[nonce] != STATUS_READY) revert AlreadyExecutedTransaction();
+            if (nonce_ != STATUS_READY) revert AlreadyExecutedTransaction();

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
-            if (executionState[nonce] != STATUS_READY) revert AlreadyExecutedTransaction();
+            if (_nonce != STATUS_READY) revert AlreadyExecutedTransaction();

            //Try to execute remote request
            // Flag 2 - BranchBridgeAgentExecutor(bridgeAgentExecutorAddress).executeWithSettlementMultiple(recipient, localRouterAddress, _payload)
            _execute(
                _payload[0] == 0x82,
                nonce,
                recipient,
                abi.encodeWithSelector(
                    BranchBridgeAgentExecutor.executeWithSettlementMultiple.selector,
                    recipient,
                    localRouterAddress,
                    _payload
                )
            );

            //DEPOSIT FLAG: 3 (Retrieve Settlement)
        } else if (flag == 0x03) {
            // Parse recipient
            address payable recipient = payable(address(uint160(bytes20(_payload[PARAMS_START:PARAMS_START_SIGNED]))));

            //Get nonce
            nonce = uint32(bytes4(_payload[PARAMS_START_SIGNED:PARAMS_TKN_START_SIGNED]));



  //Check if settlement is in retrieve mode

-            if (executionState[nonce] == STATUS_DONE) {
+            if (nonce_ == STATUS_DONE) {
                revert AlreadyExecutedTransaction();
            } else {
                //Set settlement to retrieve mode, if not already set.
-                if (executionState[nonce] == STATUS_READY) executionState[nonce] = STATUS_RETRIEVE;
+                if (nonce_ == STATUS_READY) executionState[nonce] = STATUS_RETRIEVE;

```

### ``settlementReference.owner`` should be cached  : Saves ``300 GAS`` , ``3 SLOD``


https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L244

```diff
FILE: 2023-09-maia/src/RootBridgeAgent.sol

241: Settlement storage settlementReference = getSettlement[_settlementNonce];
+    address owner_ =  settlementReference.owner ;
        // Check if Settlement hasn't been redeemed.
-        if (settlementReference.owner == address(0)) revert SettlementRetryUnavailable();
+        if (owner_ == address(0)) revert SettlementRetryUnavailable();

        // Check if caller is Settlement owner
-        if (msg.sender != settlementReference.owner) {
+        if (msg.sender != owner_) {
-            if (msg.sender != address(IPort(localPortAddress).getUserAccount(settlementReference.owner))) {
+            if (msg.sender != address(IPort(localPortAddress).getUserAccount(owner_))) {
                revert NotSettlementOwner();
            }
        }

        // Update Settlement Status
        settlementReference.status = STATUS_SUCCESS;

        // Perform Settlement Retry
        _performRetrySettlementCall(
            _hasFallbackToggled,
            settlementReference.hTokens,
            settlementReference.tokens,
            settlementReference.amounts,
            settlementReference.deposits,
            _params,
            _settlementNonce,
-            payable(settlementReference.owner),
+            payable(owner_),
            _recipient,
            settlementReference.dstChainId,
            _gParams,
            msg.value
        );
```

##

## [G-] Cache the state variables outside the loop

Saves ``100 GAS`` for each iterations 

```diff
FILE: 2023-09-maia/src/RootBridgeAgent.sol

+  uint24 _dstChainId = settlement.dstChainId;
317: // Clear Global hTokens To Recipient on Root Chain cancelling Settlement to Branch
        for (uint256 i = 0; i < settlement.hTokens.length;) {
            // Save to memory
            address _hToken = settlement.hTokens[i];

            // Check if asset
            if (_hToken != address(0)) {
                // Save to memory
-                uint24 _dstChainId = settlement.dstChainId;

                // Move hTokens from Branch to Root + Mint Sufficient hTokens to match new port deposit
                IPort(localPortAddress).bridgeToRoot(
                    msg.sender,
                    IPort(localPortAddress).getGlobalTokenFromLocal(_hToken, _dstChainId),
                    settlement.amounts[i],
                    settlement.deposits[i],
                    _dstChainId
                );
            }

##
																															
## [G-] <x> += <y> costs more gas than <x> = <x> + <y> for state variables

Saves ``300 GAS	``

Using the addition operator instead of plus-equals saves [113 gas](https://gist.github.com/IllIllI000/cbbfb267425b898e5be734d4008d4fe8)																								

```solidity
FILE: 2023-09-maia/src/BranchPort.sol

157: getPortStrategyTokenDebt[msg.sender][_token] += _amount;

169: getPortStrategyTokenDebt[msg.sender][_token] -= _amount;

172: getStrategyTokenDebt[_token] -= _amount;

```	
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L157

##

## [G-] Avoid initializing default values for state and parameters to avoid gas

Ethereum smart contracts and gas efficiency, initializing default values can indeed result in unnecessary gas costs. Gas is a limited resource on the Ethereum network, and minimizing gas consumption is crucial to keeping transaction costs low.

```diff
FILE: 2023-09-maia/src/MulticallRootRouter.sol

278: for (uint256 i = 0; i < outputParams.outputTokens.length;) {

367: for (uint256 i = 0; i < outputParams.outputTokens.length;) {

455: for (uint256 i = 0; i < outputParams.outputTokens.length;) {

557: for (uint256 i = 0; i < outputTokens.length;) {

```
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/MulticallRootRouter.sol#L278

```solidity
FILE: 2023-09-maia/src/RootBridgeAgentExecutor.sol

281: for (uint256 i = 0; i < uint256(uint8(numOfAssets));) {

```
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgentExecutor.sol#L281 		

##

## [G-] Using immutable directly is more gas efficient instead of caching

Using immutable directly is more gas efficient than caching. This is because caching requires additional gas to store and retrieve the cached data. Saves ``15 Gas`` per instance 

```diff
FILE: https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumCoreBranchRouter.sol

56: // Save root port address to memory
-        address _rootPortAddress = rootPortAddress;

        // Get global token address from root port
-        address _globalToken = IRootPort(_rootPortAddress).getLocalTokenFromUnderlying(_underlyingAddress, 
+        address _globalToken = IRootPort(rootPortAddress).getLocalTokenFromUnderlying(_underlyingAddress, localChainId);

        // Check if the global token exists
        if (_globalToken == address(0)) revert UnknownGlobalToken();

        // Deposit Assets to Port
        _underlyingAddress.safeTransferFrom(_depositor, address(this), _deposit);

        // Request Minting of Global Token
-        IRootPort(_rootPortAddress).mintToLocalBranch(_recipient, _globalToken, _deposit);
+        IRootPort(rootPortAddress).mintToLocalBranch(_recipient, _globalToken, _deposit);



- 80:  address _rootPortAddress = rootPortAddress;

        // Check if the global token exists
-        if (!IRootPort(_rootPortAddress).isGlobalToken(_globalAddress, localChainId)) revert UnknownGlobalToken();
+        if (!IRootPort(rootPortAddress).isGlobalToken(_globalAddress, localChainId)) revert UnknownGlobalToken();

        // Get the underlying token address from the root port
        address _underlyingAddress =
-            IRootPort(_rootPortAddress).getUnderlyingTokenFromLocal(_globalAddress, localChainId);
+            IRootPort(rootPortAddress).getUnderlyingTokenFromLocal(_globalAddress, localChainId);

        // Check if the underlying token exists
        if (_underlyingAddress == address(0)) revert UnknownUnderlyingToken();

-        IRootPort(_rootPortAddress).burnFromLocalBranch(_depositor, _globalAddress, _amount);
+        IRootPort(rootPortAddress).burnFromLocalBranch(_depositor, _globalAddress, _amount);

        _underlyingAddress.safeTransfer(_recipient, _amount);

```
 																							
		
## [G-] Don't cache variables only used once	

If the variable is only accessed once, it's cheaper to use the state variable directly that one time, and save the 3 gas the extra stack assignment would spend . Saves 35 GAS

###  ``abi.encode`` and ``abi.encodePacked`` are unnecessarily cache : Saves ``350 GAS`` , ``10 Instances ``

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/CoreRootRouter.sol#L167C27-L177

```diff
FILE: 2023-09-maia/src/CoreRootRouter.sol

167:  // Encode CallData
-        bytes memory params = abi.encode(_branchBridgeAgentFactory);

        // Pack funcId into data
-        bytes memory payload = abi.encodePacked(bytes1(0x03), params);

        //Add new global token to branch chain
        IBridgeAgent(bridgeAgentAddress).callOut{value: msg.value}(
-            payable(_refundee), _refundee, _dstChainId, payload, _gParams
+         payable(_refundee), _refundee, _dstChainId, abi.encodePacked(bytes1(0x03), abi.encode(_branchBridgeAgentFactory)), _gParams
        );

 
192: //Encode CallData
-        bytes memory params = abi.encode(_branchBridgeAgent);

        // Pack funcId into data
-        bytes memory payload = abi.encodePacked(bytes1(0x04), params);

        //Add new global token to branch chain
        IBridgeAgent(bridgeAgentAddress).callOut{value: msg.value}(
-            payable(_refundee), _refundee, _dstChainId, payload, _gParams
+            payable(_refundee), _refundee, _dstChainId, abi.encodePacked(bytes1(0x04), abi.encode(_branchBridgeAgent)), _gParams
        );

219: // Encode CallData
-        bytes memory params = abi.encode(_underlyingToken, _minimumReservesRatio);

        // Pack funcId into data
-        bytes memory payload = abi.encodePacked(bytes1(0x05), params);

        //Add new global token to branch chain
        IBridgeAgent(bridgeAgentAddress).callOut{value: msg.value}(
-            payable(_refundee), _refundee, _dstChainId, payload, _gParams
+            payable(_refundee), _refundee, _dstChainId, abi.encodePacked(bytes1(0x05), abi.encode(_underlyingToken, _minimumReservesRatio)), _gParams
        );

250: // Encode CallData
-        bytes memory params = abi.encode(_portStrategy, _underlyingToken, _dailyManagementLimit, _isUpdateDailyLimit);

        // Pack funcId into data
-        bytes memory payload = abi.encodePacked(bytes1(0x06), params);

        //Add new global token to branch chain
        IBridgeAgent(bridgeAgentAddress).callOut{value: msg.value}(
-            payable(_refundee), _refundee, _dstChainId, payload, _gParams
+            payable(_refundee), _refundee, _dstChainId, abi.encodePacked(bytes1(0x06), abi.encode(_portStrategy, _underlyingToken, _dailyManagementLimit, _isUpdateDailyLimit)), _gParams
        );

280: // Encode CallData
-        bytes memory params = abi.encode(_coreBranchRouter, _coreBranchBridgeAgent);

        // Pack funcId into data
-        bytes memory payload = abi.encodePacked(bytes1(0x07), params);

        //Add new global token to branch chain
        IBridgeAgent(bridgeAgentAddress).callOut{value: msg.value}(
-            payable(_refundee), _refundee, _dstChainId, payload, _gParams
+            payable(_refundee), _refundee, _dstChainId, abi.encodePacked(bytes1(0x07), abi.encode(_coreBranchRouter, _coreBranchBridgeAgent)), _gParams
        );

```

### Unnecessary cache : Saves ``40 GAS``

```diff
FILE: 2023-09-maia/src/RootPort.sol

- 195: address globalAddress = getGlobalTokenFromLocal[_localAddress][_srcChainId];
- 196:        return getLocalTokenFromGlobal[globalAddress][_dstChainId];
+ 196:        return getLocalTokenFromGlobal[getGlobalTokenFromLocal[_localAddress][_srcChainId]][_dstChainId];

- 206: address localAddress = getLocalTokenFromGlobal[_globalAddress][_srcChainId];
- 207:        return getUnderlyingTokenFromLocal[localAddress][_srcChainId];
+ 207:        return getUnderlyingTokenFromLocal[getLocalTokenFromGlobal[_globalAddress][_srcChainId]][_srcChainId];

```
##

## [G-] Use assembly to perform efficient back-to-back calls

If a similar external call is performed back-to-back, we can use assembly to reuse any function signatures and function parameters that stay the same. In addition, we can also reuse the same memory space for each function call (``scratch space`` + ``free memory pointer`` + ``zero slot``), which can potentially allow us to avoid memory expansion costs.

```diff
FILE: 2023-09-maia/src/CoreRootRouter.sol

426: ERC20(_globalAddress).name(),
427: ERC20(_globalAddress).symbol(),
428: ERC20(_globalAddress).decimals(),

```																							
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/CoreRootRouter.sol#L426-L428

```diff
FILE: 2023-09-maia/src/ArbitrumCoreBranchRouter.sol

56: string.concat("Arbitrum Ulysses ", ERC20(_underlyingAddress).name()),
57:            string.concat("arb-u", ERC20(_underlyingAddress).symbol()),
58:            ERC20(_underlyingAddress).decimals()

```
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/ArbitrumCoreBranchRouter.sol#L56-L58

```diff
FILE: 2023-09-maia/src/BaseBranchRouter.sol

63: localPortAddress = IBridgeAgent(_localBridgeAgentAddress).localPortAddress();
64: bridgeAgentExecutorAddress = IBridgeAgent(_localBridgeAgentAddress).bridgeAgentExecutorAddress();

```																								
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BaseBranchRouter.sol#L63-L64

```diff
FILE: 2023-09-maia/src/factories/ERC20hTokenBranchFactory.sol

66: ERC20(_wrappedNativeTokenAddress).name(),
67: ERC20(_wrappedNativeTokenAddress).symbol(),
68: ERC20(_wrappedNativeTokenAddress).decimals(),

```	
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/factories/ERC20hTokenBranchFactory.sol#L66-L68																									


##																																											
																									## Don't emit event 

Combine events 

use assembly for loops 

[G-] Modifiers only called once can be inlined																																																																																												
																																																																																																																	