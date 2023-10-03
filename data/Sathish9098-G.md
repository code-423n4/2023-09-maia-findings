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

## [G-] 

			

																																																
																																																																																																																	
Division by two should use bit shifting	<x> / 2 is the same as <x> >> 1. 

While the compiler uses the SHR opcode to accomplish both, the version that uses division incurs an overhead of [20 gas](https://gist.github.com/IllIllI000/ec0e4e6c4f52a6bca158f137a3afd4ff) due to JUMPs to and from a compiler utility function that introduces checks which can be avoided by using unchecked {} around the division by two																								
			

150:              uint256 x = y / 2 + 1;																							
			153:                  x = (y / x + x) / 2;																							
																										

																					
## [G-] Consider using bytes32 rather than a string	

Using the bytes types for fixed-length strings is more efficient than having the EVM have to incur the overhead of string processing. Consider whether the value needs to be a string. A good reason to keep it as a string would be if the variable is defined in an interface that this project does not own.																								
																										
```solidity
FILE: 

```																							
		


keccak256() should only need to be called on a specific string literal once	

It should be saved to an immutable variable, and the variable used instead. If the hash is being used as a part of a function selector, the cast to bytes4 should also only be done once																								
																										
			366                   keccak256(																							
			367                       "onERC1155Received(address,address,uint256,uint256,bytes)"																							
			368:                  )	


Stack variable used as a cheaper cache for a state variable is only used once	

If the variable is only accessed once, it's cheaper to use the state variable directly that one time, and save the 3 gas the extra stack assignment would spend	
																							
			160:          uint256 cachedEpoch = epoch;																							
																										

																					
																										
The result of function calls should be cached rather than re-calling the function	

The instances below point to the second+ call of the function within a single function																								
																										
																										
			/// @audit market.oracle() on line 918																							
			930:          (, info.oracleExchangeRate) = IOracle(market.oracle()).peek(																							
			/// @audit market.oracleData() on line 919																							
			931:              market.oracleData()																							
			/// @audit market.oracle() on line 918																							
			933:          info.spotExchangeRate = IOracle(market.oracle()).peekSpot(																							
			/// @audit market.oracleData() on line 919																							
			934:              market.oracleData()

## 																																											
## internal functions not called by the contract should be removed to save deployment gas	

If the functions are required by an interface, the contract should inherit from that interface and use the override keyword																								
			254       function _getAmountForBorrowPart(																							
			255           uint256 borrowPart																							
			256:      ) internal view returns (uint256) {		


Don't use _msgSender() if not supporting EIP-2771	

Use msg.sender if the code does not implement [EIP-2771 trusted forwarder](https://eips.ethereum.org/EIPS/eip-2771) support																								
																										
Using globals directly is cheaper than assigning them to variables	

Using the global directly saves 5 gas																								
																										
	118:         address liquidityPool = msg.sender;																									
	149:         address liquidityPool = msg.sender;																									
	428:         address liquidityPool = msg.sender;

State variables should be cached in stack variables rather than re-reading them from storage

State variables only set in the constructor should be declared immutable

Is there any possibility refactoring saves gas 	like external calls in the last expensive operations in the last 

Don't emit event 

Combine events 

use assembly for loops 

[G-] Modifiers only called once can be inlined	



Only read state if we are going to execute the entire logic(Saves ~2000 gas for the state read)


The function above, only mints if threshold is met. Before checking the status of whether the threshold is met or not, we making a state read(Transaction memory txn = txnHashToTransaction[txnHash];) which is quite expensive. we then proceed to check if threshold is met and execute the function if threshold is indeed met. If not met, we would not execute anything but we would still have made the state read, as it's being read before the check

We can refactor the function to have the check before to avoid wasting gas in case threshold is not met.


File: /contracts/bridge/DestinationBridge.sol
337:  function _mintIfThresholdMet(bytes32 txnHash) internal {
338:    bool thresholdMet = _checkThresholdMet(txnHash);
339:    Transaction memory txn = txnHashToTransaction[txnHash];
340:    if (thresholdMet) {
341:      _checkAndUpdateInstantMintLimit(txn.amount);
342:      if (!ALLOWLIST.isAllowed(txn.sender)) {
343:        ALLOWLIST.setAccountStatus(
344:          txn.sender,
345:          ALLOWLIST.getValidTermIndexes()[0],
346:          true
347:        );
348:      }
349:      TOKEN.mint(txn.sender, txn.amount);
350:      delete txnHashToTransaction[txnHash];
351:      emit BridgeCompleted(txn.sender, txn.amount);
352:    }
353:  }	

The function should first verify if msg.value > 0 before performing any other operation(Save 7146 Gas)

In case of a revert on msg.value == 0 we save 7146 gas on average from the protocol gas tests(to simulate the sad path ie msg.value == 0 see the test modification)	

																																																																																											
																																																																																																																	