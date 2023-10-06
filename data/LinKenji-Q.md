## Impact

`isChainId` mapping is defined as `mapping(uint256 chainId => bool isActive)` public `isChainId;`, but it should be `mapping(uint256 => bool) public isChainId;`. This could cause issues when trying to access the `isChainId` mapping with a specific chain ID.

[61](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L61)

```solidity
mapping(uint256 chainId => bool isActive) public isChainId;
```

It should be corrected to:

```solidity
mapping(uint256 => bool) public isChainId;
```

The corrected version ensures that the `isChainId` mapping is correctly defined to accept a `uint256` as a key and return a `bool` indicating whether the chain is active. The typo in the original code may cause issues when trying to access the `isChainId` mapping with a specific chain ID, potentially leading to unexpected behavior in the contract's functionality.

## Proof of Concept

```solidity
// Incorrect mapping declaration
mapping(uint256 chainId => bool isActive) public isChainId;

// Function to set a chain as active
function setChainActive(uint256 _chainId) external onlyOwner {
    isChainId[_chainId] = true;
}

// Function to check if a chain is active
function checkChainActive(uint256 _chainId) external view returns (bool) {
    return isChainId[_chainId];
}
```

In this hypothetical scenario, we have an incorrectly declared mapping `isChainId` that was intended to keep track of active chains. The function `setChainActive` is supposed to mark a chain as active by setting `true` in the mapping. The function `checkChainActive` is meant to check if a particular chain is active.

However, due to the incorrect mapping declaration, this code will not compile and will result in a syntax error. Developers will encounter an issue when trying to compile this contract because `chainId => bool isActive` is not the correct syntax for defining a mapping in Solidity.

