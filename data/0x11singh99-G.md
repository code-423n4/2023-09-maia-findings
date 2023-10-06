## Gas Optimizations

| Number                                                                                        | Issue                                                                            | Instances |
| --------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------- | :-------: |
| [[G-01](#g-01-state-variables-can-be-packed-to-save-storage-slots)]                           | State variables can be packed to save storage slots.                             |     5     |
| [[G-02](#g-02-use-calldata-instead-of-memory-for-function-arguments-that-do-not-get-mutated)] | Use `calldata` instead of `memory` for function arguments that do not get mutate |    21     |
| [[G-03](#g-03-do-not-cache-immutable-values)]                                                 | Do not cache immutable values                                                    |     7     |
| [[G-04](#g-04-no-need-to-explicitly-initialize-variables-with-default-values)]                | No need to explicitly initialize variables with default values                   |    14     |
| [[G-05](#g-05-using-ternary-operator-instead-of-if-else-saves-gas)]                           | Using `ternary` operator instead of if else saves gas                            |     1     |

## [G-01] State variables can be packed to save storage slots.

Each slot saved can avoid an extra Gsset (20000 gas) for the first setting of the state variable and subsequent reads as well, as writes have smaller gas savings.
The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to eachother in storage and this will pack the values together into a single 32 byte storage slot (if the values combined are <= 32 bytes). If the variables packed together are retrieved together in functions, we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a Gwarmaccess (100 gas) versus a Gcoldsload (2100 gas).

**_5 Instances - 5 Files_**

### Reduce uint type for `_unlocked` to `uint8` and pack with `bridgeAgentExecutorAddress` to save 1 SLOT

Since `_unlocked` is holding only 2 values 1 and 2 so uint8 is more than sufficient to hold them whenever it can be packed with another state variable to save 1 storage SLOT.

```solidity
File : src/BaseBranchRouter.sol

39: address public override bridgeAgentExecutorAddress;

   //@audit reduce it to uint8 pack with above
42: uint256 internal _unlocked = 1;

```

[39-42](https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L39-L42)

### Reduce uint type for `_unlocked` to `uint8` and pack with `depositNonce` to save 1 SLOT

```solidity
File : src/BranchBridgeAgent.sol

84:  uint32 public depositNonce;

      ...

101:  uint256 internal _unlocked = 1;

```

[101](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L101)

```diff
File : src/BranchBridgeAgent.sol

84:   uint32 public depositNonce;
+     uint8 internal _unlocked = 1;

      /// @notice Mapping from Pending deposits hash to Deposit Struct.
      mapping(uint256 depositNonce => Deposit depositInfo) public getDeposit;

      /*///////////////////////////////////////////////////////////////
                        SETTLEMENT EXECUTION STATE
      //////////////////////////////////////////////////////////////*/

      /// @notice If true, the bridge agent has already served a request with this nonce from a given chain.
      mapping(uint256 settlementNonce => uint256 state) public executionState;

      /*///////////////////////////////////////////////////////////////
                           REENTRANCY STATE
      //////////////////////////////////////////////////////////////*/

      /// @notice Re-entrancy lock modifier state.
- 101:  uint256 internal _unlocked = 1;


```

### Reduce uint type for `_unlocked` to `uint8` and pack with `coreBranchRouterAddress` to save 1 SLOT

```solidity
File : src/BranchPort.sol

25:  address public coreBranchRouterAddress;

91: uint256 internal _unlocked = 1;

```

[91](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L91)

### Reduce uint type for `_unlocked` to `uint8` and pack with `bridgeAgentExecutorAddress` to save 1 SLOT

```solidity
File : src/MulticallRootRouter.sol

77: address public bridgeAgentExecutorAddress;

80: uint256 internal _unlocked = 1;

```

[80](https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L80)

### Reduce uint type for `_unlocked` to `uint8` and pack with `settlementNonce` to save 1 SLOT

```solidity
File : src/RootBridgeAgent.sol

75: uint32 public settlementNonce;

92: uint256 internal _unlocked = 1;

```

[92](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L92)

## [G-02] Use `calldata` instead of `memory` for function arguments that do not get mutated.

When you specify a data location as memory, that value will be copied into memory. When you specify the location as calldata, the value will stay static within calldata. If the value is a large, complex type, using memory may result in extra memory expansion costs.

**Note: These are instances missed by the Bot Race.**

**_21 Instances - 6 Files_**

<details>
<summary>see instances</summary>

```solidity

File : src/factories/ERC20hTokenBranchFactory.sol

96:  function createToken(string memory _name, string memory _symbol, uint8 _decimals, bool _addPrefix)

```

[96](https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenBranchFactory.sol#L96)

```diff
File : src/factories/ERC20hTokenBranchFactory.sol

-  function createToken(string memory _name, string memory _symbol, uint8 _decimals, bool _addPrefix)
+  function createToken(string calldata _name, string calldata _symbol, uint8 _decimals, bool _addPrefix)

```

```solidity

File : src/factories/ERC20hTokenRootFactory.sol

76:   function createToken(string memory _name, string memory _symbol, uint8 _decimals)

```

[76](https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenRootFactory.sol#L76)

```diff
File : src/factories/ERC20hTokenRootFactory.sol

-   function createToken(string memory _name, string memory _symbol, uint8 _decimals)
+   function createToken(string calldata _name, string calldata _symbol, uint8 _decimals)

```

```solidity

File : src/ArbitrumCoreBranchRouter.sol

85:     function _receiveAddBridgeAgent(
        address _newBranchRouter,
        address _branchBridgeAgentFactory,
        address _rootBridgeAgent,
        address _rootBridgeAgentFactory,
        address _refundee,
        GasParams memory _gParams
92:    ) internal override {

```

[85-92](https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumCoreBranchRouter.sol#L85C1-L92C1)

```diff
File : src/ArbitrumCoreBranchRouter.sol

       function _receiveAddBridgeAgent(
        address _newBranchRouter,
        address _branchBridgeAgentFactory,
        address _rootBridgeAgent,
        address _rootBridgeAgentFactory,
        address _refundee,
-       GasParams memory _gParams
+       GasParams calldata _gParams
      ) internal override {

```

```solidity

File : src/BranchBridgeAgent.sol

860:  function _createDepositMultiple(
        uint32 _depositNonce,
        address payable _refundee,
        address[] memory _hTokens,
        address[] memory _tokens,
        uint256[] memory _amounts,
        uint256[] memory _deposits
867:   ) internal {

```

[860-867](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L860C5-L867C17)

```diff
File : src/BranchBridgeAgent.sol

    function _createDepositMultiple(
        uint32 _depositNonce,
        address payable _refundee,
-        address[] memory _hTokens,
-       address[] memory _tokens,
-       uint256[] memory _amounts,
-       uint256[] memory _deposits
+       address[] calldata _hTokens,
+       address[] calldata _tokens,
+       uint256[] calldata _amounts,
+       uint256[] calldata _deposits
    ) internal {

```

```solidity

File: src/BranchPort.sol

246:  function bridgeInMultiple(
          address _recipient,
          address[] memory _localAddresses,
          address[] memory _underlyingAddresses,
          uint256[] memory _amounts,
          uint256[] memory _deposits
252:   ) external override requiresBridgeAgent {

```

[246-252](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L246C5-L252C46)

```diff
File: src/BranchPort.sol

     function bridgeInMultiple(
          address _recipient,
-         address[] memory _localAddresses,
-         address[] memory _underlyingAddresses,
-         uint256[] memory _amounts,
-         uint256[] memory _deposits
+         address[] memory _localAddresses,
+         address[] memory _underlyingAddresses,
+         uint256[] memory _amounts,
+         uint256[] memory _deposits
      ) external override requiresBridgeAgent {

```

```solidity

File : src/BranchPort.sol

288:  function bridgeOutMultiple(
        address _depositor,
        address[] memory _localAddresses,
        address[] memory _underlyingAddresses,
        uint256[] memory _amounts,
        uint256[] memory _deposits
294:  ) external override lock requiresBridgeAgent {

```

[288-294](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L288C5-L294C51)

```diff
File :

    function bridgeOutMultiple(
        address _depositor,
-       address[] memory _localAddresses,
-       address[] memory _underlyingAddresses,
-       uint256[] memory _amounts,
-       uint256[] memory _deposits
+       address[] calldata _localAddresses,
+       address[] calldata _underlyingAddresses,
+       uint256[] calldata _amounts,
+       uint256[] calldata _deposits
    ) external override lock requiresBridgeAgent {

```

```solidity

File : src/RootBridgeAgent.sol

856:  function _performRetrySettlementCall(
        bool _hasFallbackToggled,
        address[] memory _hTokens,
        address[] memory _tokens,
        uint256[] memory _amounts,
        uint256[] memory _deposits,
        bytes memory _params,
        uint32 _settlementNonce,
        address payable _refundee,
        address _recipient,
        uint16 _dstChainId,
        GasParams memory _gParams,
        uint256 _value
869:    ) internal {

```

[856-869](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L856C4-L869C17)

```diff
File : src/RootBridgeAgent.sol

    function _performRetrySettlementCall(
        bool _hasFallbackToggled,
-       address[] memory _hTokens,
-       address[] memory _tokens,
-       uint256[] memory _amounts,
-       uint256[] memory _deposits,
-       bytes memory _params,
+       address[] calldata _hTokens,
+       address[] calldata _tokens,
+       uint256[] calldata _amounts,
+       uint256[] calldata _deposits,
+       bytes calldata _params,
        uint32 _settlementNonce,
        address payable _refundee,
        address _recipient,
        uint16 _dstChainId,
-       GasParams memory _gParams,
+       GasParams calldata _gParams,
        uint256 _value
    ) internal {

```

</details>

## [G-03] Do not cache immutable values

**_7 Instances - 1 Files_**

```solidity

File : src/ArbitrumBranchPort.sol

50:  function depositToPort(address _depositor, address _recipient, address _underlyingAddress, uint256 _deposit)
51:       external
52:       override
53:       lock
54:       requiresBridgeAgent
55:  {
56:     // Save root port address to memory
57:     address _rootPortAddress = rootPortAddress;
58:
59:     // Get global token address from root port
60:    address _globalToken = IRootPort(_rootPortAddress).getLocalTokenFromUnderlying(_underlyingAddress, localChainId);
61:
62:     // Check if the global token exists
63:     if (_globalToken == address(0)) revert UnknownGlobalToken();
64:
65:     // Deposit Assets to Port
66:        _underlyingAddress.safeTransferFrom(_depositor, address(this), _deposit);
67:
68:     // Request Minting of Global Token
69:     IRootPort(_rootPortAddress).mintToLocalBranch(_recipient, _globalToken, _deposit);
70:  }

```

[50-70](https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol#L50C4-L70C6)

```diff

File : src/ArbitrumBranchPort.sol

    function depositToPort(address _depositor, address _recipient, address _underlyingAddress, uint256 _deposit)
        external
        override
        lock
        requiresBridgeAgent
    {
        // Save root port address to memory
-       address _rootPortAddress = rootPortAddress;

        // Get global token address from root port
-      address _globalToken = IRootPort(_rootPortAddress).getLocalTokenFromUnderlying(_underlyingAddress, localChainId);
+      address _globalToken = IRootPort(rootPortAddress).getLocalTokenFromUnderlying(_underlyingAddress, localChainId);

        // Check if the global token exists
        if (_globalToken == address(0)) revert UnknownGlobalToken();

        // Deposit Assets to Port
        _underlyingAddress.safeTransferFrom(_depositor, address(this), _deposit);

        // Request Minting of Global Token
-        IRootPort(_rootPortAddress).mintToLocalBranch(_recipient, _globalToken, _deposit);
+        IRootPort(rootPortAddress).mintToLocalBranch(_recipient, _globalToken, _deposit);
    }


```

```solidity

File : src/ArbitrumBranchPort.sol

73: function withdrawFromPort(address _depositor, address _recipient, address _globalAddress, uint256 _amount)
        external
        override
        lock
        requiresBridgeAgent
    {
        // Save root port address to memory
        address _rootPortAddress = rootPortAddress;

        // Check if the global token exists
        if (!IRootPort(_rootPortAddress).isGlobalToken(_globalAddress, localChainId)) revert UnknownGlobalToken();

        // Get the underlying token address from the root port
        address _underlyingAddress =
            IRootPort(_rootPortAddress).getUnderlyingTokenFromLocal(_globalAddress, localChainId);

        // Check if the underlying token exists
        if (_underlyingAddress == address(0)) revert UnknownUnderlyingToken();

        IRootPort(_rootPortAddress).burnFromLocalBranch(_depositor, _globalAddress, _amount);

        _underlyingAddress.safeTransfer(_recipient, _amount);
95: }

```

[73-95](https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol#L73C5-L95C6)

```diff

File : src/ArbitrumBranchPort.sol

    function withdrawFromPort(address _depositor, address _recipient, address _globalAddress, uint256 _amount)
        external
        override
        lock
        requiresBridgeAgent
    {
        // Save root port address to memory
-        address _rootPortAddress = rootPortAddress;

        // Check if the global token exists
-        if (!IRootPort(_rootPortAddress).isGlobalToken(_globalAddress, localChainId)) revert UnknownGlobalToken();
+        if (!IRootPort(rootPortAddress).isGlobalToken(_globalAddress, localChainId)) revert UnknownGlobalToken();

        // Get the underlying token address from the root port
        address _underlyingAddress =
-           IRootPort(_rootPortAddress).getUnderlyingTokenFromLocal(_globalAddress, localChainId);
+           IRootPort(rootPortAddress).getUnderlyingTokenFromLocal(_globalAddress, localChainId);

        // Check if the underlying token exists
        if (_underlyingAddress == address(0)) revert UnknownUnderlyingToken();

-       IRootPort(_rootPortAddress).burnFromLocalBranch(_depositor, _globalAddress, _amount);
+       IRootPort(rootPortAddress).burnFromLocalBranch(_depositor, _globalAddress, _amount);

        _underlyingAddress.safeTransfer(_recipient, _amount);
    }


```

## [G-04] No need to explicitly initialize variables with default values.

If a variable is not set/initialized, it is assumed to have the default value (0 for uint, false for bool, address(0) for address, etc.). Explicitly initializing it with its default value is an anti-pattern and wastes gas.

### _`for (uint256 i = 0; i < numIterations; ++i)`_ should be replaced with: _`for (uint256 i; i < numIterations; ++i)`_

**_14 Instances - 7 Files_**

<details>
<summary>see instances</summary>

```diff
File : src/BaseBranchRouter.sol

- 192:  for (uint256 i = 0; i < _hTokens.length;) {
+ 192:  for (uint256 i; i < _hTokens.length;) {

```

[192](https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L192C8-L192C52)

```diff
File : src/BranchBridgeAgent.sol

- 513:  for (uint256 i = 0; i < numOfAssets;) {
+ 513:  for (uint256 i; i < numOfAssets;) {

```

[513](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L513)

```diff
File : src/BranchPort.sol

- 257:  for (uint256 i = 0; i < length;) {
+ 257:  for (uint256 i; i < length;) {

```

[257](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L257C9-L257C43)

```diff
File :  src/BranchPort.sol

- 305:  for (uint256 i = 0; i < length;) {
+ 305:  for (uint256 i; i < length;) {

```

[305](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L305C9-L305C43)

```diff
File : src/MulticallRootRouter.sol

- 278:  for (uint256 i = 0; i < outputParams.outputTokens.length;) {
+ 278:  for (uint256 i; i < outputParams.outputTokens.length;) {

```

[278](https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L278C12-L278C73)

```diff
File : src/MulticallRootRouter.sol

- 367:   for (uint256 i = 0; i < outputParams.outputTokens.length;) {
+ 367:   for (uint256 i; i < outputParams.outputTokens.length;) {

```

[367](https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L367C11-L367C73)

```diff
File : src/MulticallRootRouter.sol

- 455:  for (uint256 i = 0; i < outputParams.outputTokens.length;) {
+ 455:  for (uint256 i; i < outputParams.outputTokens.length;) {

```

[455](https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L455)

```diff
File : src/MulticallRootRouter.sol

- 557:  for (uint256 i = 0; i < outputTokens.length;) {
+ 557:  for (uint256 i; i < outputTokens.length;) {

```

[557](https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L557)

```diff
File : src/RootBridgeAgent.sol

- 318  for (uint256 i = 0; i < settlement.hTokens.length;) {
+ 318  for (uint256 i; i < settlement.hTokens.length;) {

```

[318](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L318)

```diff
File : src/RootBridgeAgent.sol

- 399:  for (uint256 i = 0; i < length;) {
+ 399:  for (uint256 i; i < length;) {

```

[399](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L399)

```diff
File : src/RootBridgeAgent.sol

- 1070:  for (uint256 i = 0; i < hTokens.length;) {
+ 1070:  for (uint256 i; i < hTokens.length;) {

```

[1070](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1070C9-L1070C51)

```diff
File : src/RootBridgeAgentExecutor.sol

- 281:  for (uint256 i = 0; i < uint256(uint8(numOfAssets));) {
+ 281:  for (uint256 i; i < uint256(uint8(numOfAssets));) {

```

[281](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L281C7-L281C64)

```diff
File : src/VirtualAccount.sol

- 70:  for (uint256 i = 0; i < length;) {
+ 70:  for (uint256 i; i < length;) {

```

[70](https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L70)

```diff
File : src/VirtualAccount.sol

- 90:  for (uint256 i = 0; i < length;) {
+ 90:  for (uint256 i; i < length;) {

```

[90](https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L90C9-L90C43)

</details>

## [G-05] Using `ternary` operator instead of if else saves gas

When using the if-else construct, Solidity generates bytecode that includes a JUMP operation. The JUMP operation requires the EVM to change the program counter to a different location in the bytecode, resulting in additional gas costs. On the other hand, the ternary operator doesn't involve a JUMP operation, as it generates bytecode that utilizes conditional stack manipulation.

**_1 Instance - 1 File_**

```solidity

File :src/CoreBranchRouter.sol

284:  function _manageStrategyToken(address _underlyingToken, uint256 _minimumReservesRatio) internal {
        // Save Port Address to memory
        address _localPortAddress = localPortAddress;

        // Check if the token is an active Strategy Token
        if (IPort(_localPortAddress).isStrategyToken(_underlyingToken)) {
            // If so, toggle it off.
            IPort(_localPortAddress).toggleStrategyToken(_underlyingToken);
        } else {
            // If not, add it.
            IPort(_localPortAddress).addStrategyToken(_underlyingToken, _minimumReservesRatio);
        }
296:  }

```

[284-296](https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L284C3-L296C6)

```diff
File :src/CoreBranchRouter.sol

  function _manageStrategyToken(address _underlyingToken, uint256 _minimumReservesRatio) internal {
        // Save Port Address to memory
        address _localPortAddress = localPortAddress;

        // Check if the token is an active Strategy Token
-        if (IPort(_localPortAddress).isStrategyToken(_underlyingToken)) {
-           // If so, toggle it off.
-           IPort(_localPortAddress).toggleStrategyToken(_underlyingToken);
-       } else {
-           // If not, add it.
-           IPort(_localPortAddress).addStrategyToken(_underlyingToken, _minimumReservesRatio);
-       }
+        IPort(_localPortAddress).isStrategyToken(_underlyingToken)
+        ? IPort(_localPortAddress).toggleStrategyToken(_underlyingToken)
+        : IPort(_localPortAddress).addStrategyToken(_underlyingToken, _minimumReservesRatio);
  }

```
