## Summary

|                | Issues                                                        | Instances |
| -------------- | ------------------------------------------------------------- | --------- |
| [[G-0](#gas0)] | Unnecessary initialization for loop variables                 | 15        |
| [[G-1](#gas1)] | Unnecessary concatenation of strings in `ERC20hTokenRoot.sol` | 1         |

## Gas

### [G-0] Unnecessary initialization for loop variables <a id="gas0"></a>

No need to initialize the loop variable `i` in the beginning. Default value for uint256 is 0

_Instances [total 15]:_

```Javascript
File: src/BaseBranchRouter.sol

192:         for (uint256 i = 0; i < _hTokens.length;) {
```

GitHub: [192](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BaseBranchRouter.sol#L192)

```Javascript
File: src/BranchBridgeAgent.sol

447:         for (uint256 i = 0; i < deposit.tokens.length;) {

513:         for (uint256 i = 0; i < numOfAssets;) {
```

GitHub: [447](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L447), [513](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L513)

```Javascript
File: src/BranchPort.sol

257:         for (uint256 i = 0; i < length;) {

305:         for (uint256 i = 0; i < length;) {
```

GitHub: [257](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L257), [305](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L305)

```Javascript
File: src/MulticallRootRouter.sol

278:             for (uint256 i = 0; i < outputParams.outputTokens.length;) {

367:             for (uint256 i = 0; i < outputParams.outputTokens.length;) {

455:             for (uint256 i = 0; i < outputParams.outputTokens.length;) {

557:         for (uint256 i = 0; i < outputTokens.length;) {

```

GitHub: [278](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/MulticallRootRouter.sol#L278), [367](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/MulticallRootRouter.sol#L367), [455](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/MulticallRootRouter.sol#L455), [557](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/MulticallRootRouter.sol#L557)

```Javascript
File: src/RootBridgeAgent.sol

318:         for (uint256 i = 0; i < settlement.hTokens.length;) {

399:         for (uint256 i = 0; i < length;) {

1070:        for (uint256 i = 0; i < hTokens.length;) {
```

GitHub: [318](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L318), [399](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L399), [1070](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L1070)

```Javascript
File: src/RootBridgeAgentExecutor.sol

281:         for (uint256 i = 0; i < uint256(uint8(numOfAssets));) {
```

GitHub: [281](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgentExecutor.sol#L281)

```Javascript
File: src/VirtualAccount.sol

70:          for (uint256 i = 0; i < length;) {

90:          for (uint256 i = 0; i < length;) {
```

GitHub: [70](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/VirtualAccount.sol#L70), [90](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/VirtualAccount.sol#L90)

### [G-1] Unnecessary concatenation of strings in `ERC20hTokenRoot.sol` <a id="gas1"></a>

There is no need to use `string.concate()` since there is only one argument passed.

_Instance:_

```Javascript
File: src/token/ERC20hTokenRoot.sol

38    ) ERC20(string(string.concat(_name)), string(string.concat(_symbol)), _decimals) {
```

GitHub: [38](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/token/ERC20hTokenRoot.sol#L38)