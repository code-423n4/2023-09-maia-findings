### Report 1:
Absence of comma "," in comment description causes misinterpretation of code from IRootBridgeAgent.sol. As corrected in the code snippet below 
"has not been executed yet, triggering" is different from "has not been executed, yet triggering", 
not having a comma makes it even more complicated as this comment description is relevant to this code description:  https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L711-L718 .
The comment Description: https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/IRootBridgeAgent.sol#L54 .
```solidity
54. --- *       0x08 | Call to `retrieveDeposit()´. (clears a deposit that has not been executed yet triggering `_fallback`)
54. +++ *       0x08 | Call to `retrieveDeposit()´. (clears a deposit that has not been executed yet, triggering `_fallback`)
```
###  Report 2:
Maia Protocol should consider adding a return value for burn function call just as it is done for the mint function to help in validation to avoid unecessary failure of implemenation which could affect proper functionality of protocol
https://github.com/code-423n4/2023-09-maia/blob/main/src/token/ERC20hTokenRoot.sol#L69-L73
```solidity
 --- function burn(address from, uint256 amount, uint256 chainId) external onlyOwner {
 +++ function burn(address from, uint256 amount, uint256 chainId) external onlyOwner returns (bool) {
        getTokenBalance[chainId] -= amount;
        _burn(from, amount);
+++ return true;
    }
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/token/ERC20hTokenBranch.sol#L35-L37
```solidity
---  function burn(uint256 amount) public override onlyOwner {
+++  function burn(uint256 amount) public override onlyOwner returns (bool) {
        _burn(msg.sender, amount);
+++ return true;
    }
```
This bool return value should be used to validate the burn function where it is being called as listed below
[L527](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L527) of BranchPort.sol, [L321](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L321) of RootPort.sol, [L332](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L332) of RootPort.sol & [L1161](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1161) of RootBridgeAgent.sol.

###  Report 3:
Missing Zero Check for _amount in the mintToLocalBranch(...) function, a validation is necessary as the major functionality of the function is to mint which is not possible if amount is zero, so a reversion is necessary if amount is zero to avoid unnecessary malicious interaction.
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L342
```solidity
   function mintToLocalBranch(address _to, address _hToken, uint256 _amount)
        external
        override
        requiresLocalBranchPort
    {
+++    require(_amount > 0 , "Invalid Amount" )
        if (!isGlobalAddress[_hToken]) revert UnrecognizedToken();
        if (!ERC20hTokenRoot(_hToken).mint(_to, _amount, localChainId)) revert UnableToMint();
    }
```
###  Report 4:
A better way to write the two conditions from the code snippet below is to use the "||" operator, it makes it more cleaner, efficient and readable
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1060-L1061
```solidity
---        if (_globalAddresses.length != _amounts.length) revert InvalidInputParamsLength();
---        if (_amounts.length != _deposits.length) revert InvalidInputParamsLength();
+++        if (_globalAddresses.length != _amounts.length || _amounts.length != _deposits.length) revert InvalidInputParamsLength();
```
