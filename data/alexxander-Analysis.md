Since this is a second audit round of Ulysses Omnichain System, this report will comment on the newly refactored code.

## Positives
1. Incorporating LayerZero as the message transport layer has greatly reduced the complexity of the code & has improved the user flows to be more straightforward. 
2. The manual gas handling state that was present in the previous implementation is absent and this greatly reduces the surface for potential vulnerabilities that can come from tracking what gas is owed.
3. Previously disclosed vulnerabilities have been acknowledged & the mitigations are done in such a way that not only remove vulnerabilities but also simplify the code.

## Areas of Concern
1. Some of the RootBridgeAgentExecutor & MulticallRootRouter functionalities could be extracted into a overall less number of functions.
2. There are currently approvals of tokens that are not fully utilized in the MulticallRootRouter "safeApprove". During the audit no vulnerabilites were discovered, however, it's good for the developers to keep in mind that after execution from the MultiCallRootRouter there could still be left approval of tokens towards the RootBridgeAgent.

## Time spent
25 hours





### Time spent:
25 hours