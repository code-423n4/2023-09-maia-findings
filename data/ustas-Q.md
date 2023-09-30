## Low issues

1. Implementation of the ERC20 by Solmate in `ERC20hTokenRoot` and `ERC20hTokenBranch` doesn't support `increaseAllowance()` and `decreaseAllowance()` which can lead to known problems with ERC20. It might be better to use OpenZeppelin's implementation of ERC20.
https://forum.openzeppelin.com/t/explain-the-practical-use-of-increaseallowance-and-decreaseallowance-functions-on-erc20/15103/2
https://github.com/code-423n4/2023-09-maia/blob/ffbe532c6f5224a55ce099b4016bd8806bdbc913/src/token/ERC20hTokenRoot.sol#L6
https://github.com/code-423n4/2023-09-maia/blob/ffbe532c6f5224a55ce099b4016bd8806bdbc913/src/token/ERC20hTokenRoot.sol#L12
https://github.com/code-423n4/2023-09-maia/blob/ffbe532c6f5224a55ce099b4016bd8806bdbc913/src/token/ERC20hTokenBranch.sol#L12

2. There's no `VirtualAccount.withdrawERC1155()`, which might affect UX because the only way to withdraw ERC1155 token is to use raw and dangerous `VirtualAccount.call()`. Other standards (ERC20 and ERC721) have their respective `.withdraw` functions.
https://github.com/code-423n4/2023-09-maia/blob/ffbe532c6f5224a55ce099b4016bd8806bdbc913/src/VirtualAccount.sol#L55-L63

1. There's no need to implement the `getFeeEstimate()` in `ArbitrumBranchBridgeAgent` since it doesn't use LayerZero. The function might be misleading to the users.
https://github.com/code-423n4/2023-09-maia/blob/ffbe532c6f5224a55ce099b4016bd8806bdbc913/src/BranchBridgeAgent.sol#L161-L173

1. There's a duplicate of `BranchPort.setCoreBranchRouter()` in the form of `BranchPort.setCoreRouter()`. Both functions change the `coreBranchRouterAddress` state variable, but the second one isn't used anywhere in the system and is gated (so cannot be called as external function).
https://github.com/code-423n4/2023-09-maia/blob/ffbe532c6f5224a55ce099b4016bd8806bdbc913/src/BranchPort.sol#L331-L335
https://github.com/code-423n4/2023-09-maia/blob/ffbe532c6f5224a55ce099b4016bd8806bdbc913/src/BranchPort.sol#L414-L424

1. Unnecessary concatenation and casting to string `string(string.concat(_name))` and `string(string.concat(_symbol))`. This can be replaced with just `_name` and `_symbol`.
https://github.com/code-423n4/2023-09-maia/blob/ffbe532c6f5224a55ce099b4016bd8806bdbc913/src/token/ERC20hTokenRoot.sol#L38

1. Comments and names of the mappings tell that the first argument is `chainId`, but in fact, it is the second argument. 
https://github.com/code-423n4/2023-09-maia/blob/ffbe532c6f5224a55ce099b4016bd8806bdbc913/src/RootPort.sol#L89-L101
Examples of use:
https://github.com/code-423n4/2023-09-maia/blob/ffbe532c6f5224a55ce099b4016bd8806bdbc913/src/RootPort.sol#L195
https://github.com/code-423n4/2023-09-maia/blob/ffbe532c6f5224a55ce099b4016bd8806bdbc913/src/RootPort.sol#L196
https://github.com/code-423n4/2023-09-maia/blob/ffbe532c6f5224a55ce099b4016bd8806bdbc913/src/RootPort.sol#L231