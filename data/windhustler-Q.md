## Users are able to create hTokens that don't belong to any underlying asset.

If we look at the [addLocalToken](https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L62) function, it always deploys a new hToken contract through the hTokenFactory. This holds true even if hToken was already added for the underlying address. 
Consider adding a call to the `RootChain` to check if `hToken` for a specific `underlyingAddress` was already added. 