Unproper mapping define:

https://github.com/code-423n4/2023-09-maia/blob/746905cdd3aa165ba1f9274c360dfd9f0c52e781/src/RootPort.sol#L89-L90

This is how mapping:
```solidity 
 mapping(address chainId => mapping(uint256 localAddress => address globalAddress)) public getGlobalTokenFromLocal;
```
but how it autally use is
```solidity
 getGlobalTokenFromLocal[_localAddress][_srcChainId] = _globalAddress
```
So the correct define should be:

```solidity 
mapping(address localAddress => mapping(uint256 chainId => address globalAddress))   public getGlobalTokenFromLocal;
```
