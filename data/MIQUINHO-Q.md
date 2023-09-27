# [QA-01] Missing zero address checks
should check if the addresses are not zero
- Permalink (github)
[ERC20hTokenBranch.sol#L21](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/token/ERC20hTokenBranch.sol#L21)
[VirtualAccount.sol#L36-37](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/VirtualAccount.sol#L36)
[CoreBranchRouter.sol#L31](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/CoreBranchRouter.sol#L31)
# [QA-02] Missing requiresApprovedCaller modifier
 Other functions that change state and send eth have a modifier called 'requiresApprovedCaller' but the function payableCall does not have the modifier 'requiresApprovedCaller'
- Permalink (github)
[VirtualAccount.sol#L85](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/VirtualAccount.sol#L85)

# [QA-03] Unchecked return value of low level 'call' and 'send'
- Permalink (Github)
[RootBridgeAgent.sol#L927](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L927)
[RootBridgeAgent.sol#L835](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L835)
[RootBridgeAgent.sol#L823](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L823)
[RootBridgeAgent.sol#L915](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L915)
[BranchBridgeAgent.sol#L770](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L770)
[BranchBridgeAgent.sol#L785](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L785)

# [QA-04] Parameter isnt used!
- Permalink (Github)
 [ArbitrumCoreBranchRouter.sol#L51](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/ArbitrumCoreBranchRouter.sol#L51)
[MulticallRootRouter.sol#L137](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/MulticallRootRouter.sol#L137)
[MulticallRootRouter.sol#L223](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/MulticallRootRouter.sol#L223)
[MulticallRootRouter.sol#L312](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/MulticallRootRouter.sol#L312)
 [MulticallRootRouter.sol#L401](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/MulticallRootRouter.sol#L401)
[RootBridgeAgent.sol#L423](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L423)
[ArbitrumBranchBridgeAgent.sol#L99](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/ArbitrumBranchBridgeAgent.sol#L99)
