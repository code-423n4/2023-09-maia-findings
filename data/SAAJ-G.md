 Gas optimisation
https://www.rareskills.io/post/gas-optimization

# Gas Optimizations Report
https://github.com/code-423n4/2023-08-livepeer/blob/main/bot-report.md#g-22-use-calldata-instead-of-memory-for-immutable-arguments
https://solodit.xyz/issues/g-08-checking-non-zero-value-can-avoid-an-external-call-to-save-gas-code4rena-connext-connext-git
https://github.com/code-423n4/2023-08-livepeer-findings/blob/main/data/Sathish9098-G.md

## Summary
|	|	Issue								| Instances	|
| ---	|	-----								|	-----	|
|[G-01]| Storage over memory	|	02	|
|[G 02]|abi.encode() is less efficient than abi.encodePacked()				|	12	|
|[G-03]|It's cheaper to declare the variable outside the loop	|	04	|

Link to the Code:

# [G-01] Storage over memory (02 Instances)
Some functions are using memory to read state variables while using storage is more gas efficient.
When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read.

```diff
File: 2023-09-maia/src/RootBridgeAgentExecutor.sol

- 88:	DepositParams memory dParams = DepositParams({
+ 88:	DepositParams storage dParams = DepositParams({

- 173:	DepositParams memory dParams = DepositParams({
+ 173:	DepositParams storage dParams = DepositParams({
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol


# [G 02] abi.encode() is less efficient than abi.encodePacked()(12 Instances)
Refer to this [link]( https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison).

```diff
File: 2023-09-maia/src/CoreRootRouter.sol

- 126:        bytes memory params = abi.encode(
+ 126:        bytes memory params = abi.encodePacked(

- 168:        bytes memory params = abi.encode(_branchBridgeAgentFactory);
+ 168:        bytes memory params = abi.encodePacked(_branchBridgeAgentFactory);

- 193:        bytes memory params = abi.encode(_branchBridgeAgent);
+ 193:        bytes memory params = abi.encodePacked(_branchBridgeAgent);

- 220:        bytes memory params = abi.encode(_underlyingToken, _minimumReservesRatio);
+ 220:        bytes memory params = abi.encodePacked(_underlyingToken, _minimumReservesRatio);

- 251:        bytes memory params = abi.encode(_portStrategy, _underlyingToken, _dailyManagementLimit, _isUpdateDailyLimit);
+ 251:        bytes memory params = abi.encodePacked(_portStrategy, _underlyingToken, _dailyManagementLimit, _isUpdateDailyLimit);

- 281:        bytes memory params = abi.encode(_coreBranchRouter, _coreBranchBridgeAgent);
+ 281:        bytes memory params = abi.encodePacked(_coreBranchRouter, _coreBranchBridgeAgent);

- 424:        bytes memory params = abi.encode(
+ 424:        bytes memory params = abi.encodePacked(

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L173


```diff
File: 2023-09-maia/src/ CoreBranchRouter.sol

- 48:        bytes memory params = abi.encode(msg.sender, _globalAddress, _dstChainId, [_gParams[1], _gParams[2]]);
+ 48:        bytes memory params = abi.encodePacked(msg.sender, _globalAddress, _dstChainId, [_gParams[1], _gParams[2]]);

- 72:        bytes memory params = abi.encode(_underlyingAddress, newToken, newToken.name(), newToken.symbol(), decimals);
+ 72:        bytes memory params = abi.encodePacked(_underlyingAddress, newToken, newToken.name(), newToken.symbol(), decimals);

- 178:        bytes memory params = abi.encode(_globalAddress, newToken);
+ 178:        bytes memory params = abi.encodePacked(_globalAddress, newToken);

- 226:        bytes memory params = abi.encode(newBridgeAgent, _rootBridgeAgent);
+ 226:        bytes memory params = abi.encodePacked(newBridgeAgent, _rootBridgeAgent);
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol



```diff
File: 2023-09-maia/src/ BranchBridgeAgent.sol

- 471:        bytes memory params = abi.encode(_settlementNonce, msg.sender, _params, _gParams[1]);
+ 471:        bytes memory params = abi. abi.encodePacked(_settlementNonce, msg.sender, _params, _gParams[1]);

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol




# [G-03] It's cheaper to declare the variable outside the loop (04 Instances)
Declaring a variable inside a loop result in variable being redeclared during each loop iteration which consume higher gas.
The variable gets reallocated when declared outside loop making it more gas efficient.

```diff
File: 2023-09-maia/src/RootBridgeAgent.sol

+ uint256 currentIterationOffset;
 281:    for (uint256 i = 0; i < uint256(uint8(numOfAssets));) {
            // Cache offset
- 283:    uint256 currentIterationOffset = PARAMS_START + i;

        // Clear Global hTokens To Recipient on Root Chain cancelling Settlement to Branch
+	 address _hToken;
318:    for (uint256 i = 0; i < settlement.hTokens.length;) {
            // Save to memory
- 320:       address _hToken = settlement.hTokens[i];


+		uint16 destChainId;
 1070:       for (uint256 i = 0; i < hTokens.length;) {
            // Populate Addresses for Settlement
            hTokens[i] = IPort(localPortAddress).getLocalTokenFromGlobal(_globalAddresses[i], _dstChainId);
            tokens[i] = IPort(localPortAddress).getUnderlyingTokenFromLocal(hTokens[i], _dstChainId);
            // Avoid stack too deep
 - 1076:           uint16 destChainId = _dstChainId;
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol


```diff
File: 2023-09-maia/src/BranchBridgeAgent.sol

+	uint256 currentIterationOffset;
 512:       for (uint256 i = 0; i < numOfAssets;) {
            // Cache common offset
 - 515:        uint256 currentIterationOffset = PARAMS_START + i;
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol


