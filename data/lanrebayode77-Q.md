## Low Findings 
### 1. Incorrect Spelling
https://github.com/code-423n4/2023-09-maia/blob/c0dc3550e0754571b82d7bfd8f0282ac8fa5e42f/src/BranchBridgeAgent.sol#L69
```
    /// @notice Address for Local Router used for custom actions for different hApps.
```
hApps instead of dApps
```
    /// @notice Address for Local Router used for custom actions for different dApps.
```

### 2. Natspec error
https://github.com/code-423n4/2023-09-maia/blob/c0dc3550e0754571b82d7bfd8f0282ac8fa5e42f/src/interfaces/IBranchBridgeAgent.sol#L70
```
 *  | callOutMultiple0x2            |  1b(n) + 20b(recipient) + 4b     |   	     32b + 32b + 32b + 32b      |   ---	   |
```
```callOutMultiple0x2```   instead of ```callOutMultiple = 0x2```  

### 3. Array Mismatch
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L248-L254
No check for equal array length, could lead to array mismatch
``` solidity
 function bridgeInMultiple(
        address _recipient,
        address[] memory _localAddresses,
        address[] memory _underlyingAddresses,
        uint256[] memory _amounts,
        uint256[] memory _deposits
    ) external override requiresBridgeAgent {
        // Cache Length
        uint256 length = _localAddresses.length;

        // Loop through token inputs
        for (uint256 i = 0; i < length;) {
```
check that all arrays length are equally in length
``` solidity
require(length == _underlyingAddresses.length, "mis-match");
require(length == _amounts.length, "mis-match");
require(length == _deposits.length, "mis-match");
```

### 3. No Initialize modifier in initializtion function
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/CoreRootRouter.sol#L83
Consider adding initialize modifier to ensure the function is not called again after setting it once.

### 4. Incorrect natspec comment
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L60-L61
The code indicates a mapping from a uint256 to a bool, to show if a particular chainId is active or not, but the comment states a difference description which is mis-leading.
``` solidity
 /// @notice Mapping from address to Bridge Agent.
    mapping(uint256 chainId => bool isActive) public isChainId;
```
Consider changing to the correct comment. 
``` solidity
 /// @notice Mapping from chainId(uint256) to bool(indicating activeness).
    mapping(uint256 chainId => bool isActive) public isChainId;
```
