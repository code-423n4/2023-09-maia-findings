## Unnecessary string concatenation in `ERC20hTokenRoot`
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/token/ERC20hTokenRoot.sol#L38

## Duplicate check in `BranchBridgeAgent@_createDepositMultiple`
Checks in https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L869-L872 are also done in https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L299-L302
