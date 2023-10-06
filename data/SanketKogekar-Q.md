
- Incorrect require message for `_rootChainId == _localChainId` when it actually should be:

```solidity
require(
    _rootChainId == _localChainId,
    "_rootChainId and _localChainId cannot be same."
);
```

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L126-L129

- The encoding misses to include `_dstChainId`

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/CoreRootRouter.sol#L424-L431

- The `_decode` function makes no difference and is unnecessary.

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/MulticallRootRouter.sol#L581-L583

- Code must revert if `outputTokens.length != amountsOut.length`

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/MulticallRootRouter.sol#L547-L549

- There is no pause mechanism for the protocol in case Layer0 has an issue.

- Unchecked `.call()` value can lead to loss of funds / silent failures.

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L833-L838

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L925-L930

- Received eth will be stuck and won't be able to withdraw.

`receive() external payable {}` exists is several contracts.

- `setLocalAddress` must be renamed to `setLocalAndGlobalAddress` and requires to add the check:

`if (_globalAddress == address(0)) revert InvalidGlobalAddress();`

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L259-L270

- Missing address(0) and 0 validation for `to` and `amount`

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/token/ERC20hTokenRoot.sol#L57-L61

- Missing address(0) validation for `from` and `amount` and `burn()` does not return true similar to how `mint()` does.

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/token/ERC20hTokenRoot.sol#L69-L72