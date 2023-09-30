## QA Report


Q1 - Call custom error function in for loop without argument, which clarify what exact call has fail.
When an error occurs, it is difficult for the user to understand which call caused the error because the error does not contain the call sequence number.
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/VirtualAccount.sol#L90C1-L104C1

Q2 - Same error for different conditions. 
It is difficult for the user to understand why the error occurred. It is recommended to use different custom errors function for each conditional. 
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/ArbitrumBranchBridgeAgent.sol#L126C1-L127C9
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L869C1-L873C1
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L299C1-L302
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/CoreRootRouter.sol#L461-L463
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/CoreRootRouter.sol#L117C1-L123
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L1205C1-L1213
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L307C1-L308
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L357-L372
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L1057-L1061
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L1205-L1213
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L484-L494
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L528-L529
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L544-L545

Q3 - Function do not allow send bytes to EOA. 
The user may want to send bytes to the EOA address(text message), but the function only allows it to be sent if the address is a contract. So, variable `success` always will have default value (false) and it cause revert.
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/VirtualAccount.sol#L74

Q4 - User can not specify other address for withdrawing from port, if his main address in token's blacklist.
if user's address(which deposited tokens to port) was included in blacklist, user will not withdraw tokens to own address, because tx will revert. User should have opportunity specify address for receiving tokens. 
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/ArbitrumBranchBridgeAgent.sol#L79


Q5 - Token contracts are not protected for the ERC20 approval race condition.
A known race condition exists within the present implementation of the ERC20 standard. 
Example scenario :
1.Alice calls approve(Bob, 1000), allocating 1000 tokens for Bob to spend
2.Alice opts to change the amount approved for Bob to spend to a lesser amount via approve(Bob, 500). 3. Once mined, this decreases the number of tokens that Bob can spend to 500.
3.Bob sees the transaction and calls transferFrom(Alice, X, 1000) before approve(Bob, 500) has been mined.
4.If Bob’s transaction is mined prior to Alice’s, 1000 tokens will be transferred by Bob.
However, once Alice’s transaction is mined, Bob can call transferFrom(Alice, X, 500), transferring a total of 1500 tokens despite Alice attempting to limit the total token allowance to 500.
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/token/ERC20hTokenBranch.sol#L12
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/token/ERC20hTokenRoot.sol#L12
