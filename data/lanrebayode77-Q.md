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