### Possible wrong check
In [CoreRootRouter](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/CoreRootRouter.sol#L163), the `_rootBridgeAgentFactory` is not used, yet it is checked for being a valid bridge agent factory, while the one that is ultimately being interacted with is `_branchBridgeAgentFactory`.

### Unnecessary string concatenation in [ERC20hTokenRoot](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/token/ERC20hTokenRoot.sol#L38)

### Duplicate check in `BranchBridgeAgent@_createDepositMultiple`
Checks in [BranchBridgeAgent](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L869-L872) are also done in [BranchPort](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L299-L302)

### `ArbitrumCoreBranchRouter@addLocalToken`, `ArbitrumBranchBridgeAgent@depositToPort` and `ArbitrumBranchBridgeAgent@withdrawFromPort` are payable

### Unnecessary cast to payable
`_refundee` is already payable in [BranchBridgeAgent](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L774) and [RootBridgeAgent](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L944)

### Wrong comments
Comments are not accurate in [CoreBranchRouter](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/CoreBranchRouter.sol#L188) and [ArbitrumCoreBranchRouter](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/ArbitrumCoreBranchRouter.sol#L73)

### Potentially unneeded checks on unused parameters
In [BranchBridgeAgentFactory](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/factories/BranchBridgeAgentFactory.sol#L123C1-L126) and [RootPort](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L544)

### Unnecessary cast to uint8
`numOfAssets` is already uint8 in [RootBridgeAgentExecutor](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgentExecutor.sol#L281)
