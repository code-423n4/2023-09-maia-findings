## [01] `setCoreRouter` in Branchport.sol can be removed

`setCoreRouter` in Branchport.sol appears to be dead code. Although it requires the CoreRouter to call it using the modifier `requiresCoreRouter` there is no reference to this in any of the router contracts. The functionality seems to have been migrated to `setCoreBranchRouter` in Branchport.sol.

```
File: src/BranchPort.sol

331      function setCoreRouter(address _newCoreRouter) external override requiresCoreRouter {
332          require(coreBranchRouterAddress != address(0), "CoreRouter address is zero");
333          require(_newCoreRouter != address(0), "New CoreRouter address is zero");
334          coreBranchRouterAddress = _newCoreRouter;
335:     }
```
## Recommendation
The `setCoreRouter` function can be removed from Branchport.sol and the function can also be removed from the interface IBranchPort.sol.

## [02] Arrays are pushed to but are never referenced
[`strategyTokens`](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L367) and [`bridgeAgents`](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L421) in [BranchPort](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol) are appended to arrays that are never used.

## Recommendation
Remove these arrays as storage variables and remove the `.push` functions so they are not appended to. 

## [03] `manageStrategyToken` doesn't check value is within range

[`manageStrategyToken`](https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L212) allows the `_minimumReservesRatio` to be set for new strategy tokens that are added to a Branch Port. This is passed to a branch and eventually executes [`addStrategyToken()`](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L362) where the `_minimumReservesRatio` is required to be <= the [`DIVISIONER`](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L97) (1e4) and >= [`MIN_RESERVE_RATIO`](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L98C51-L98C54) (3e3) otherwise the function will revert. 

## Recommendation
This check could be performed at the source [`manageStrategyToken`](https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L212) avoiding a cross chain call to set a value that may end up reverting, costing gas and then needing to be called again.