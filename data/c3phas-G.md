# Codebase optimization report

## Auditor's Disclaimer 

While we try our best to maintain readability in the provided code snippets, some functions have been truncated to highlight the affected portions.

It's important to note that during the implementation of these suggested changes, developers must exercise caution to avoid introducing vulnerabilities. Although the optimizations have been tested prior to these recommendations, it is the responsibility of the developers to conduct thorough testing again.

Code reviews and additional testing are strongly advised to minimize any potential risks associated with the refactoring process.

## Table of Contents

- [Codebase optimization report](#codebase-optimization-report)
  - [Auditor's Disclaimer](#auditors-disclaimer)
  - [Table of Contents](#table-of-contents)
  - [Codebase impressions](#codebase-impressions)
  - [Tighly pack storage variables/optimize the order of variable declaration](#tighly-pack-storage-variablesoptimize-the-order-of-variable-declaration)
    - [Pack `_unlocked` with `bridgeAgentExecutorAddress` by reducing the size of `_unlocked` to `uint8`(Save 1 SLOT: 2K gas)](#pack-_unlocked-with-bridgeagentexecutoraddress-by-reducing-the-size-of-_unlocked-to-uint8save-1-slot-2k-gas)
    - [Pack `_unlocked` with `settlementNonce`(Save 1 SLOT: 2K gas)](#pack-_unlocked-with-settlementnoncesave-1-slot-2k-gas)
    - [Pack `_unlocked` with `localBridgeAgentAddress`(Save 1 SLOT: 2K gas)](#pack-_unlocked-with-localbridgeagentaddresssave-1-slot-2k-gas)
    - [Pack `_unlocked` with `depositNonce`(Save 1 SLOT: 2K gas)](#pack-_unlocked-with-depositnoncesave-1-slot-2k-gas)
  - [Not initializing variables here introduces the moving from 0 -\> 1 scenario(only affects deployment)](#not-initializing-variables-here-introduces-the-moving-from-0---1-scenarioonly-affects-deployment)
    - [Restructure the assignment model here (save 15K gas)](#restructure-the-assignment-model-here-save-15k-gas)
    - [RootBridgeAgent.sol: Assign the value during declaration(save 15K gas)](#rootbridgeagentsol-assign-the-value-during-declarationsave-15k-gas)
  - [We can optimize  the function `redeemDeposit`  by refactoring the code to fail Early - 2.1k half of the time(scenario where sender is address(0))](#we-can-optimize--the-function-redeemdeposit--by-refactoring-the-code-to-fail-early---21k-half-of-the-timescenario-where-sender-is-address0)
  - [When using storage instead of memory, we should cache any fields that need to be re-read in stack variables](#when-using-storage-instead-of-memory-we-should-cache-any-fields-that-need-to-be-re-read-in-stack-variables)
    - [RootBridgeAgent.sol: Cache `settlementReference.owner` in memory](#rootbridgeagentsol-cache-settlementreferenceowner-in-memory)
    - [RootBridgeAgent.sol: Cache `settlementReference.owner` in memory](#rootbridgeagentsol-cache-settlementreferenceowner-in-memory-1)
  - [A modifier used only once and not being inherited should be inlined to save gas](#a-modifier-used-only-once-and-not-being-inherited-should-be-inlined-to-save-gas)
  - [Caching immutables to memory only increases gas](#caching-immutables-to-memory-only-increases-gas)
  - [Conclusion](#conclusion)


## Codebase impressions

The developers have done an excellent job of identifying and implementing some of the most evident optimizations. 
However, we have identified additional areas where optimization is possible, some of which may not be immediately apparent.

## Tighly pack storage variables/optimize the order of variable declaration

Here, the storage variables can be tightly packed.

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/MulticallRootRouter.sol#L76-L80

### Pack `_unlocked` with `bridgeAgentExecutorAddress` by reducing the size of `_unlocked` to `uint8`(Save 1 SLOT: 2K gas)

```solidity
File: /src/MulticallRootRouter.sol
76:    /// @notice Bridge Agent Executor Address.
77:    address public bridgeAgentExecutorAddress;

79:    /// @notice Re-entrancy lock modifier state.
80:    uint256 internal _unlocked = 1;
```

`_unlocked` only has two possibilities , 1 or 2, as such `uint8` should be more than enough to hold the value. 
Reducing this size allows us to pack `_unlocked` with `bridgeAgentExecutorAddress` as they are being accessed together in some capacity which is shown below

The variable `_unlocked` is being used by the modifier `lock()` while the address variable `bridgeAgentExecutorAddress` has been used on the internal function `_requiresExecutor()` which is in turn used by the modifier `requiresExecutor()`. Now both of this modifiers `lock()` and `requiresExecutor()`  are used on the same functions eg https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/MulticallRootRouter.sol#L137 
```solidity
137:    function execute(bytes calldata encodedData, uint16) external payable override lock requiresExecutor {
```
which means our two variables are being accessed in the same transaction and as such packing them would have a significant gas savings

```diff
diff --git a/src/MulticallRootRouter.sol b/src/MulticallRootRouter.sol
index 0621f81..5c19ece 100644
--- a/src/MulticallRootRouter.sol
+++ b/src/MulticallRootRouter.sol
@@ -77,7 +77,7 @@ contract MulticallRootRouter is IRootRouter, Ownable {
     address public bridgeAgentExecutorAddress;

     /// @notice Re-entrancy lock modifier state.
-    uint256 internal _unlocked = 1;
+    uint8 internal _unlocked = 1;

```


https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L74-L92

### Pack `_unlocked` with `settlementNonce`(Save 1 SLOT: 2K gas)

```solidity
File:/src/RootBridgeAgent.sol#L74-L92
74:    /// @notice Deposit nonce used for identifying transaction.
75:    uint32 public settlementNonce;

78:    mapping(uint256 nonce => Settlement settlementInfo) public getSettlement;

85:    mapping(uint256 chainId => mapping(uint256 nonce => uint256 state)) public executionState;

92:    uint256 internal _unlocked = 1;
```

Similar case with the previous instance, we can reduce the size of `_unlocked` to `uint8` and pack it with `settlementNonce` since they are accessed on one transaction.
`_unlocked` is being used by the modifier `lock()` which is then applied to functions that access the variable `settlementNonce` . see the function below
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L175-L195
```solidity
File: /src/RootBridgeAgent.sol
175:    function callOutAndBridge(

183:    ) external payable override lock requiresRouter {
184:        // Create Settlement and Perform call
185:        bytes memory payload = _createSettlement(
186:            settlementNonce,

195:        );
```

We can see the above function uses the modifier `lock` which has the following implementation. https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L1190-L1195
```solidity
1190:    modifier lock() {
1191:        require(_unlocked == 1);
1192:        _unlocked = 2;
1193:        _;
1194:        _unlocked = 1;
1195:    }
```
We can save 1 SLOT by refactoring the code as follows

```diff
diff --git a/src/RootBridgeAgent.sol b/src/RootBridgeAgent.sol
index a6ad0ef..8b5843d 100644
--- a/src/RootBridgeAgent.sol
+++ b/src/RootBridgeAgent.sol
@@ -67,29 +67,31 @@ contract RootBridgeAgent is IRootBridgeAgent, BridgeAgentConstants {
     /// @notice If true, bridge agent manager has allowed for a new given branch bridge agent to be synced/added.
     mapping(uint256 chainId => bool allowed) public isBranchBridgeAgentAllowed;

+
     /*///////////////////////////////////////////////////////////////
-                            SETTLEMENTS STATE
+                            EXECUTOR STATE
     //////////////////////////////////////////////////////////////*/

-    /// @notice Deposit nonce used for identifying transaction.
-    uint32 public settlementNonce;
+    /// @notice If true, bridge agent has already served a request with this nonce from  a given chain. Chain -> Nonce -> Bool
+    mapping(uint256 chainId => mapping(uint256 nonce => uint256 state)) public executionState;

-    /// @notice Mapping from Settlement nonce to Settlement Struct.
-    mapping(uint256 nonce => Settlement settlementInfo) public getSettlement;

     /*///////////////////////////////////////////////////////////////
-                            EXECUTOR STATE
+                            SETTLEMENTS STATE
     //////////////////////////////////////////////////////////////*/

-    /// @notice If true, bridge agent has already served a request with this nonce from  a given chain. Chain -> Nonce -> Bool
-    mapping(uint256 chainId => mapping(uint256 nonce => uint256 state)) public executionState;
+    /// @notice Mapping from Settlement nonce to Settlement Struct.
+    mapping(uint256 nonce => Settlement settlementInfo) public getSettlement;
+
+        /// @notice Deposit nonce used for identifying transaction.
+    uint32 public settlementNonce;

     /*///////////////////////////////////////////////////////////////
                             REENTRANCY STATE
     //////////////////////////////////////////////////////////////*/

     /// @notice Re-entrancy lock modifier state.
-    uint256 internal _unlocked = 1;
+    uint8 internal _unlocked = 1;

```

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BaseBranchRouter.sol#L32-L42

### Pack `_unlocked` with `localBridgeAgentAddress`(Save 1 SLOT: 2K gas)

```solidity
File: /src/BaseBranchRouter.sol
32:    /// @inheritdoc IBranchRouter
33:    address public localPortAddress;

35:    /// @inheritdoc IBranchRouter
36:    address public override localBridgeAgentAddress;

38:    /// @inheritdoc IBranchRouter
39:    address public override bridgeAgentExecutorAddress;

41:    /// @notice Re-entrancy lock modifier state.
42:    uint256 internal _unlocked = 1;
```

Reduce the size of `_unlocked` to `uint8` and pack with `localBridgeAgentAddress`. We are packing with `localBridgeAgentAddress` because the variable `_unlocked` and `localBridgeAgentAddress` are being accessed in the same transaction

```diff
diff --git a/src/BaseBranchRouter.sol b/src/BaseBranchRouter.sol
index 62bc287..275b77a 100644
--- a/src/BaseBranchRouter.sol
+++ b/src/BaseBranchRouter.sol
@@ -35,11 +35,13 @@ contract BaseBranchRouter is IBranchRouter, Ownable {
     /// @inheritdoc IBranchRouter
     address public override localBridgeAgentAddress;

+        /// @notice Re-entrancy lock modifier state.
+    uint8 internal _unlocked = 1;
+
     /// @inheritdoc IBranchRouter
     address public override bridgeAgentExecutorAddress;

-    /// @notice Re-entrancy lock modifier state.
-    uint256 internal _unlocked = 1;
```



https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L84-L101

### Pack `_unlocked` with `depositNonce`(Save 1 SLOT: 2K gas)

```solidity
File: /src/BranchBridgeAgent.sol
84:    uint32 public depositNonce;

87:    mapping(uint256 depositNonce => Deposit depositInfo) public getDeposit;

94:    mapping(uint256 settlementNonce => uint256 state) public executionState;

101:    uint256 internal _unlocked = 1;
```
Reduce the size of `_unlocked` to `uint8` and pack with `depositNonce`
```diff
diff --git a/src/BranchBridgeAgent.sol b/src/BranchBridgeAgent.sol
index b076d2d..0df2312 100644
--- a/src/BranchBridgeAgent.sol
+++ b/src/BranchBridgeAgent.sol
@@ -80,25 +80,27 @@ contract BranchBridgeAgent is IBranchBridgeAgent, BridgeAgentConstants {
                             DEPOSITS STATE
     //////////////////////////////////////////////////////////////*/

-    /// @notice Deposit nonce used for identifying the transaction.
-    uint32 public depositNonce;

     /// @notice Mapping from Pending deposits hash to Deposit Struct.
     mapping(uint256 depositNonce => Deposit depositInfo) public getDeposit;
+
+    /// @notice Deposit nonce used for identifying the transaction.
+    uint32 public depositNonce;

     /*///////////////////////////////////////////////////////////////
-                        SETTLEMENT EXECUTION STATE
+                           REENTRANCY STATE
     //////////////////////////////////////////////////////////////*/

-    /// @notice If true, the bridge agent has already served a request with thi
s nonce from a given chain.
-    mapping(uint256 settlementNonce => uint256 state) public executionState;
+    /// @notice Re-entrancy lock modifier state.
+    uint8 internal _unlocked = 1;

     /*///////////////////////////////////////////////////////////////
-                           REENTRANCY STATE
+                        SETTLEMENT EXECUTION STATE
     //////////////////////////////////////////////////////////////*/

-    /// @notice Re-entrancy lock modifier state.
-    uint256 internal _unlocked = 1;
+    /// @notice If true, the bridge agent has already served a request with this nonce from a given chain.
+    mapping(uint256 settlementNonce => uint256 state) public executionState;
+

```

## Not initializing variables here introduces the moving from 0 -> 1 scenario(only affects deployment) 

### Restructure the assignment model here (save 15K gas)

Starting at `1` (keeping the slot non-zero), will cost 5k per change (5k + 5k) vs 20k + 5k, saving you 15k gas 
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L84C31-L84C32
```solidity
File: /src/BranchBridgeAgent.sol
84:    uint32 public depositNonce;
```
Since we are not initializing this value, the default is applied which is `0`. In the constructor , we initialize this value to `1` which means we move from `0 -> 1`
We could save some gas by initializing this value when it's being declared. Also, since in the constructor we always assign a static value ie `1` we don't really need to have it there anymore if we initialize during declaration
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L117-L143

```solidity
117:    constructor(

124:    ) {

140:        depositNonce = 1;

143:    }
```

Changing a value from 0 to 1 costs more than 20000 gas while changing from 1 to 2 would only cost 5000 gas
```diff
diff --git a/src/BranchBridgeAgent.sol b/src/BranchBridgeAgent.sol
index b076d2d..6c9f60d 100644
--- a/src/BranchBridgeAgent.sol
+++ b/src/BranchBridgeAgent.sol
@@ -81,7 +81,7 @@ contract BranchBridgeAgent is IBranchBridgeAgent, BridgeAgentConstants {
     //////////////////////////////////////////////////////////////*/

     /// @notice Deposit nonce used for identifying the transaction.
-    uint32 public depositNonce;
+    uint32 public depositNonce = 1;

     /// @notice Mapping from Pending deposits hash to Deposit Struct.
     mapping(uint256 depositNonce => Deposit depositInfo) public getDeposit;
@@ -137,7 +137,6 @@ contract BranchBridgeAgent is IBranchBridgeAgent, BridgeAgentConstants {
         localRouterAddress = _localRouterAddress;
         localPortAddress = _localPortAddress;
         bridgeAgentExecutorAddress = DeployBranchBridgeAgentExecutor.deploy();
-        depositNonce = 1;

         rootBridgeAgentPath = abi.encodePacked(_rootBridgeAgentAddress, address(this));
     }
```


https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L74-L75

### RootBridgeAgent.sol: Assign the value during declaration(save 15K gas)

```solidity
File: /src/RootBridgeAgent.sol
74:    /// @notice Deposit nonce used for identifying transaction.
75:    uint32 public settlementNonce;
```
The above variable is being assigned in the constructor as follows

```solidity
121:        settlementNonce = 1;
```

As with our previous case, since the variable was not assigned anything at declaration, it assumed the value `0` . This means when assigning the value in the constructor , we are  basically moving from `0 -> 1`  a typical case we see with `bools` for true/false. Since the value to be assigned is hardcoded, there's no need to do this in the constructor, we can assign the value during declaration. This would eliminate the `0->1` which consumes too much gas.

```diff
diff --git a/src/RootBridgeAgent.sol b/src/RootBridgeAgent.sol
index a6ad0ef..dcadf7c 100644
--- a/src/RootBridgeAgent.sol
+++ b/src/RootBridgeAgent.sol
@@ -72,7 +72,7 @@ contract RootBridgeAgent is IRootBridgeAgent, BridgeAgentConstants {
     //////////////////////////////////////////////////////////////*/

     /// @notice Deposit nonce used for identifying transaction.
-    uint32 public settlementNonce;
+    uint32 public settlementNonce = 1;

     /// @notice Mapping from Settlement nonce to Settlement Struct.
     mapping(uint256 nonce => Settlement settlementInfo) public getSettlement;
@@ -118,7 +118,6 @@ contract RootBridgeAgent is IRootBridgeAgent, BridgeAgentConstants {
         localPortAddress = _localPortAddress;
         localRouterAddress = _localRouterAddress;
         bridgeAgentExecutorAddress = DeployRootBridgeAgentExecutor.deploy(address(this));
-        settlementNonce = 1;
     }
```

## We can optimize  the function `redeemDeposit`  by refactoring the code to fail Early - 2.1k half of the time(scenario where sender is address(0))

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L434-L457
```solidity
File: /src/BranchBridgeAgent.sol
434:    function redeemDeposit(uint32 _depositNonce) external override lock {
435:        // Get storage reference
436:        Deposit storage deposit = getDeposit[_depositNonce];

438:        // Check Deposit
439:        if (deposit.status == STATUS_SUCCESS) revert DepositRedeemUnavailable();
440:        if (deposit.owner == address(0)) revert DepositRedeemUnavailable();
441:        if (deposit.owner != msg.sender) revert NotDepositOwner();
```
Of interest to us is the last two if statements, we first verify that `deposit.owner` is not equal to `address(0)` and revert if it happens to be `address(0)`
We then validate that the `msg.sender` is in fact the `deposit.owner`, so for the last check to pass, we would need the `deposit.owner == msg.sender` to hold

If we reorder the two checks and validate `deposit.owner != msg.sender` first, it would mean any occurrence of the variable `deposit.owner` can be replaced by  `msg.sender` since `msg.sender` would be equal to `deposit.owner`

```diff
         // Check Deposit
         if (deposit.status == STATUS_SUCCESS) revert DepositRedeemUnavailable();
-        if (deposit.owner == address(0)) revert DepositRedeemUnavailable();
         if (deposit.owner != msg.sender) revert NotDepositOwner();
+        if (msg.sender == address(0)) revert DepositRedeemUnavailable();
```
After the first refactor, we get a clear view of how the checks are being done and we can now refactor even more by moving the check `msg.sender == address()` to the top before reading any state variables
We can therefore save an SLOAD (Gscold = 2.1K) gas and another two sloads for reading the variable `deposit.status` and deposit.owner

```diff
     function redeemDeposit(uint32 _depositNonce) external override lock {
+        if (msg.sender == address(0)) revert DepositRedeemUnavailable();
         // Get storage reference
         Deposit storage deposit = getDeposit[_depositNonce];

         // Check Deposit
         if (deposit.status == STATUS_SUCCESS) revert DepositRedeemUnavailable();
-        if (deposit.owner == address(0)) revert DepositRedeemUnavailable();
         if (deposit.owner != msg.sender) revert NotDepositOwner();

```


## When using storage instead of memory, we should cache any fields that need to be re-read in stack variables

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L233-L271

### RootBridgeAgent.sol: Cache `settlementReference.owner` in memory 

```solidity
File: /src/RootBridgeAgent.sol
233:    function retrySettlement(

239:    ) external payable override lock {
240:        // Get storage reference
241:        Settlement storage settlementReference = getSettlement[_settlementNonce];

244:        if (settlementReference.owner == address(0)) revert SettlementRetryUnavailable();

247:        if (msg.sender != settlementReference.owner) {
248:            if (msg.sender != address(IPort(localPortAddress).getUserAccount(settlementReference.owner))) {
249:                revert NotSettlementOwner();
250:            }
251:        }

257:        _performRetrySettlementCall(

265:            payable(settlementReference.owner),

270:        );
271:    }
```
The following suggestion is already being applied in other parts of this code, see https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L279

```diff
diff --git a/src/RootBridgeAgent.sol b/src/RootBridgeAgent.sol
index a6ad0ef..a1fd294 100644
--- a/src/RootBridgeAgent.sol
+++ b/src/RootBridgeAgent.sol
@@ -240,12 +240,15 @@ contract RootBridgeAgent is IRootBridgeAgent, BridgeAgentConstants {
         // Get storage reference
         Settlement storage settlementReference = getSettlement[_settlementNonce];

+                // Get Settlement owner.
+        address settlementOwner = settlementReference.owner;
+
         // Check if Settlement hasn't been redeemed.
-        if (settlementReference.owner == address(0)) revert SettlementRetryUnavailable();
+        if (settlementOwner == address(0)) revert SettlementRetryUnavailable();

         // Check if caller is Settlement owner
-        if (msg.sender != settlementReference.owner) {
-            if (msg.sender != address(IPort(localPortAddress).getUserAccount(settlementReference.owner))) {
+        if (msg.sender != settlementOwner) {
+            if (msg.sender != address(IPort(localPortAddress).getUserAccount(settlementOwner))) {
                 revert NotSettlementOwner();
             }
         }
@@ -262,7 +265,7 @@ contract RootBridgeAgent is IRootBridgeAgent, BridgeAgentConstants {
             settlementReference.deposits,
             _params,
             _settlementNonce,
-            payable(settlementReference.owner),
+            payable(settlementOwner),
             _recipient,
             settlementReference.dstChainId,
             _gParams,
```


https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L667-L691

### RootBridgeAgent.sol: Cache `settlementReference.owner` in memory 

```solidity
File: /src/RootBridgeAgent.sol
667:            Settlement storage settlementReference = getSettlement[nonce];

670:            if (settlementReference.owner == address(0)) revert SettlementRetryUnavailable();

673:            if (owner != settlementReference.owner) {
674:                if (owner != address(IPort(localPortAddress).getUserAccount(settlementReference.owner))) {
675:                    revert NotSettlementOwner();
676:                }
677:            }

683:            _performRetrySettlementCall(
            
691:                payable(settlementReference.owner),
```

```diff
             Settlement storage settlementReference = getSettlement[nonce];

+            address settlementOwner = settlementReference.owner;
+
             // Check if Settlement hasn't been redeemed.
-            if (settlementReference.owner == address(0)) revert SettlementRetryUnavailable();
+            if (settlementOwner == address(0)) revert SettlementRetryUnavailable();

             // Check settlement owner
-            if (owner != settlementReference.owner) {
-                if (owner != address(IPort(localPortAddress).getUserAccount(settlementReference.owner))) {
+            if (owner != settlementOwner) {
+                if (owner != address(IPort(localPortAddress).getUserAccount(settlementOwner))) {
                     revert NotSettlementOwner();
                 }
             }
@@ -688,7 +690,7 @@ contract RootBridgeAgent is IRootBridgeAgent, BridgeAgentConstants {
                 settlementReference.deposits,
                 params,
                 nonce,
-                payable(settlementReference.owner),
+                payable(settlementOwner),
```

## A modifier used only once and not being inherited should be inlined to save gas

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/factories/ERC20hTokenRootFactory.sol#L96-L103
```solidity
File: /src/factories/ERC20hTokenRootFactory.sol
96:    modifier requiresCoreRouterOrPort() {
97:        if (msg.sender != coreRootRouterAddress) {
98:            if (msg.sender != rootPortAddress) {
99:                revert UnrecognizedCoreRouterOrPort();
100:            }
101:        }
102:        _;
103:    }
```
The above modifier is only used on https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/factories/ERC20hTokenRootFactory.sol#L78

**Similar instances**
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L553-L556
```solidity
File: /src/BranchPort.sol
553:    modifier requiresBridgeAgentFactory() {
554:        if (!isBridgeAgentFactory[msg.sender]) revert UnrecognizedBridgeAgentFactory();
555:        _;
556:    }
```
The above modifier is only used on https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L319


https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L559-L563
```solidity
File: /src/BranchPort.sol
559:    modifier requiresPortStrategy(address _token) {
560:        if (!isStrategyToken[_token]) revert UnrecognizedStrategyToken();
561:        if (!isPortStrategy[msg.sender][_token]) revert UnrecognizedPortStrategy();
562:        _;
563:    }
```
The modifier above is only called on https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L144

## Caching immutables to memory only increases gas

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/ArbitrumBranchPort.sol#L57-L69
```solidity
File: /src/ArbitrumBranchPort.sol
57:        address _rootPortAddress = rootPortAddress;

60:        address _globalToken = IRootPort(_rootPortAddress).getLocalTokenFromUnderlying(_underlyingAddress, localChainId);

63:        if (_globalToken == address(0)) revert UnknownGlobalToken();

66:        _underlyingAddress.safeTransferFrom(_depositor, address(this), _deposit);

69:        IRootPort(_rootPortAddress).mintToLocalBranch(_recipient, _globalToken, _deposit);
```

```diff
diff --git a/src/ArbitrumBranchPort.sol b/src/ArbitrumBranchPort.sol
index 5777422..cc9c872 100644
--- a/src/ArbitrumBranchPort.sol
+++ b/src/ArbitrumBranchPort.sol
@@ -53,11 +53,8 @@ contract ArbitrumBranchPort is BranchPort, IArbitrumBranchPort {
         lock
         requiresBridgeAgent
     {
-        // Save root port address to memory
-        address _rootPortAddress = rootPortAddress;
-
         // Get global token address from root port
-        address _globalToken = IRootPort(_rootPortAddress).getLocalTokenFromUnderlying(_underlyingAddress, localChainId);
+        address _globalToken = IRootPort(rootPortAddress).getLocalTokenFromUnderlying(_underlyingAddress, localChainId);

         // Check if the global token exists
         if (_globalToken == address(0)) revert UnknownGlobalToken();
@@ -66,7 +63,7 @@ contract ArbitrumBranchPort is BranchPort, IArbitrumBranchPort {
         _underlyingAddress.safeTransferFrom(_depositor, address(this), _deposit);

         // Request Minting of Global Token
-        IRootPort(_rootPortAddress).mintToLocalBranch(_recipient, _globalToken, _deposit);
+        IRootPort(rootPortAddress).mintToLocalBranch(_recipient, _globalToken, _deposit);
     }
```


https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/ArbitrumBranchPort.sol#L80-L92
```solidity
File: /src/ArbitrumBranchPort.sol
80:        address _rootPortAddress = rootPortAddress;

83:        if (!IRootPort(_rootPortAddress).isGlobalToken(_globalAddress, localChainId)) revert UnknownGlobalToken();

86:        address _underlyingAddress =
87:      IRootPort(_rootPortAddress).getUnderlyingTokenFromLocal(_globalAddress, localChainId);

90:        if (_underlyingAddress == address(0)) revert UnknownUnderlyingToken();

92:        IRootPort(_rootPortAddress).burnFromLocalBranch(_depositor, _globalAddress, _amount);
```

```diff
diff --git a/src/ArbitrumBranchPort.sol b/src/ArbitrumBranchPort.sol
index 5777422..3124d7d 100644
--- a/src/ArbitrumBranchPort.sol
+++ b/src/ArbitrumBranchPort.sol
@@ -76,20 +76,18 @@ contract ArbitrumBranchPort is BranchPort, IArbitrumBranchPort {
         lock
         requiresBridgeAgent
     {
-        // Save root port address to memory
-        address _rootPortAddress = rootPortAddress;

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
     }
```

## Conclusion

It is important to emphasize that the provided recommendations aim to enhance the efficiency of the code without compromising its readability. We understand the value of maintainable and easily understandable code to both developers and auditors.

As you proceed with implementing the suggested optimizations, please exercise caution and be diligent in conducting thorough testing. It is crucial to ensure that the changes are not introducing any new vulnerabilities and that the desired performance improvements are achieved. Review code changes, and perform thorough testing to validate the effectiveness and security of the refactored code.

Should you have any questions or need further assistance, please don't hesitate to reach out.
