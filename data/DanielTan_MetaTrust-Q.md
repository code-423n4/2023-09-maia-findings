# Lines of code
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L259-L270

# The `setLocalAddress` function lacks setting or validating the `_globalAddress`

# Description

The `setLocalAddress` function of the `RootPort` contract lacks setting `isGlobalAddress[_globalAddress]` as true or validating whether `isGlobalAddress[_globalAddress]` is true(valid) or not. 

Before executing the statement `getGlobalTokenFromLocal[_localAddress][_srcChainId] = _globalAddress;`, it is needed to make sure the `_globalAddress` is a valid global address.

# Impact
Functions, `bridgeToRoot`, `bridgeToRootFromLocalBranch`, and `bridgeToLocalBranchFromRoot` will check the `isGlobalAddress` first, missing setting the `isGlobalAddress` or any wrong value of `isGlobalAddress` will block other functions.

# Recommendation
Consider validating `isGlobalAddress[_globalAddress]` is true or not or setting the `isGlobalAddress[_globalAddress]` as true before executing the rest statements of the `setLocalAddress` function.