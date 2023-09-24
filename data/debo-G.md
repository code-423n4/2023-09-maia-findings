## [G-01] Don't Initialize Variables with Default Value

## Description

In the provided code snippets, there is a gas optimization vulnerability related to the initialization of variables with default values. Gas optimization is a critical aspect of Ethereum smart contract development because excessive gas usage can lead to high transaction costs and inefficient execution on the Ethereum network. This vulnerability occurs when developers unnecessarily initialize variables with their default values, which consumes extra gas.

## Vulnerable Code Snippets

Here are the vulnerable code snippets with unnecessary variable initialization:

1. **BaseBranchRouter.sol** - Line 192
   ```solidity
   for (uint256 i = 0; i < _hTokens.length;) {
   ```

2. **BranchBridgeAgent.sol** - Line 447
   ```solidity
   for (uint256 i = 0; i < deposit.tokens.length;) {
   ```

3. **BranchBridgeAgent.sol** - Line 513
   ```solidity
   for (uint256 i = 0; i < numOfAssets;) {
   ```

4. **BranchPort.sol** - Line 257
   ```solidity
   for (uint256 i = 0; i < length;) {
   ```

5. **BranchPort.sol** - Line 305
   ```solidity
   for (uint256 i = 0; i < length;) {
   ```

6. **MulticallRootRouter.sol** - Line 278
   ```solidity
   for (uint256 i = 0; i < outputParams.outputTokens.length;) {
   ```

7. **MulticallRootRouter.sol** - Line 367
   ```solidity
   for (uint256 i = 0; i < outputParams.outputTokens.length;) {
   ```

8. **MulticallRootRouter.sol** - Line 455
   ```solidity
   for (uint256 i = 0; i < outputParams.outputTokens.length;) {
   ```

9. **MulticallRootRouter.sol** - Line 557
   ```solidity
   for (uint256 i = 0; i < outputTokens.length;) {
   ```

10. **RootBridgeAgent.sol** - Line 318
    ```solidity
    for (uint256 i = 0; i < settlement.hTokens.length;) {
    ```

11. **RootBridgeAgent.sol** - Line 399
    ```solidity
    for (uint256 i = 0; i < length;) {
    ```

12. **RootBridgeAgent.sol** - Line 1070
    ```solidity
    for (uint256 i = 0; i < hTokens.length;) {
    ```

13. **RootBridgeAgentExecutor.sol** - Line 281
    ```solidity
    for (uint256 i = 0; i < uint256(uint8(numOfAssets));) {
    ```

14. **VirtualAccount.sol** - Line 70
    ```solidity
    for (uint256 i = 0; i < length;) {
    ```

15. **VirtualAccount.sol** - Line 90
    ```solidity
    for (uint256 i = 0; i < length;) {
    ```

## Ethical Example Exploit - Proof of Concept (PoC)

To demonstrate the impact of this gas optimization vulnerability, let's consider an ethical example exploit that exploits this issue to waste gas on the Ethereum network. 

```solidity
// Exploit contract
pragma solidity ^0.8.0;

contract GasWaster {
    function wasteGas() external {
        for (uint256 i = 0; i < 1000; i++) {
            // No operation, just waste gas
        }
    }
}
```

In this PoC, an attacker deploys a contract called `GasWaster`, which contains a loop that iterates 1000 times without performing any meaningful operation. By invoking the `wasteGas` function of this contract multiple times, an attacker can exploit the vulnerable contracts mentioned above by causing them to consume unnecessary gas due to the initialization of variables with default values. This can lead to increased transaction costs for users of these contracts.

## Mitigation

To mitigate this gas optimization vulnerability, developers should avoid initializing variables with their default values when unnecessary. Simply removing the initialization statement (e.g., `uint256 i = 0;`) from the vulnerable code snippets will help conserve gas and improve the efficiency of smart contracts.

## References
```solidity
202309-maia/src/BaseBranchRouter.sol::192 => for (uint256 i = 0; i < _hTokens.length;) {
202309-maia/src/BranchBridgeAgent.sol::447 => for (uint256 i = 0; i < deposit.tokens.length;) {
202309-maia/src/BranchBridgeAgent.sol::513 => for (uint256 i = 0; i < numOfAssets;) {
202309-maia/src/BranchPort.sol::257 => for (uint256 i = 0; i < length;) {
202309-maia/src/BranchPort.sol::305 => for (uint256 i = 0; i < length;) {
202309-maia/src/MulticallRootRouter.sol::278 => for (uint256 i = 0; i < outputParams.outputTokens.length;) {
202309-maia/src/MulticallRootRouter.sol::367 => for (uint256 i = 0; i < outputParams.outputTokens.length;) {
202309-maia/src/MulticallRootRouter.sol::455 => for (uint256 i = 0; i < outputParams.outputTokens.length;) {
202309-maia/src/MulticallRootRouter.sol::557 => for (uint256 i = 0; i < outputTokens.length;) {
202309-maia/src/RootBridgeAgent.sol::318 => for (uint256 i = 0; i < settlement.hTokens.length;) {
202309-maia/src/RootBridgeAgent.sol::399 => for (uint256 i = 0; i < length;) {
202309-maia/src/RootBridgeAgent.sol::1070 => for (uint256 i = 0; i < hTokens.length;) {
202309-maia/src/RootBridgeAgentExecutor.sol::281 => for (uint256 i = 0; i < uint256(uint8(numOfAssets));) {
202309-maia/src/VirtualAccount.sol::70 => for (uint256 i = 0; i < length;) {
202309-maia/src/VirtualAccount.sol::90 => for (uint256 i = 0; i < length;) {
```
## [G-02] Cache Array Length Outside of Loop
## Description
Caching the array length outside a loop is a common optimization technique in Solidity to reduce gas consumption, as reading the array length on each iteration can be costly. However, this optimization is safe only if the array's length remains constant throughout the loop execution. If the array's length changes within the loop, caching the length outside the loop could lead to incorrect behavior or vulnerabilities.

## Impact

The impact of not caching the array length outside of a loop varies depending on the specific use case and whether the array length can change during the loop execution. Here are some potential impacts:

1. **Gas Consumption**: Without caching the array length, reading it on each iteration of the loop can consume unnecessary gas, making the contract more expensive to execute.

2. **Inefficiency**: Inefficient gas usage can make the contract less economical for users, discouraging participation.

3. **Vulnerability to Reentrancy**: If the loop includes any external function calls, not caching the array length can introduce reentrancy vulnerabilities, where malicious contracts can manipulate the array's length to disrupt the contract's logic.

4. **Incorrect Behavior**: If the array length changes within the loop, it can lead to incorrect contract behavior, which might not always result in a security vulnerability but can still lead to undesirable outcomes.

## Ethical Example Exploit PoC

Below is a Proof of Concept (PoC) for an ethical example exploit in the `202309-maia/src/BranchBridgeAgent.sol::447` code snippet:

```solidity
// Assume this is the deposit.tokens array in BranchBridgeAgent.sol
address[] public tokens;

// Ethical Example Exploit PoC
function maliciousFunction() public {
    // Suppose the tokens array length can be manipulated by a malicious user
    // Inside the loop, the array length is not cached outside the loop
    for (uint256 i = 0; i < tokens.length;) {
        // Malicious user changes the array length
        if (tokens.length > 10) {
            // Perform malicious actions based on the manipulated array length
            // ...
            // An attacker could exploit this by causing unexpected behavior due to the changing array length.
        }
    }
}
```

In this PoC, a malicious user can manipulate the `tokens` array's length during the loop execution, potentially causing unexpected behavior or vulnerabilities in the contract.

## Mitigation

To mitigate this vulnerability, you should ensure that the array length is cached outside the loop if the array's length can change during the loop execution. Additionally, thoroughly test the contract to ensure that it behaves as expected and does not have any vulnerabilities related to this optimization.

It's essential to follow best practices for gas optimization and security in Solidity smart contract development to reduce the risk of vulnerabilities and ensure the contract's reliability and security.

## References
```sol
202309-maia/src/BaseBranchRouter.sol::192 => for (uint256 i = 0; i < _hTokens.length;) {
202309-maia/src/BranchBridgeAgent.sol::243 => uint8(_dParams.hTokens.length),
202309-maia/src/BranchBridgeAgent.sol::320 => uint8(_dParams.hTokens.length),
202309-maia/src/BranchBridgeAgent.sol::359 => if (uint8(deposit.hTokens.length) == 1) {
202309-maia/src/BranchBridgeAgent.sol::383 => } else if (uint8(deposit.hTokens.length) > 1) {
202309-maia/src/BranchBridgeAgent.sol::389 => uint8(deposit.hTokens.length),
202309-maia/src/BranchBridgeAgent.sol::400 => uint8(deposit.hTokens.length),
202309-maia/src/BranchBridgeAgent.sol::412 => if (payload.length == 0) revert DepositRetryUnavailableUseCallout();
202309-maia/src/BranchBridgeAgent.sol::447 => for (uint256 i = 0; i < deposit.tokens.length;) {
202309-maia/src/BranchBridgeAgent.sol::869 => if (_hTokens.length > MAX_TOKENS_LENGTH) revert InvalidInput();
202309-maia/src/BranchBridgeAgent.sol::870 => if (_hTokens.length != _tokens.length) revert InvalidInput();
202309-maia/src/BranchBridgeAgent.sol::871 => if (_tokens.length != _amounts.length) revert InvalidInput();
202309-maia/src/BranchBridgeAgent.sol::872 => if (_amounts.length != _deposits.length) revert InvalidInput();
202309-maia/src/BranchBridgeAgent.sol::942 => if (_srcAddress.length != 40) revert LayerZeroUnauthorizedCaller();
202309-maia/src/BranchBridgeAgentExecutor.sol::87 => if (_payload.length > PARAMS_SETTLEMENT_OFFSET) {
202309-maia/src/BranchBridgeAgentExecutor.sol::119 => if (_payload.length > settlementEndOffset) {
202309-maia/src/BranchPort.sol::254 => uint256 length = _localAddresses.length;
202309-maia/src/BranchPort.sol::257 => for (uint256 i = 0; i < length;) {
202309-maia/src/BranchPort.sol::296 => uint256 length = _localAddresses.length;
202309-maia/src/BranchPort.sol::299 => if (length > 255) revert InvalidInputArrays();
202309-maia/src/BranchPort.sol::300 => if (length != _underlyingAddresses.length) revert InvalidInputArrays();
202309-maia/src/BranchPort.sol::301 => if (_underlyingAddresses.length != _amounts.length) revert InvalidInputArrays();
202309-maia/src/BranchPort.sol::302 => if (_amounts.length != _deposits.length) revert InvalidInputArrays();
202309-maia/src/BranchPort.sol::305 => for (uint256 i = 0; i < length;) {
202309-maia/src/MulticallRootRouter.sol::278 => for (uint256 i = 0; i < outputParams.outputTokens.length;) {
202309-maia/src/MulticallRootRouter.sol::367 => for (uint256 i = 0; i < outputParams.outputTokens.length;) {
202309-maia/src/MulticallRootRouter.sol::455 => for (uint256 i = 0; i < outputParams.outputTokens.length;) {
202309-maia/src/MulticallRootRouter.sol::557 => for (uint256 i = 0; i < outputTokens.length;) {
202309-maia/src/RootBridgeAgent.sol::318 => for (uint256 i = 0; i < settlement.hTokens.length;) {
202309-maia/src/RootBridgeAgent.sol::392 => // Cache length
202309-maia/src/RootBridgeAgent.sol::393 => uint256 length = _dParams.hTokens.length;
202309-maia/src/RootBridgeAgent.sol::396 => if (length > MAX_TOKENS_LENGTH) revert InvalidInputParams();
202309-maia/src/RootBridgeAgent.sol::399 => for (uint256 i = 0; i < length;) {
202309-maia/src/RootBridgeAgent.sol::871 => if (_hTokens.length == 0) revert SettlementRetryUnavailableUseCallout();
202309-maia/src/RootBridgeAgent.sol::877 => if (_hTokens.length == 1) {
202309-maia/src/RootBridgeAgent.sol::891 => } else if (_hTokens.length > 1) {
202309-maia/src/RootBridgeAgent.sol::896 => uint8(_hTokens.length),
202309-maia/src/RootBridgeAgent.sol::1056 => // Check if valid length
202309-maia/src/RootBridgeAgent.sol::1057 => if (_globalAddresses.length > MAX_TOKENS_LENGTH) revert InvalidInputParamsLength();
202309-maia/src/RootBridgeAgent.sol::1059 => // Check if valid length
202309-maia/src/RootBridgeAgent.sol::1060 => if (_globalAddresses.length != _amounts.length) revert InvalidInputParamsLength();
202309-maia/src/RootBridgeAgent.sol::1061 => if (_amounts.length != _deposits.length) revert InvalidInputParamsLength();
202309-maia/src/RootBridgeAgent.sol::1067 => address[] memory hTokens = new address[](_globalAddresses.length);
202309-maia/src/RootBridgeAgent.sol::1068 => address[] memory tokens = new address[](_globalAddresses.length);
202309-maia/src/RootBridgeAgent.sol::1070 => for (uint256 i = 0; i < hTokens.length;) {
202309-maia/src/RootBridgeAgent.sol::1092 => uint8(hTokens.length),
202309-maia/src/RootBridgeAgent.sol::1210 => if (_srcAddress.length != 40) revert LayerZeroUnauthorizedCaller();
202309-maia/src/RootBridgeAgentExecutor.sol::100 => if (_payload.length > PARAMS_TKN_SET_SIZE) {
202309-maia/src/RootBridgeAgentExecutor.sol::131 => uint256 length = _payload.length;
202309-maia/src/RootBridgeAgentExecutor.sol::134 => if (length > PARAMS_END_OFFSET + (numOfAssets * PARAMS_TKN_SET_SIZE_MULTIPLE)) {
202309-maia/src/RootBridgeAgentExecutor.sol::185 => if (_payload.length > PARAMS_SETTLEMENT_OFFSET) {
202309-maia/src/RootBridgeAgentExecutor.sol::220 => _payload.length
202309-maia/src/RootBridgeAgentExecutor.sol::253 => *     1. First byte is the number of assets to be bridged in. Equals length of all arrays.
202309-maia/src/RootBridgeAgentExecutor.sol::261 => *     5. Each of the 4 token related arrays are of length N and start at the following indexes:
202309-maia/src/VirtualAccount.sol::67 => uint256 length = calls.length;
202309-maia/src/VirtualAccount.sol::68 => returnData = new bytes[](length);
202309-maia/src/VirtualAccount.sol::70 => for (uint256 i = 0; i < length;) {
202309-maia/src/VirtualAccount.sol::87 => uint256 length = calls.length;
202309-maia/src/VirtualAccount.sol::88 => returnData = new bytes[](length);
202309-maia/src/VirtualAccount.sol::90 => for (uint256 i = 0; i < length;) {
202309-maia/src/interfaces/ILayerZeroEndpoint.sol::10 => // @param _destination - the address on destination chain (in bytes). address length/format may vary by chains
202309-maia/src/interfaces/IRootBridgeAgent.sol::280 => *     1. First byte is the number of assets to be bridged in. Equals length of all arrays.
```
## [G-03] Use != 0 instead of > 0 for Unsigned Integer Comparison
## Description
While using != 0 for unsigned integer comparison can provide gas optimizations and code readability benefits, it should be applied judiciously. 

**Benefits**
Using != 0 for unsigned integer comparisons offers several potential benefits:

Gas Optimization: It can reduce gas costs since the EVM (Ethereum Virtual Machine) can optimize the comparison operation more efficiently.
Code Readability: It can make the code more concise and readable, especially when checking if a value is non-zero.

**Bad**
```sol
if (_hTokenAmount > 0)
```
**Good**
```
if (_hTokenAmount != 0)
```
## References
```sol
202309-maia/src/ArbitrumBranchPort.sol::127 => if (_deposit > 0) {
202309-maia/src/ArbitrumBranchPort.sol::132 => if (_amount - _deposit > 0) {
202309-maia/src/BaseBranchRouter.sol::165 => if (_amount - _deposit > 0) {
202309-maia/src/BaseBranchRouter.sol::173 => if (_deposit > 0) {
202309-maia/src/BranchBridgeAgent.sol::906 => if (_amount - _deposit > 0) {
202309-maia/src/BranchBridgeAgent.sol::912 => if (_deposit > 0) {
202309-maia/src/BranchPort.sol::259 => if (_amounts[i] - _deposits[i] > 0) {
202309-maia/src/BranchPort.sol::266 => if (_deposits[i] > 0) {
202309-maia/src/BranchPort.sol::525 => if (_hTokenAmount > 0) {
202309-maia/src/BranchPort.sol::531 => if (_deposit > 0) {
202309-maia/src/RootBridgeAgent.sol::363 => if (_dParams.amount > 0) {
202309-maia/src/RootBridgeAgent.sol::370 => if (_dParams.deposit > 0) {
202309-maia/src/RootBridgeAgent.sol::1146 => if (_underlyingAddress == address(0)) if (_deposit > 0) revert UnrecognizedUnderlyingAddress();
202309-maia/src/RootBridgeAgent.sol::1149 => if (_amount - _deposit > 0) {
202309-maia/src/RootBridgeAgent.sol::1156 => if (_deposit > 0) {
202309-maia/src/RootPort.sol::284 => if (_amount - _deposit > 0) {
202309-maia/src/RootPort.sol::290 => if (_deposit > 0) if (!ERC20hTokenRoot(_hToken).mint(_recipient, _deposit, _srcChainId)) revert UnableToMint();
202309-maia/src/VirtualAccount.sol::152 => return size > 0;
```
## [G-04] Long Revert Strings
### Code Snippet
```solidity
require(_bridgeAgentFactory != address(0), "Bridge Agent Factory cannot be 0 address.");
```

### Analysis
The code snippet checks if a `_bridgeAgentFactory` variable is not equal to the zero address before allowing further execution. This is a common pattern used to prevent certain critical functions or operations from being executed with uninitialized or invalid contract addresses. However, the error message associated with this check, "Bridge Agent Factory cannot be 0 address," is not ideal for security reasons.

### Security Impact
#### Gas Optimization for Long Revert Strings
The impact of using a long error message string like "Bridge Agent Factory cannot be 0 address" can lead to increased gas costs when the condition fails. Solidity reverts with an error message by consuming additional gas, and the longer the error message, the more gas is consumed. In Ethereum and other blockchain networks, gas is a valuable resource, and optimizing gas usage is crucial for efficient smart contract execution.

#### Ethical Example Exploit
Let's consider an ethical example exploit scenario:

**Scenario**: An attacker deploys a malicious smart contract that repeatedly calls the function containing the vulnerable check with a valid `_bridgeAgentFactory` address.

**Exploit Steps**:
1. The attacker deploys a malicious contract and provides it with a legitimate `_bridgeAgentFactory` address.

```solidity
contract MaliciousContract {
    address private _bridgeAgentFactory;
    
    constructor(address factoryAddress) {
        _bridgeAgentFactory = factoryAddress;
    }
    
    function maliciousFunction() public {
        // Repeatedly call the vulnerable function
        while (true) {
            // This consumes gas due to the revert, but the attacker can afford it
            require(_bridgeAgentFactory != address(0), "Bridge Agent Factory cannot be 0 address");
        }
    }
}
```

2. The attacker deploys the `MaliciousContract` and calls the `maliciousFunction()`.

```solidity
MaliciousContract malicious = new MaliciousContract(validFactoryAddress);
malicious.maliciousFunction();
```

**Exploit Impact**:
- The malicious contract consumes a significant amount of gas with each execution of `require`, leading to increased costs for other users of the blockchain.
- It could potentially cause a denial-of-service (DoS) attack by congesting the network and making it expensive for legitimate transactions to execute.

### Proof of Concept (PoC)
```solidity
pragma solidity ^0.8.0;

contract MaliciousContract {
    address private _bridgeAgentFactory;
    
    constructor(address factoryAddress) {
        _bridgeAgentFactory = factoryAddress;
    }
    
    function maliciousFunction() public {
        // Repeatedly call the vulnerable function
        while (true) {
            require(_bridgeAgentFactory != address(0), "Bridge Agent Factory cannot be 0 address");
        }
    }
}

contract TargetContract {
    address public factoryAddress;
    
    constructor() {
        factoryAddress = address(0); // Initialize factoryAddress to the zero address
    }
    
    function setFactoryAddress(address newAddress) public {
        factoryAddress = newAddress;
    }
}
```

In this PoC, we have two contracts: `MaliciousContract` and `TargetContract`. The `MaliciousContract` repeatedly calls the vulnerable function, while the `TargetContract` allows setting the `_bridgeAgentFactory` address. An attacker can deploy the `MaliciousContract` and set the `TargetContract`'s `factoryAddress` to exploit the vulnerability.

### Mitigation
To optimize gas usage and mitigate the potential exploit described above, it is advisable to use shorter and more generic error messages like "Invalid input" or "Operation failed." Additionally, consider implementing access control mechanisms to prevent unauthorized access to critical functions.

```solidity
require(_bridgeAgentFactory != address(0), "Invalid input");
```

Shortening revert strings to fit in 32 bytes will decrease gas costs for deployment and gas costs when the revert condition has been met.
If the contract(s) in scope allow using Solidity >=0.8.4, consider using Custom Errors as they are more gas efficient while allowing developers to describe the error in detail using NatSpec.
**Bad**
```sol
require(_bridgeAgentFactory != address(0), "Bridge Agent Factory cannot be 0 address.");
```
**Good**
```sol
require(_bridgeAgentFactory != address(0), "BAF cannot be 0 address");
```
**References**
```sol
02309-maia/src/ArbitrumBranchBridgeAgent.sol::4 => import {IArbitrumBranchPort as IArbPort} from "./interfaces/IArbitrumBranchPort.sol";
202309-maia/src/ArbitrumBranchBridgeAgent.sol::5 => import {IRootBridgeAgent} from "./interfaces/IRootBridgeAgent.sol";
202309-maia/src/ArbitrumBranchBridgeAgent.sol::6 => import {GasParams, IBranchBridgeAgent} from "./interfaces/IBranchBridgeAgent.sol";
202309-maia/src/ArbitrumBranchPort.sol::7 => import {IArbitrumBranchPort} from "./interfaces/IArbitrumBranchPort.sol";
202309-maia/src/ArbitrumCoreBranchRouter.sol::6 => import {IBranchBridgeAgent as IBridgeAgent, GasParams} from "./interfaces/IBranchBridgeAgent.sol";
202309-maia/src/ArbitrumCoreBranchRouter.sol::7 => import {IBranchBridgeAgentFactory as IBridgeAgentFactory} from "./interfaces/IBranchBridgeAgentFactory.sol";
202309-maia/src/ArbitrumCoreBranchRouter.sol::8 => import {IArbitrumBranchPort as IPort} from "./interfaces/IArbitrumBranchPort.sol";
202309-maia/src/BaseBranchRouter.sol::21 => } from "./interfaces/IBranchBridgeAgent.sol";
202309-maia/src/BranchBridgeAgent.sol::8 => import {BridgeAgentConstants} from "./interfaces/BridgeAgentConstants.sol";
202309-maia/src/BranchBridgeAgent.sol::17 => } from "./interfaces/IBranchBridgeAgent.sol";
202309-maia/src/BranchBridgeAgent.sol::18 => import {ILayerZeroEndpoint} from "./interfaces/ILayerZeroEndpoint.sol";
202309-maia/src/BranchBridgeAgent.sol::125 => require(_rootBridgeAgentAddress != address(0), "Root Bridge Agent Address cannot be the zero address.");
202309-maia/src/BranchBridgeAgent.sol::128 => "Layerzero Endpoint Address cannot be the zero address."
202309-maia/src/BranchBridgeAgent.sol::130 => require(_localRouterAddress != address(0), "Local Router Address cannot be the zero address.");
202309-maia/src/BranchBridgeAgent.sol::131 => require(_localPortAddress != address(0), "Local Port Address cannot be the zero address.");
202309-maia/src/BranchBridgeAgentExecutor.sol::11 => import {BridgeAgentConstants} from "./interfaces/BridgeAgentConstants.sol";
202309-maia/src/BranchBridgeAgentExecutor.sol::12 => import {SettlementParams, SettlementMultipleParams} from "./interfaces/IBranchBridgeAgent.sol";
202309-maia/src/BranchPort.sol::127 => require(_bridgeAgentFactory != address(0), "BridgeAgentFactory is zero address");
202309-maia/src/CoreBranchRouter.sol::7 => import {IBranchBridgeAgent as IBridgeAgent, GasParams} from "./interfaces/IBranchBridgeAgent.sol";
202309-maia/src/CoreBranchRouter.sol::8 => import {IBranchBridgeAgentFactory as IBridgeAgentFactory} from "./interfaces/IBranchBridgeAgentFactory.sol";
202309-maia/src/CoreBranchRouter.sol::10 => import {ICoreBranchRouter} from "./interfaces/ICoreBranchRouter.sol";
202309-maia/src/CoreBranchRouter.sol::11 => import {IERC20hTokenBranchFactory as ITokenFactory} from "./interfaces/IERC20hTokenBranchFactory.sol";
202309-maia/src/CoreRootRouter.sol::8 => import {IERC20hTokenRootFactory as IFactory} from "./interfaces/IERC20hTokenRootFactory.sol";
202309-maia/src/CoreRootRouter.sol::15 => } from "./interfaces/IRootBridgeAgent.sol";
202309-maia/src/MulticallRootRouter.sol::13 => } from "./interfaces/IRootBridgeAgent.sol";
202309-maia/src/RootBridgeAgent.sol::8 => import {ILayerZeroEndpoint} from "./interfaces/ILayerZeroEndpoint.sol";
202309-maia/src/RootBridgeAgent.sol::10 => import {IBranchBridgeAgent} from "./interfaces/IBranchBridgeAgent.sol";
202309-maia/src/RootBridgeAgent.sol::11 => import {IERC20hTokenRoot} from "./interfaces/IERC20hTokenRoot.sol";
202309-maia/src/RootBridgeAgent.sol::13 => import {BridgeAgentConstants} from "./interfaces/BridgeAgentConstants.sol";
202309-maia/src/RootBridgeAgent.sol::23 => } from "./interfaces/IRootBridgeAgent.sol";
202309-maia/src/RootBridgeAgent.sol::111 => require(_lzEndpointAddress != address(0), "Layerzero Enpoint Address cannot be zero address");
202309-maia/src/RootBridgeAgent.sol::112 => require(_localPortAddress != address(0), "Port Address cannot be zero address");
202309-maia/src/RootBridgeAgent.sol::113 => require(_localRouterAddress != address(0), "Router Address cannot be zero address");
202309-maia/src/RootBridgeAgentExecutor.sol::7 => import {IRootBridgeAgent} from "./interfaces/IRootBridgeAgent.sol";
202309-maia/src/RootBridgeAgentExecutor.sol::9 => import {BridgeAgentConstants} from "./interfaces/BridgeAgentConstants.sol";
202309-maia/src/RootPort.sol::8 => import {IERC20hTokenRootFactory} from "./interfaces/IERC20hTokenRootFactory.sol";
202309-maia/src/RootPort.sol::9 => import {IRootBridgeAgent as IBridgeAgent} from "./interfaces/IRootBridgeAgent.sol";
202309-maia/src/RootPort.sol::130 => require(_bridgeAgentFactory != address(0), "Bridge Agent Factory cannot be 0 address.");
202309-maia/src/RootPort.sol::131 => require(_coreRootRouter != address(0), "Core Root Router cannot be 0 address.");
202309-maia/src/RootPort.sol::152 => require(_coreRootBridgeAgent != address(0), "Core Root Bridge Agent cannot be 0 address.");
202309-maia/src/RootPort.sol::153 => require(_coreLocalBranchBridgeAgent != address(0), "Core Local Branch Bridge Agent cannot be 0 address.");
202309-maia/src/RootPort.sol::154 => require(_localBranchPortAddress != address(0), "Local Branch Port Address cannot be 0 address.");
202309-maia/src/VirtualAccount.sol::9 => import {ERC1155Receiver} from "@openzeppelin/contracts/token/ERC1155/utils/ERC1155Receiver.sol";
202309-maia/src/VirtualAccount.sol::10 => import {IERC1155Receiver} from "@openzeppelin/contracts/token/ERC1155/IERC1155Receiver.sol";
202309-maia/src/VirtualAccount.sol::11 => import {IERC721Receiver} from "@openzeppelin/contracts/token/ERC721/IERC721Receiver.sol";
202309-maia/src/factories/ArbitrumBranchBridgeAgentFactory.sol::4 => import {IArbitrumBranchPort as IPort} from "../interfaces/IArbitrumBranchPort.sol";
202309-maia/src/factories/ArbitrumBranchBridgeAgentFactory.sol::57 => require(_coreRootBridgeAgent != address(0), "Core Root Bridge Agent Address cannot be 0");
202309-maia/src/factories/ArbitrumBranchBridgeAgentFactory.sol::85 => msg.sender == localCoreBranchRouterAddress, "Only the Core Branch Router can create a new Bridge Agent."
202309-maia/src/factories/ArbitrumBranchBridgeAgentFactory.sol::89 => "Root Bridge Agent Factory Address does not match."
202309-maia/src/factories/BranchBridgeAgentFactory.sol::9 => import {IBranchBridgeAgentFactory} from "../interfaces/IBranchBridgeAgentFactory.sol";
202309-maia/src/factories/BranchBridgeAgentFactory.sol::61 => require(_rootBridgeAgentFactoryAddress != address(0), "Root Bridge Agent Factory Address cannot be 0");
202309-maia/src/factories/BranchBridgeAgentFactory.sol::64 => "Layerzero Endpoint Address cannot be the zero address."
202309-maia/src/factories/BranchBridgeAgentFactory.sol::66 => require(_localCoreBranchRouterAddress != address(0), "Core Branch Router Address cannot be 0");
202309-maia/src/factories/BranchBridgeAgentFactory.sol::88 => require(_coreRootBridgeAgent != address(0), "Core Root Bridge Agent cannot be 0");
202309-maia/src/factories/BranchBridgeAgentFactory.sol::121 => msg.sender == localCoreBranchRouterAddress, "Only the Core Branch Router can create a new Bridge Agent."
202309-maia/src/factories/BranchBridgeAgentFactory.sol::125 => "Root Bridge Agent Factory Address does not match."
202309-maia/src/factories/ERC20hTokenBranchFactory.sol::8 => import {IERC20hTokenBranchFactory, ERC20hTokenBranch} from "../interfaces/IERC20hTokenBranchFactory.sol";
202309-maia/src/factories/ERC20hTokenRootFactory.sol::8 => import {IERC20hTokenRootFactory, ERC20hTokenRoot} from "../interfaces/IERC20hTokenRootFactory.sol";
202309-maia/src/factories/RootBridgeAgentFactory.sol::4 => import {IRootBridgeAgentFactory} from "../interfaces/IRootBridgeAgentFactory.sol";
202309-maia/src/interfaces/ILayerZeroEndpoint.sol::5 => import "./ILayerZeroUserApplicationConfig.sol";
202309-maia/src/interfaces/IRootPort.sol::5 => import {GasParams} from "../interfaces/IRootBridgeAgent.sol";
202309-maia/src/interfaces/IRootRouter.sol::4 => import {DepositParams, DepositMultipleParams} from "../interfaces/IRootBridgeAgent.sol";
202309-maia/src/interfaces/IVirtualAccount.sol::4 => import {IERC721Receiver} from "@openzeppelin/contracts/token/ERC721/IERC721Receiver.sol";
202309-maia/src/token/ERC20hTokenBranch.sol::8 => import {IERC20hTokenBranch} from "../interfaces/IERC20hTokenBranch.sol";
202309-maia/src/token/ERC20hTokenRoot.sol::8 => import {IERC20hTokenRoot} from "../interfaces/IERC20hTokenRoot.sol";
```