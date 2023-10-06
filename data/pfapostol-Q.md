## Summary table

|     Index     | Issue                                                                                   | Instances |
| :-----------: | :-------------------------------------------------------------------------------------- | :-------: |
| [N-01](#N-01) | [Inaccurate comments for bridge agents, factories and chain id](#N-01)                  |     7     |
| [N-02](#N-02) | [Mismatched array lengths can cause issues with user operations](#N-02)                 |     4     |
| [N-03](#N-03) | [Modifier duplication in multiple contracts](#N-03)                                     |     5     |
| [N-04](#N-04) | [Duplicated statements in multiple functions.](#N-04)                                   |     2     |
| [N-05](#N-05) | [Unnecessary modifiers for empty or revert functions.](#N-05) |     5     |
| [N-06](#N-06) | [Duplicate function bodies.](#N-06)                                       |     1     |

24 instances over 6 issues

## 01. **Inaccurate comments for bridge agents, factories and chain id (7 instances)**

<a name="N-01"></a>

### Description: 
The comments in the code contain incorrect information regarding the addresses and purpose of the bridge agents, factories and chain id. 
### Solution: 
Update the comments to accurately describe the purpose and addresses of bridge agents and factories.

- src\BranchPort.sol:[32](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L31), [34](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L34), [41](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L41), [44](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L44)

The comment for `isBridgeAgent` should specify information for the bridge agent address instead of the underlying address.

The comment for `bridgeAgents` should provide information for the bridge agent address instead of the branch router.

The comment for `isBridgeAgentFactory` should include information for the bridge agent factory address instead of the underlying address.

The comment for `bridgeAgentFactories` should provide information for the bridge agent factory address instead of the branch router.


```solidity
31    /// @notice Mapping from Underlying Address to isUnderlying (bool).
32    mapping(address bridgeAgent => bool isActiveBridgeAgent) public isBridgeAgent;
...
34    /// @notice Branch Routers deployed in branch chain.
35    address[] public bridgeAgents;
...
41    /// @notice Mapping from Underlying Address to isUnderlying (bool).
42    mapping(address bridgeAgentFactory => bool isActiveBridgeAgentFactory) public isBridgeAgentFactory;
...
44     /// @notice Branch Routers deployed in branch chain.
45    address[] public bridgeAgentFactories;
```

- src\RootPort.sol:[42](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L42), [60](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L60), [76](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L76)

For `coreRootBridgeAgentAddress` comment specifies info for core router instead of Bridge Agent
For `isChainId` comment specifies info for Bridge Agent mapping instead of chain ids
For `isBridgeAgentFactory` comment specifies info for Underlying address instead of bridge agent factory

The comment for `coreRootBridgeAgentAddress` should provide information for the bridge agent address instead of the core router.

The comment for `isChainId` should specify information for the chain IDs mapping instead of the bridge agent mapping.

The comment for `isBridgeAgentFactory` should include information for the bridge agent factory address instead of the underlying address.

```solidity
42     /// @notice The address of the core router in charge of adding new tokens to the system.
43    address public coreRootBridgeAgentAddress;
...
60    /// @notice Mapping from address to Bridge Agent.
61    mapping(uint256 chainId => bool isActive) public isChainId;
...
76    /// @notice Mapping from Underlying Address to isUnderlying (bool).
77    mapping(address bridgeAgentFactory => bool isActive) public isBridgeAgentFactory;
```

---

## 02. **Mismatched array lengths can cause issues with user operations (4 instances)** 

<a name="N-02"></a>

### Description: 
If the requirement for the arrays to have the same length is not enforced, user operations may not be successfully executed. This is because there might be a mismatch between the number of items iterated over and the number of items provided in the second array. 

### Solution: 
Enforce that the arrays have the same length before iterating over them. This can help avoid mismatches and ensure that operations are executed correctly.

- src/BaseBranchRouter.sol:[186](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BaseBranchRouter.sol#L186)
- src/BranchPort.sol:[246](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L246)
- src/MulticallRootRouter.sol:[544](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/MulticallRootRouter.sol#L544)
- src/RootBridgeAgent.sol:[856](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L856)

```diff
diff --git a/src/BaseBranchRouter.sol b/src/BaseBranchRouter.sol
index 62bc287..87c4ac4 100644
--- a/src/BaseBranchRouter.sol
+++ b/src/BaseBranchRouter.sol
@@ -189,6 +189,11 @@ contract BaseBranchRouter is IBranchRouter, Ownable {
         uint256[] memory _amounts,
         uint256[] memory _deposits
     ) internal {
+        if (
+            _hTokens.length != _tokens.length || _tokens.length != _amounts.length || _amounts.length != _deposits.length
+        ) revert("Length missmatch");
+
+
         for (uint256 i = 0; i < _hTokens.length;) {
             _transferAndApproveToken(_hTokens[i], _tokens[i], _amounts[i], _deposits[i]);
 
diff --git a/src/BranchPort.sol b/src/BranchPort.sol
index ba60acc..619c958 100644
--- a/src/BranchPort.sol
+++ b/src/BranchPort.sol
@@ -252,6 +252,9 @@ contract BranchPort is Ownable, IBranchPort {
     ) external override requiresBridgeAgent {
         // Cache Length
         uint256 length = _localAddresses.length;
+        if (
+            length != _underlyingAddresses.length || length != _amounts.length || length != _deposits.length
+        ) revert("Length missmatch");
 
         // Loop through token inputs
         for (uint256 i = 0; i < length;) {
diff --git a/src/MulticallRootRouter.sol b/src/MulticallRootRouter.sol
index 0621f81..bd241b7 100644
--- a/src/MulticallRootRouter.sol
+++ b/src/MulticallRootRouter.sol
@@ -550,6 +550,10 @@ contract MulticallRootRouter is IRootRouter, Ownable {
         uint16 dstChainId,
         GasParams memory gasParams
     ) internal virtual {
+        if (
+            outputTokens.length != amountsOut.length || amountsOut.length != depositsOut.length
+        ) revert("Length missmatch");
+
         // Save bridge agent address to memory
         address _bridgeAgentAddress = bridgeAgentAddress;
 
diff --git a/src/RootBridgeAgent.sol b/src/RootBridgeAgent.sol
index a6ad0ef..6fa9d65 100644
--- a/src/RootBridgeAgent.sol
+++ b/src/RootBridgeAgent.sol
@@ -870,6 +870,10 @@ contract RootBridgeAgent is IRootBridgeAgent, BridgeAgentConstants {
         // Check if payload is ready for message
         if (_hTokens.length == 0) revert SettlementRetryUnavailableUseCallout();
 
+        if (
+            _hTokens.length != _tokens.length || _tokens.length != _amounts.length || _amounts.length != _deposits.length
+        ) revert("Length missmatch");
+
         // Get packed data
         bytes memory payload;
```

---

## 03. **Modifier duplication in multiple contracts (5 instances)**

<a name="N-03"></a>

### Description: 
Several contracts have the same modifier defined within them. This duplication can be avoided by moving the modifier to a separate library and importing it where needed.
### Solution: 
Extract the common modifier to a separate library file and import it in the contracts that require it.

- src/RootBridgeAgent.sol:[1191](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L1191)

- src/MulticallRootRouter.sol:[591](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/MulticallRootRouter.sol#L591)

- src/BranchPort.sol:[567](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L567)

- src/BranchBridgeAgent.sol:[923](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L923)

- src/BaseBranchRouter.sol:[213](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BaseBranchRouter.sol#L213)

Example:

```solidity
// SPDX-License-Identifier: SEE LICENSE IN LICENSE
pragma solidity ^0.8.0;

contract Lock {
    /// @notice Re-entrancy lock modifier state.
    uint256 internal _unlocked = 1;

    /// @notice Modifier for a simple re-entrancy check.
    modifier lock() {
        require(_unlocked == 1);
        _unlocked = 2;
        _;
        _unlocked = 1;
    }
}
```

```diff
diff --git a/src/BaseBranchRouter.sol b/src/BaseBranchRouter.sol
index 62bc287..5e294ce 100644
--- a/src/BaseBranchRouter.sol
+++ b/src/BaseBranchRouter.sol
@@ -7,6 +7,7 @@ import {SafeTransferLib} from "solady/utils/SafeTransferLib.sol";
 import {ERC20} from "solmate/tokens/ERC20.sol";
 
 import {IBranchRouter} from "./interfaces/IBranchRouter.sol";
+import "src/lock.sol";
 
 import {
     IBranchBridgeAgent as IBridgeAgent,
@@ -22,7 +23,7 @@ import {
 
 /// @title Base Branch Router Contract
 /// @author MaiaDAO
-contract BaseBranchRouter is IBranchRouter, Ownable {
+contract BaseBranchRouter is IBranchRouter, Ownable, Lock {
     using SafeTransferLib for address;
 
     /*///////////////////////////////////////////////////////////////
@@ -38,9 +39,6 @@ contract BaseBranchRouter is IBranchRouter, Ownable {
     /// @inheritdoc IBranchRouter
     address public override bridgeAgentExecutorAddress;
 
-    /// @notice Re-entrancy lock modifier state.
-    uint256 internal _unlocked = 1;
-
     /*///////////////////////////////////////////////////////////////
                             CONSTRUCTOR
     //////////////////////////////////////////////////////////////*/
@@ -207,12 +205,4 @@ contract BaseBranchRouter is IBranchRouter, Ownable {
         if (msg.sender != bridgeAgentExecutorAddress) revert UnrecognizedBridgeAgentExecutor();
         _;
     }
-
-    /// @notice Modifier for a simple re-entrancy check.
-    modifier lock() {
-        require(_unlocked == 1);
-        _unlocked = 2;
-        _;
-        _unlocked = 1;
-    }
 }
diff --git a/src/BranchBridgeAgent.sol b/src/BranchBridgeAgent.sol
index b076d2d..3a5a831 100644
--- a/src/BranchBridgeAgent.sol
+++ b/src/BranchBridgeAgent.sol
@@ -18,7 +18,7 @@ import {
 import {ILayerZeroEndpoint} from "./interfaces/ILayerZeroEndpoint.sol";
 
 import {BranchBridgeAgentExecutor, DeployBranchBridgeAgentExecutor} from "./BranchBridgeAgentExecutor.sol";
-
+import "src/lock.sol";
 /// @title Library for Branch Bridge Agent Deployment
 library DeployBranchBridgeAgent {
     function deploy(
@@ -42,7 +42,7 @@ library DeployBranchBridgeAgent {
 
 /// @title Branch Bridge Agent Contract
 /// @author MaiaDAO
-contract BranchBridgeAgent is IBranchBridgeAgent, BridgeAgentConstants {
+contract BranchBridgeAgent is IBranchBridgeAgent, BridgeAgentConstants, Lock {
     using ExcessivelySafeCall for address;
 
     /*///////////////////////////////////////////////////////////////
@@ -93,13 +93,6 @@ contract BranchBridgeAgent is IBranchBridgeAgent, BridgeAgentConstants {
     /// @notice If true, the bridge agent has already served a request with this nonce from a given chain.
     mapping(uint256 settlementNonce => uint256 state) public executionState;
 
-    /*///////////////////////////////////////////////////////////////
-                           REENTRANCY STATE
-    //////////////////////////////////////////////////////////////*/
-
-    /// @notice Re-entrancy lock modifier state.
-    uint256 internal _unlocked = 1;
-
     /*///////////////////////////////////////////////////////////////
                              CONSTRUCTOR
     //////////////////////////////////////////////////////////////*/
@@ -918,14 +911,6 @@ contract BranchBridgeAgent is IBranchBridgeAgent, BridgeAgentConstants {
                                 MODIFIERS
     //////////////////////////////////////////////////////////////*/
 
-    /// @notice Modifier for a simple re-entrancy check.
-    modifier lock() {
-        require(_unlocked == 1);
-        _unlocked = 2;
-        _;
-        _unlocked = 1;
-    }
-
     /// @notice Modifier verifies the caller is the Layerzero Enpoint or Local Branch Bridge Agent.
     modifier requiresEndpoint(address _endpoint, bytes calldata _srcAddress) {
         _requiresEndpoint(_endpoint, _srcAddress);
diff --git a/src/BranchPort.sol b/src/BranchPort.sol
index ba60acc..26d5a56 100644
--- a/src/BranchPort.sol
+++ b/src/BranchPort.sol
@@ -11,10 +11,11 @@ import {IPortStrategy} from "./interfaces/IPortStrategy.sol";
 import {IBranchPort} from "./interfaces/IBranchPort.sol";
 
 import {ERC20hTokenBranch} from "./token/ERC20hTokenBranch.sol";
+import "src/lock.sol";
 
 /// @title Branch Port - Omnichain Token Management Contract
 /// @author MaiaDAO
-contract BranchPort is Ownable, IBranchPort {
+contract BranchPort is Ownable, IBranchPort, Lock {
     using SafeTransferLib for address;
 
     /*///////////////////////////////////////////////////////////////
@@ -83,13 +84,6 @@ contract BranchPort is Ownable, IBranchPort {
     mapping(address strategy => mapping(address token => uint256 dailyLimitRemaining)) public
         strategyDailyLimitRemaining;
 
-    /*///////////////////////////////////////////////////////////////
-                            REENTRANCY STATE
-    //////////////////////////////////////////////////////////////*/
-
-    /// @notice Reentrancy lock guard state.
-    uint256 internal _unlocked = 1;
-
     /*///////////////////////////////////////////////////////////////
                             CONSTANTS
     //////////////////////////////////////////////////////////////*/
@@ -561,12 +555,4 @@ contract BranchPort is Ownable, IBranchPort {
         if (!isPortStrategy[msg.sender][_token]) revert UnrecognizedPortStrategy();
         _;
     }
-
-    /// @notice Modifier for a simple re-entrancy check.
-    modifier lock() {
-        require(_unlocked == 1);
-        _unlocked = 2;
-        _;
-        _unlocked = 1;
-    }
 }
diff --git a/src/MulticallRootRouter.sol b/src/MulticallRootRouter.sol
index 0621f81..c92e54f 100644
--- a/src/MulticallRootRouter.sol
+++ b/src/MulticallRootRouter.sol
@@ -13,6 +13,7 @@ import {
 } from "./interfaces/IRootBridgeAgent.sol";
 import {IRootRouter, DepositParams, DepositMultipleParams} from "./interfaces/IRootRouter.sol";
 import {IVirtualAccount, Call} from "./interfaces/IVirtualAccount.sol";
+import "src/lock.sol";
 
 struct OutputParams {
     // Address to receive the output assets.
@@ -54,7 +55,7 @@ struct OutputMultipleParams {
  *         0x06         | multicallSignedMultipleOutput
  *
  */
-contract MulticallRootRouter is IRootRouter, Ownable {
+contract MulticallRootRouter is IRootRouter, Ownable, Lock {
     using SafeTransferLib for address;
 
     /*///////////////////////////////////////////////////////////////
@@ -76,8 +77,6 @@ contract MulticallRootRouter is IRootRouter, Ownable {
     /// @notice Bridge Agent Executor Address.
     address public bridgeAgentExecutorAddress;
 
-    /// @notice Re-entrancy lock modifier state.
-    uint256 internal _unlocked = 1;
 
     /*///////////////////////////////////////////////////////////////
                             CONSTRUCTOR
@@ -586,13 +585,6 @@ contract MulticallRootRouter is IRootRouter, Ownable {
                             MODIFIERS
     ////////////////////////////////////////////////////////////*/
 
-    /// @notice Modifier for a simple re-entrancy check.
-    modifier lock() {
-        require(_unlocked == 1);
-        _unlocked = 2;
-        _;
-        _unlocked = 1;
-    }
 
     /// @notice Modifier verifies the caller is the Bridge Agent Executor.
     modifier requiresExecutor() {
diff --git a/src/RootBridgeAgent.sol b/src/RootBridgeAgent.sol
index a6ad0ef..9d9a2c7 100644
--- a/src/RootBridgeAgent.sol
+++ b/src/RootBridgeAgent.sol
@@ -26,10 +26,11 @@ import {IRootPort as IPort} from "./interfaces/IRootPort.sol";
 
 import {VirtualAccount} from "./VirtualAccount.sol";
 import {DeployRootBridgeAgentExecutor, RootBridgeAgentExecutor} from "./RootBridgeAgentExecutor.sol";
+import "src/lock.sol";
 
 /// @title Root Bridge Agent Contract
 /// @author MaiaDAO
-contract RootBridgeAgent is IRootBridgeAgent, BridgeAgentConstants {
+contract RootBridgeAgent is IRootBridgeAgent, BridgeAgentConstants, Lock {
     using SafeTransferLib for address;
     using ExcessivelySafeCall for address;
     /*///////////////////////////////////////////////////////////////
@@ -84,13 +85,6 @@ contract RootBridgeAgent is IRootBridgeAgent, BridgeAgentConstants {
     /// @notice If true, bridge agent has already served a request with this nonce from  a given chain. Chain -> Nonce -> Bool
     mapping(uint256 chainId => mapping(uint256 nonce => uint256 state)) public executionState;
 
-    /*///////////////////////////////////////////////////////////////
-                            REENTRANCY STATE
-    //////////////////////////////////////////////////////////////*/
-
-    /// @notice Re-entrancy lock modifier state.
-    uint256 internal _unlocked = 1;
-
     /*///////////////////////////////////////////////////////////////
                             CONSTRUCTOR
     //////////////////////////////////////////////////////////////*/
@@ -1186,14 +1180,6 @@ contract RootBridgeAgent is IRootBridgeAgent, BridgeAgentConstants {
                             MODIFIERS
     //////////////////////////////////////////////////////////////*/
 
-    /// @notice Modifier for a simple re-entrancy check.
-    modifier lock() {
-        require(_unlocked == 1);
-        _unlocked = 2;
-        _;
-        _unlocked = 1;
-    }
-
     /// @notice Internal function to verify msg sender is Bridge Agent's Router.
     modifier requiresRouter() {
         if (msg.sender != localRouterAddress) revert UnrecognizedRouter();
```

---

# 04. **Duplicated statements in multiple functions. (2 instances)**

<a name="N-04"></a>

### Description: 
Certain statements are repeated in multiple functions within the same contract. This duplication can be reduced by creating a modifier that includes these common statements.
### Solution: 
Create a modifier that includes the common statements and apply it to the relevant functions.

- src/factories/ArbitrumBranchBridgeAgentFactory.sol:[84](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/factories/ArbitrumBranchBridgeAgentFactory.sol#L84)
- src/factories/BranchBridgeAgentFactory.sol:[120](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/factories/BranchBridgeAgentFactory.sol#L120)

```diff
diff --git a/src/factories/ArbitrumBranchBridgeAgentFactory.sol b/src/factories/ArbitrumBranchBridgeAgentFactory.sol
index 3f07a7d..6455d9a 100644
--- a/src/factories/ArbitrumBranchBridgeAgentFactory.sol
+++ b/src/factories/ArbitrumBranchBridgeAgentFactory.sol
@@ -80,15 +80,14 @@ contract ArbitrumBranchBridgeAgentFactory is BranchBridgeAgentFactory {
         address _newBranchRouterAddress,
         address _rootBridgeAgentAddress,
         address _rootBridgeAgentFactoryAddress
-    ) external virtual override returns (address newBridgeAgent) {
-        require(
-            msg.sender == localCoreBranchRouterAddress, "Only the Core Branch Router can create a new Bridge Agent."
-        );
-        require(
-            _rootBridgeAgentFactoryAddress == rootBridgeAgentFactoryAddress,
-            "Root Bridge Agent Factory Address does not match."
-        );
-
+    )
+        external
+        virtual
+        override
+        requireCoreBranchRouter
+        requireRootBridgeAgentFactory(_rootBridgeAgentFactoryAddress)
+        returns (address newBridgeAgent)
+    {
         newBridgeAgent = address(
             DeployArbitrumBranchBridgeAgent.deploy(
                 rootChainId, _rootBridgeAgentAddress, _newBranchRouterAddress, localPortAddress
diff --git a/src/factories/BranchBridgeAgentFactory.sol b/src/factories/BranchBridgeAgentFactory.sol
index c789a27..33b594e 100644
--- a/src/factories/BranchBridgeAgentFactory.sol
+++ b/src/factories/BranchBridgeAgentFactory.sol
@@ -116,15 +116,13 @@ contract BranchBridgeAgentFactory is Ownable, IBranchBridgeAgentFactory {
         address _newBranchRouterAddress,
         address _rootBridgeAgentAddress,
         address _rootBridgeAgentFactoryAddress
-    ) external virtual returns (address newBridgeAgent) {
-        require(
-            msg.sender == localCoreBranchRouterAddress, "Only the Core Branch Router can create a new Bridge Agent."
-        );
-        require(
-            _rootBridgeAgentFactoryAddress == rootBridgeAgentFactoryAddress,
-            "Root Bridge Agent Factory Address does not match."
-        );
-
+    )
+        external
+        virtual
+        requireCoreBranchRouter
+        requireRootBridgeAgentFactory(_rootBridgeAgentFactoryAddress)
+        returns (address newBridgeAgent)
+    {
         newBridgeAgent = address(
             DeployBranchBridgeAgent.deploy(
                 rootChainId,
@@ -138,4 +136,19 @@ contract BranchBridgeAgentFactory is Ownable, IBranchBridgeAgentFactory {
 
         IPort(localPortAddress).addBridgeAgent(newBridgeAgent);
     }
+
+    modifier requireCoreBranchRouter() {
+        require(
+            msg.sender == localCoreBranchRouterAddress, "Only the Core Branch Router can create a new Bridge Agent."
+        );
+        _;
+    }
+
+    modifier requireRootBridgeAgentFactory(address _rootBridgeAgentFactoryAddress) {
+        require(
+            _rootBridgeAgentFactoryAddress == rootBridgeAgentFactoryAddress,
+            "Root Bridge Agent Factory Address does not match."
+        );
+        _;
+    }
 }
```

---

## 05. **Unnecessary modifiers for empty or revert functions. (5 instances)**

<a name="N-05"></a>

### Description: 
Some functions in the code either don't have any implementation or unconditionally revert. Applying a modifier to these functions is unnecessary.
### Solution: 
Remove the unnecessary modifiers from the empty or revert functions.

- src/CoreRootRouter.sol:[355](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/CoreRootRouter.sol#L355), [365](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/CoreRootRouter.sol#L365), [370](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/CoreRootRouter.sol#L370), [380](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/CoreRootRouter.sol#L380), [390](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/CoreRootRouter.sol#L390)

```diff
diff --git a/src/CoreRootRouter.sol b/src/CoreRootRouter.sol
index 3104a2a..8db63d1 100644
--- a/src/CoreRootRouter.sol
+++ b/src/CoreRootRouter.sol
@@ -351,7 +351,6 @@ contract CoreRootRouter is IRootRouter, Ownable {
         external
         payable
         override
-        requiresExecutor
     {
         revert();
     }
@@ -361,13 +360,12 @@ contract CoreRootRouter is IRootRouter, Ownable {
         external
         payable
         override
-        requiresExecutor
     {
         revert();
     }
 
     /// @inheritdoc IRootRouter
-    function executeSigned(bytes memory, address, uint16) external payable override requiresExecutor {
+    function executeSigned(bytes memory, address, uint16) external payable override {
         revert();
     }
 
@@ -376,7 +374,6 @@ contract CoreRootRouter is IRootRouter, Ownable {
         external
         payable
         override
-        requiresExecutor
     {
         revert();
     }
@@ -386,7 +383,6 @@ contract CoreRootRouter is IRootRouter, Ownable {
         external
         payable
         override
-        requiresExecutor
     {
         revert();
     }
```

---

## 06. **Duplicate function bodies. (1 instance)** 

<a name="N-06"></a>

### Description: 
Two functions in the code have exactly the same body and perform the same actions. This duplication can be avoided by merging the two functions into a single one.
### Solution: 
Merge the two functions into a single one and remove the duplicated code.

- src\MulticallRootRouter.sol:[321](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/MulticallRootRouter.sol#L321)

`executeSignedDepositSingle` have the same body as `executeSigned`

```diff
diff --git a/src/MulticallRootRouter.sol b/src/MulticallRootRouter.sol
index 0621f81..f392413 100644
--- a/src/MulticallRootRouter.sol
+++ b/src/MulticallRootRouter.sol
@@ -221,7 +221,7 @@ contract MulticallRootRouter is IRootRouter, Ownable {
      *
      */
     function executeSigned(bytes calldata encodedData, address userAccount, uint16)
-        external
+        public
         payable
         override
         lock
@@ -316,76 +316,7 @@ contract MulticallRootRouter is IRootRouter, Ownable {
         requiresExecutor
         lock
     {
-        // Parse funcId
-        bytes1 funcId = encodedData[0];
-
-        /// FUNC ID: 1 (multicallNoOutput)
-        if (funcId == 0x01) {
-            // Decode Params
-            Call[] memory calls = abi.decode(_decode(encodedData[1:]), (Call[]));
-
-            // Make requested calls
-            IVirtualAccount(userAccount).call(calls);
-
-            /// FUNC ID: 2 (multicallSingleOutput)
-        } else if (funcId == 0x02) {
-            // Decode Params
-            (Call[] memory calls, OutputParams memory outputParams, uint16 dstChainId, GasParams memory gasParams) =
-                abi.decode(_decode(encodedData[1:]), (Call[], OutputParams, uint16, GasParams));
-
-            // Make requested calls
-            IVirtualAccount(userAccount).call(calls);
-
-            // Withdraw assets from Virtual Account
-            IVirtualAccount(userAccount).withdrawERC20(outputParams.outputToken, outputParams.amountOut);
-
-            // Bridge Out assets
-            _approveAndCallOut(
-                IVirtualAccount(userAccount).userAddress(),
-                outputParams.recipient,
-                outputParams.outputToken,
-                outputParams.amountOut,
-                outputParams.depositOut,
-                dstChainId,
-                gasParams
-            );
-
-            /// FUNC ID: 3 (multicallMultipleOutput)
-        } else if (funcId == 0x03) {
-            // Decode Params
-            (
-                Call[] memory calls,
-                OutputMultipleParams memory outputParams,
-                uint16 dstChainId,
-                GasParams memory gasParams
-            ) = abi.decode(_decode(encodedData[1:]), (Call[], OutputMultipleParams, uint16, GasParams));
-
-            // Make requested calls
-            IVirtualAccount(userAccount).call(calls);
-
-            // Withdraw assets from Virtual Account
-            for (uint256 i = 0; i < outputParams.outputTokens.length;) {
-                IVirtualAccount(userAccount).withdrawERC20(outputParams.outputTokens[i], outputParams.amountsOut[i]);
-
-                unchecked {
-                    ++i;
-                }
-            }
-
-            // Bridge Out assets
-            _approveMultipleAndCallOut(
-                IVirtualAccount(userAccount).userAddress(),
-                outputParams.recipient,
-                outputParams.outputTokens,
-                outputParams.amountsOut,
-                outputParams.depositsOut,
-                dstChainId,
-                gasParams
-            );
-            /// UNRECOGNIZED FUNC ID
-        } else {
-            revert UnrecognizedFunctionId();
-        }
+        executeSigned(encodedData, userAccount, 0);
     }
 
     /**
```