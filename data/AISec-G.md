   # Cache Array Length Outside of Loop

Caching the array length outside a loop saves reading it on each iteration, as long as the array's length is not changed during the loop.

Reading array length at each iteration of the loop takes 6 gas (3 for mload and 3 to place memory_offset) in the stack.

https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol?plain=1#L192
```
        for (uint256 i = 0; i < _hTokens.length;) {
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol?plain=1#L557
```
        for (uint256 i = 0; i < outputTokens.length;) {
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol?plain=1#L1070
```
        for (uint256 i = 0; i < hTokens.length;) {
```

## Recommended Mitigation Steps
```solidity
uint256 len = _hTokens.length
for (uint256 i; i < len;) {
...
}
```
