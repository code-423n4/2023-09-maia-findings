# Missing Re-entrancy  protection. 

# Vulnerability details

There are re-entrancy guards missing in VirtualAccount.sol function: withdrawNative, withdrawERC20 and withdrawERC721    

# Lines of code

VirtualAccount.sol#L51  
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/VirtualAccount.sol#L51C7-L51C7

VirtualAccount.sol#L56
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/VirtualAccount.sol#L56

VirtualAccount.sol#L61
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/VirtualAccount.sol#L61

# Recommended Mitigation Steps

To protect against cross-function reentrancy attacks, it may be necessary to use a OpenZeppelin ReentrancyGuard which provides a modifier to any function called nonReentrant that guards the function against reentrancy attacks.




