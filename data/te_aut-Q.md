SPELLING ERROR: SLoC - #19

CONTRACT NAME: ArbitrumCoreBranchRouter.sol
Contract link: https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumCoreBranchRouter.sol
LINE OF CODE:   @dev    The function `addGlobalToken` is is not available since it's used
Recommendation: remove one of the two ‘is’ from the comment on SLoC #19

TOOLS USED: Manual review

POC: The ArbitrumCoreBranchRouter.sol contract has a comment of the contract ArbitrumCoreBranchRouter on SLoC #19. The comment’s intention is used to explain what the contract does, however, the word repetition error can be confusing. If it’s an error, remove one of the two “is” otherwise, update the comment.
