A. Any comments for the judge to contextualize your findings
   I have focused on to ensure 1) the state transition of each component is properly performed; 2) funds have been transferred properly from one component to another without messing up. 
   

B. Approach taken in evaluating the codebase
   1) Understand the interaction among the triple (RootPort, RootBridgeAgent, and coreRootRouter) and the triple (BranchPort, BranchBridgeAgent, and CoreBranchRouter).
   2) Understand the interaction between RootBridgeAgent and BranchBridgeAgent via Layerzero messaging as follows: 

RootBridgeAgent callout:
0x00: callout
0x01/0x81: callOutAndBridge(createSettlement)
0x02/0x82: calloutAndBridgeMultiple(_createSettlementMultiple)
0x03: retrieveSettlement
0x04: _performFallbackCall


RootBridgeAgeent.lzReceiveNonBlocking will route call to RootBridgeAgentExecutor or local calls:
0x00: executeSystemRequest
0x01: executeNoDeposit
0x02: executeWithDeposit
0x03: executeWithDepositMultiple
0x04: executeSignedNoDeposit
0x05/0x85: executeSignedWithDeposit
0x06/0x86: executeSignedWithDepositMultiple 
0x07/0x87: _performRetrySettlementCall
0x08: _performFallbackCall
0x09: set  getSettlement[nonce].status = STATUS_FAILED

----------------------------------------------------------------------------------------------------

BranchBridgeAgent callout: 
0x00: calloutSystem
0x01: callOut
0x02: callOutAndBridge
0x03: callOutAndBridgeMultiple
0x04: calloutSigned
0x05/0x85: callOutSignedAndBridge
0x06/0x86: callOutSignedAndBridgeMultiple
0x07/0x87: retrySettlement
0x08: retrieveDeposit
0x09: _performFallbackCall

BranchBridgeAgent. lzReceiveNonBlocking will route calls to BranchBridgeAgentExecutor or local calls: 
0x00: executeNoSettlement
0x01/0x81: executeWithSettlement
0x02/0x82: executeWithSettlementMultiple
0x03: _performFallbackCall
0x04: Fallback change getDeposit[nonce].status = STATUS_FAILED;

C. Architecture recommendations
   1) The complexity is greatly increased by ``RootBridgeAgentExecutor``  and ``BranchBridgeAgentExecutor``.   As a result, many interactions between ``RootBridgeAgent`` and other components have to go through ``RootBridgeAgentExecutor``. The same is true for ``BranchBridgeAgentExecutor``. 

   Mitigation: combine ``RootBridgeAgentExecutor`` into ``RootBridgeAgent``; combine ``BranchBridgeAgentExecutor`` into ``BranchBridgeAgent``. 

   2) The communication between between CoreRootRouter and RootBridgeAgent can be direct instead of via RootPort. For example, consider the flow CoreRootRouter._syncBranchBridgeAgent() -> rootPortAddress.syncBranchBridgeAgentWithRoot() -> _rootBridgeAgent.syncBranchBridgeAgent(_newBranchBridgeAgent, _branchChainId). The complexity can be reduced if CoreRootRouter._syncBranchBridgeAgent() calls _rootBridgeAgent.syncBranchBridgeAgent(_newBranchBridgeAgent, _branchChainId) directly. 

D. Codebase quality analysis
    
E. Centralization risks
    
F. Mechanism review
   1) ``CoreRootRouter`` should focus on the functionality of "routing". Since the functionality of ``_addLocalToken``,  ``_setLocalToken``, ``_syncBranchBridgeAgent`` are implemented in ``BranchBridgeAgent.sol``, these internal functions should be moved to ``c.sol``. Currently, these internal functions just unwrap the payload and then wrap another payload to call ``BranchBridgeAgent.sol`` - a waste of time and only make the code harder to manage. Simplicity is the key for security. 
   
G. Systemic risks







### Time spent:
100 hours