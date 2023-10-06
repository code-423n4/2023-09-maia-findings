1. SPELLING ERROR: SLoC - #19

CONTRACT NAME: ArbitrumCoreBranchRouter.sol
Contract link: https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumCoreBranchRouter.sol
LINE OF CODE:   @dev    The function `addGlobalToken` is is not available since it's used
Recommendation: remove one of the two ‘is’ from the comment on SLoC #19

TOOLS USED: Manual review

POC: The ArbitrumCoreBranchRouter.sol contract has a comment of the contract ArbitrumCoreBranchRouter on SLoC #19. The comment’s intention is used to explain what the contract does, however, the word repetition error can be confusing. If it’s an error, remove one of the two “is” otherwise, update the comment.


2. SPELLING ERROR: SLoC - #74 AND #80

CONTRACT NAME: ArbitrumBranchBridgeAgent.sol
Contract link: https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchBridgeAgent.sol
LINE OF CODE:  SLoC #74 * @notice Function to withdraw a single asset to the local Port.
SLoC #80 IArbPort(localPortAddress).withdrawFromPort(msg.sender, msg.sender, localAddress, amount);

Recommendation: The ‘withdraw a single asset to’ can  be changed to ‘withdraw a single asset from’ to match the function ‘withrdrawFromPort’.

TOOLS USED: Manual review

POC: The ArbitrumBranchBridgeAgent.sol contract has a comment of the function withdrawFromPort on SLoC #74. The comment’s intention is used to explain what the function withdrawFromPort does, however, the sentencing of  the comment ‘withdraw a single asset to’  can be confusing. If it’s an error, change the word ‘to’ to ‘from’ to make sense of the function  withdrawFromPort on SLoc #80 otherwise, update the comment.