### QA Report Issues List

- [x] **Low 01** → Head Overflow Bug in Calldata Tuple ABI-Reencoding
- [x] **Low 02** → Important `_fee` return value doesn’t check
- [x] **Low 03** → `remoteBranchExecutionGas`  must be lower than `gasLimit`
- [x] **Low 04** → If the same values are used for status values in the same contract, this may cause logic errors
- [x] **Low 05** → Project Upgrade and Stop Scenario should be
- [x] **Low 06** → Array lengths doesn't check  a few Struct
- [x] **Low 07**→ Missing Event for  initialize
- [x] **Non-Critical 08**→ The function updates addresses in bulk. This can lead to unnecessary transaction costs and complexity.

</br>
</br>

### [Low-01] Head Overflow Bug in Calldata Tuple ABI-Reencoding

There is a known security vulnerability between versions 0.5.8 - 0.8.16 of Solidity, details on it below

The effects of the bug manifest when a contract performs ABI-encoding of a tuple that meets all of the following conditions:
The last component of the tuple is a (potentially nested) statically-sized calldata array with the most base type being either uint or bytes32. E.g. bytes32[10] or uint[2][2][2].
The tuple contains at least one dynamic component. E.g. bytes or a struct containing a dynamic array.
The code is using ABI coder v2, which is the default since Solidity 0.8.0.


https://blog.soliditylang.org/2022/08/08/calldata-tuple-reencoding-head-overflow-bug/


```solidity
   3: pragma solidity ^0.8.0;

src\interfaces\IVirtualAccount.sol:
   7: struct Call {
   8:     address target;
   9:     bytes callData;
  10: }

src\VirtualAccount.sol:
  63  
  64  
  65:     /// @inheritdoc IVirtualAccount
  66:     function call(Call[] calldata calls) external override requiresApprovedCaller returns (bytes[] memory returnData) {
  67:         uint256 length = calls.length;
  68:         returnData = new bytes[](length);
  69: 
  70:         for (uint256 i = 0; i < length;) {
  71:             bool success;
  72:             Call calldata _call = calls[i];
  73: 
  74:             if (isContract(_call.target)) (success, returnData[i]) = _call.target.call(_call.callData);
  75: 
  76:             if (!success) revert CallFailed();
  77: 
  78:             unchecked {
  79:                 ++i;
  80:             }
  81:         }
  82:     }
```

#### Recommended Mitigation Steps

Because of this problem, I recommend coding the contract with fixed pragma instead of floating pragma for compiling with min 0.8.16 and higher versions. VirtualAccount contract can be compiled with floating pragma with all versions 0.8


</br>


### [Low-02]  Important `_fee` return value doesn’t check

RootBridgeAgent.getFeeEstimate() function calculate to estimate fee value,  but doesn’t check to  return `_fee` value for 0 value

0 value check can be doesn't  important mostly but this function  has critical return value.

```solidity

src\RootBridgeAgent.sol:
  138      /// @inheritdoc IRootBridgeAgent
  139      /// @inheritdoc IRootBridgeAgent
  140:     function getFeeEstimate(
  141:         uint256 _gasLimit,
  142:         uint256 _remoteBranchExecutionGas,
  143:         bytes calldata _payload,
  144:         uint16 _dstChainId
  145:     ) external view returns (uint256 _fee) {
  146:         (_fee,) = ILayerZeroEndpoint(lzEndpointAddress).estimateFees(    // @audit _fee value is not check
  147:             _dstChainId,
  148:             address(this),
  149:             _payload,
  150:             false,
  151:             abi.encodePacked(uint16(2), _gasLimit, _remoteBranchExecutionGas, getBranchBridgeAgent[_dstChainId])
  152:         );
  153:     }
```

### [Low-03] `remoteBranchExecutionGas`  must be lower than `gasLimit`.

This situation is clearly stated in `GasParams` struck (`remoteBranchExecutionGas` must be lower than `gasLimit`.)

```solidity
src\interfaces\BridgeAgentStructs.sol:
   7  
   8: struct GasParams {
   9:     uint256 gasLimit; // gas allocated for the cross-chain call.
  10:     uint256 remoteBranchExecutionGas; //gas allocated for remote branch execution. Must be lower than `gasLimit`.
  11: }


```

However, it is observed that this check is not made in many functions.

```solidity

 function callOut(bytes calldata _params, GasParams calldata _gParams) external payable override lock {
        IBridgeAgent(localBridgeAgentAddress).callOut{value: msg.value}(payable(msg.sender), _params, _gParams);
    } // @audit _gParams value doesn’t check for lowet than



 function retrieveDeposit(uint32 _depositNonce, GasParams calldata _gParams) external payable override lock {
        // @audit _gParams value doesn’t check for lowet than
        if (getDeposit[_depositNonce].owner != msg.sender) revert NotDepositOwner();

        bytes memory payload = abi.encodePacked(bytes1(0x08), msg.sender, _depositNonce);

        _performCall(payable(msg.sender), payload, _gParams);
    }

```

####  Recommended Mitigation Steps

The remoteBranchExecutionGas should be less than gasLimit. There should be a check to ensure this condition is always met to avoid running out of gas during execution.


### [Low-04] If the same values are used for status values in the same contract, this may cause logic errors.

Statuses are used in the same contract, but if the values are the same, this will be prone to errors, so use the Enum system

```solidity
src\interfaces\BridgeAgentConstants.sol:
   8   */
   9   */
  10: contract BridgeAgentConstants {
  11:     // Settlement / Deposit Execution Status
  12: 
  13:     uint8 internal constant STATUS_READY = 0;
  14: 
  15:     uint8 internal constant STATUS_DONE = 1;
  20: 
  21:     uint8 internal constant STATUS_FAILED = 1;
  22: 
  23:     uint8 internal constant STATUS_SUCCESS = 0;



```


### [Low-05] Project Upgrade and Stop Scenario should be

At the start of the project, the system may need to be stopped or upgraded, I suggest you have a script beforehand and add it to the documentation.
This can also be called an " EMERGENCY STOP (CIRCUIT BREAKER) PATTERN ".

This can be done by adding the 'pause' architecture, which is included in many projects, to critical functions, and by authorizing the existing onlyOwner.

https://github.com/maxwoe/solidity_patterns/blob/master/security/EmergencyStop.sol


### [Low-06] Array lengths doesn't check  a few Struct


`DepositMultipleInput` , ` DepositMultipleParams` , ` SettlementMultipleInput` , ` SettlementMultipleParams`  structs has potential lengths check issue

The lengths of the hTokens, tokens, amounts, and deposits arrays should be consistent. An inconsistency can lead to out-of-bounds errors or incorrect operations.

A modifier or a check to ensure that all array lengths are equal.



```solidity

src\interfaces\BridgeAgentStructs.sol:

  22: 
  29: 
  30: struct DepositMultipleInput {
  31:     //Deposit Info
  32:     address[] hTokens; //Input Local hTokens Address.
  33:     address[] tokens; //Input Native / underlying Token Address.
  34:     uint256[] amounts; //Amount of Local hTokens deposited for interaction.
  35:     uint256[] deposits; //Amount of native tokens deposited for interaction.
  36: }
  37: 
  38: struct DepositMultipleParams {
  39:     //Deposit Info
  40:     uint8 numberOfAssets; //Number of assets to deposit.
  41:     uint32 depositNonce; //Deposit nonce.
  42:     address[] hTokens; //Input Local hTokens Address.
  43:     address[] tokens; //Input Native / underlying Token Address.
  44:     uint256[] amounts; //Amount of Local hTokens deposited for interaction.
  45:     uint256[] deposits; //Amount of native tokens deposited for interaction.
  46: }

  73: 
  74: struct SettlementMultipleInput {
  75:     address[] globalAddresses; //Input Global hTokens Addresses.
  76:     uint256[] amounts; //Amount of Local hTokens deposited for interaction.
  77:     uint256[] deposits; //Amount of native tokens deposited for interaction.
  78: }
  79: 
  88: 
  89: struct SettlementMultipleParams {
  90:     uint8 numberOfAssets; // Number of assets to deposit.
  91:     address recipient; // Recipient of the settlement.
  92:     uint32 settlementNonce; // Settlement nonce.
  93:     address[] hTokens; // Input Local hTokens Addresses.
  94:     address[] tokens; // Input Native / underlying Token Addresses.
  95:     uint256[] amounts; // Amount of Local hTokens deposited for interaction.
  96:     uint256[] deposits; // Amount of native tokens deposited for interaction.
  97: }

```

### [Low-07] Missing Event for  initialize

**Context:**
```solidity
10 results - 9 files

src\BaseBranchRouter.sol:
  59       */
  60:     function initialize(address _localBridgeAgentAddress) external onlyOwner {
  61          require(_localBridgeAgentAddress != address(0), "Bridge Agent address cannot be 0");

src\BranchPort.sol:
  121       */
  122:     function initialize(address _coreBranchRouter, address _bridgeAgentFactory) external virtual onlyOwner {
  123          require(coreBranchRouterAddress == address(0), "Contract already initialized");

src\CoreRootRouter.sol:
  82  
  83:     function initialize(address _bridgeAgentAddress, address _hTokenFactory) external onlyOwner {
  84          require(_setup, "Contract is already initialized");

src\MulticallRootRouter.sol:
  108       */
  109:     function initialize(address _bridgeAgentAddress) external onlyOwner {
  110          require(_bridgeAgentAddress != address(0), "Bridge Agent Address cannot be 0");

src\RootPort.sol:
  128       */
  129:     function initialize(address _bridgeAgentFactory, address _coreRootRouter) external onlyOwner {
  130          require(_bridgeAgentFactory != address(0), "Bridge Agent Factory cannot be 0 address.");

  146       */
  147:     function initializeCore(
  148          address _coreRootBridgeAgent,

src\factories\ArbitrumBranchBridgeAgentFactory.sol:
  55       */
  56:     function initialize(address _coreRootBridgeAgent) external override onlyOwner {
  57          require(_coreRootBridgeAgent != address(0), "Core Root Bridge Agent Address cannot be 0");

src\factories\BranchBridgeAgentFactory.sol:
  86       */
  87:     function initialize(address _coreRootBridgeAgent) external virtual onlyOwner {
  88          require(_coreRootBridgeAgent != address(0), "Core Root Bridge Agent cannot be 0");

src\factories\ERC20hTokenBranchFactory.sol:
  59       */
  60:     function initialize(address _wrappedNativeTokenAddress, address _coreRouter) external onlyOwner {
  61          require(_coreRouter != address(0), "CoreRouter address cannot be 0");

src\factories\ERC20hTokenRootFactory.sol:
  48       */
  49:     function initialize(address _coreRouter) external onlyOwner {
  50          require(_coreRouter != address(0), "CoreRouter address cannot be 0");

```

**Description:**
Events help non-contract tools to track changes, and events prevent users from being surprised by changes
Issuing event-emit during initialization is a detail that many projects skip

**Recommendation:**
Add Event-Emit




### [Non Critical-08] The function updates addresses in bulk. This can lead to unnecessary transaction costs and complexity.
The function checks addresses at startup to check for invalid addresses, this is good practice. However, this check must be repeated each time the address is updated.

Although this is not a security vulnerability, mass updating of addresses can increase the risk of accidentally entering an incorrect address. Additionally, this type of update may increase the possibility of a malicious user manipulating the system.


```solidity
src\RootPort.sol:
  237      /// @inheritdoc IRootPort
  238      /// @inheritdoc IRootPort
  239:     function setAddresses(
  240:         address _globalAddress,
  241:         address _localAddress,
  242:         address _underlyingAddress,
  243:         uint256 _srcChainId
  244:     ) external override requiresCoreRootRouter {
  245:         if (_globalAddress == address(0)) revert InvalidGlobalAddress();
  246:         if (_localAddress == address(0)) revert InvalidLocalAddress();
  247:         if (_underlyingAddress == address(0)) revert InvalidUnderlyingAddress();
  248: 
  249:         isGlobalAddress[_globalAddress] = true;
  250:         getGlobalTokenFromLocal[_localAddress][_srcChainId] = _globalAddress;
  251:         getLocalTokenFromGlobal[_globalAddress][_srcChainId] = _localAddress;
  252:         getLocalTokenFromUnderlying[_underlyingAddress][_srcChainId] = _localAddress;
  253:         getUnderlyingTokenFromLocal[_localAddress][_srcChainId] = _underlyingAddress;
  254: 
  255:         emit LocalTokenAdded(_underlyingAddress, _localAddress, _globalAddress, _srcChainId);
  256:     }


```


**Recommendation:**
Add stricter checks for address updates. For example, check if an address is already registered and only update new or changed addresses.








