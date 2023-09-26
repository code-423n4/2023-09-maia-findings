# Incorrect NatSpec and named mapping parameters

In [**RootPort.sol**](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L89-L101) there are four mappings mapping from one type of token to another based on a chainId. 

```jsx
    /// @notice ChainId -> Local Address -> Global Address
    mapping(address chainId => mapping(uint256 localAddress => address globalAddress)) public getGlobalTokenFromLocal;

    /// @notice ChainId -> Global Address -> Local Address
    mapping(address chainId => mapping(uint256 globalAddress => address localAddress)) public getLocalTokenFromGlobal;

    /// @notice ChainId -> Underlying Address -> Local Address
    mapping(address chainId => mapping(uint256 underlyingAddress => address localAddress)) public
        getLocalTokenFromUnderlying;

    /// @notice Mapping from Local Address to Underlying Address.
    mapping(address chainId => mapping(uint256 localAddress => address underlyingAddress)) public
        getUnderlyingTokenFromLocal;
```

 Both the NatSpec and the named parameters are wrong since the map path is not **chainId => token1 => token2 **, but **token1 => chainId => token2**. A chainId is uint256, not an address. Also, everywhere in the code, the **token1 => chainId => token2** is used.
