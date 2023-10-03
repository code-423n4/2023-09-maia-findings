# [N-01] _remoteBranchExecutionGas is misleading
In several places, calls through LayerZero accept a "remoteBranchExecutionGas" parameter. This naming is misleading because it always refers to native airdropped amount, that always has to do more with bridging fees than with gas.

# [N-02] BranchPort bridgeOut and bridgeOutMultiple can revert to arithmetic underflow
When briding out tokens, BranchPort checks the difference between the provided `amount` and `deposit`.

```Solidity
    uint256 _hTokenAmount = _amount - _deposit;
```

When `deposit` is bigger than `amount`, the `bridgeOut` and `bridgeOutMultiple` functions will revert due to underflow. Since this error is rechable by user-facing functions, consider adding a `require` statement with a proper error message.

# [N-03] Named mappings in RootPort are misleading
Several mappings in RootPort have misleading variable names. Consider applying the following changes:

```diff
    // RootPort.sol:89
-    /// @notice ChainId -> Local Address -> Global Address
-    mapping(address chainId => mapping(uint256 localAddress => address globalAddress)) public getGlobalTokenFromLocal;
+    /// @notice Local Address -> ChainId -> Global Address
+    mapping(address localAddress => mapping(uint256 chainId => address globalAddress)) public getGlobalTokenFromLocal;

-    /// @notice ChainId -> Global Address -> Local Address
-    mapping(address chainId => mapping(uint256 globalAddress => address localAddress)) public getLocalTokenFromGlobal;
+    /// @notice Global Address -> ChainId -> Local Address
+    mapping(address globalAddress => mapping(uint256 chainId => address localAddress)) public getLocalTokenFromGlobal;

-    /// @notice ChainId -> Underlying Address -> Local Address
-    mapping(address chainId => mapping(uint256 underlyingAddress => address localAddress)) public
+    /// @notice Underlying Address -> ChainId -> Local Address
+    mapping(address underlyingAddress => mapping(uint256 chainId => address localAddress)) public
        getLocalTokenFromUnderlying;

    /// @notice Mapping from Local Address to Underlying Address.
-    mapping(address chainId => mapping(uint256 localAddress => address underlyingAddress)) public
+    mapping(address localAddress => mapping(uint256 chainId => address underlyingAddress)) public
        getUnderlyingTokenFromLocal;
```