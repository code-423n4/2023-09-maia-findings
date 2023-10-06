In contract ``RootBridgeAgent.sol``, under function ``_createSettlementMultiple``, there is a for loop which reads the ``hTokens.length`` for each iteration:

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L1070
`` for (uint256 i = 0; i < hTokens.length;) ``

Caching the length of the array saves around 3 gas per iteration. It is recommended to cache the length and use it in the for loop:
``` 
    uint256 length = hTokens.length;
    for (uint256 i; i < length)
```
