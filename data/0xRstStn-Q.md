# It is not checked that different arrays have the same length.

### Description
In contract ``BaseBranchRouter.sol``, in function ``callOutAndBridgeMultiple``, the argument ``_dParams`` is of the class ``DepositMultipleInput`` which is a structure that consists of several arrays. Function ``callOutAndBridgeMultiple`` invokes ``_transferAndApproveMultipleTokens`` which contains a for loop that takes ``_hTokens.length`` as the number of iterations of the loop:
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BaseBranchRouter.sol#L186-L199

### Impact
If the other arrays have different lengths, the function would revert.

### Recommended Mitigation
Check that the lengths of the different arrays match as done in other functions as in ``_createDepositMultiple`` in contract ``BranchBridgeAgent.sol`` 
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L860

