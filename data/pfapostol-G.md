## Summary table

|     Index     | Issue                                                                                   | Instances |
| :-----------: | :-------------------------------------------------------------------------------------- | :-------: |
| [G-01](#G-01) | [Removal of redundant redefine in a loop](#G-01)                                                               |     1     |
| [G-02](#G-02) | [Caching repeating calculations](#G-02)                                                     |     1     |
| [G-03](#G-03) | [Extract common modifier to a library](#G-03)                                          |     5     |
| [G-04](#G-04) | [Merge duplicated statements into a modifier](#G-04)  |     2     |
| [G-05](#G-05) | [Remove unnecessary modifiers from empty/revert functions](#G-05) |     5     |

14 instances over 5 issues

---

## 1. **Removal of redundant redefine in a loop (1 instance)**

<a name="G-01"></a>

### Description: 
In the `VirtualAccount.sol` contract, the same `Call` struct is redefined inside the loop, which is unnecessary as it can be moved outside the loop for optimization.

By moving the `Call` struct definition outside the loop, unnecessary memory allocations and reassignments are avoided, resulting in gas savings.

### Gas Savings: 
The gas savings would vary depending on the number of loop iterations and the complexity of the `Call` struct, but it reduces gas cost by avoiding redundant memory operations.

- src/VirtualAccount.sol:[72](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/VirtualAccount.sol#L72))

```diff
diff --git a/src/VirtualAccount.sol b/src/VirtualAccount.sol
index f6a9134..1f02e61 100644
--- a/src/VirtualAccount.sol
+++ b/src/VirtualAccount.sol
@@ -67,9 +67,10 @@ contract VirtualAccount is IVirtualAccount, ERC1155Receiver {
         uint256 length = calls.length;
         returnData = new bytes[](length);
 
+        Call calldata _call;
         for (uint256 i = 0; i < length;) {
             bool success;
-            Call calldata _call = calls[i];
+            _call = calls[i];
 
             if (isContract(_call.target)) (success, returnData[i]) = _call.target.call(_call.callData);
```

---


## 2. **Caching repeating calculations (1 instance)**

<a name="G-02"></a>

### Description: 
In the `RootPort.sol` contract, the difference between `_amount` and `_deposit` is calculated multiple times within the same function. By caching the difference in a variable, gas consumption is reduced.


- src\RootPort.sol:[286](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src\RootPort.sol#L286))

```diff
diff --git a/src/RootPort.sol b/src/RootPort.sol
index c379b6e..aad1258 100644
--- a/src/RootPort.sol
+++ b/src/RootPort.sol
@@ -281,9 +281,10 @@ contract RootPort is Ownable, IRootPort {
     {
         if (!isGlobalAddress[_hToken]) revert UnrecognizedToken();
 
-        if (_amount - _deposit > 0) {
+        uint256 _diff = _amount - _deposit;
+        if (_diff > 0) {
             unchecked {
-                _hToken.safeTransfer(_recipient, _amount - _deposit);
+                _hToken.safeTransfer(_recipient, _diff);
             }
         }
 
```

---

## 3. **Extract common modifier to a library (5 instances)**

<a name="G-03"></a>

### Description: 
In multiple contracts, the same modifier is defined within each contract. Extracting the common modifier to a separate library reduces code duplication and optimizes gas consumption.

By creating a library that contains the shared modifier and importing it in the relevant contracts, code duplication is avoided. This results in reduced bytecode size and gas savings.

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

## 4. **Merge duplicated statements into a modifier (2 instances)**

<a name="G-04"></a>

### Description: 
Two contracts have duplicated statements in separate functions. Merging these duplicated statements into a single 
modifier eliminates redundancy and improves gas efficiency.

### Gas Savings: 
The gas savings depend on the number of times the duplicated statements are executed and the complexity of the statements themselves. By eliminating the redundancy, it reduces the overall gas cost for executing those statements.

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


## 5. **Remove unnecessary modifiers from empty/revert functions (5 instances)**

<a name="G-05"></a>

### Description: 
Several functions in the code either don't have any implementation or unconditionally revert. Applying modifiers to these functions is unnecessary and can be removed to optimize gas consumption.

By removing unnecessary modifiers from empty/revert functions, the bytecode size is reduced, resulting in gas savings.

### Gas Savings: 
The gas savings would depend on the complexity of the modifier and the number of functions affected. Removing unnecessary modifiers reduces the bytecode size, resulting in gas savings.

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

