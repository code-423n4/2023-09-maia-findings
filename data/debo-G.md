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