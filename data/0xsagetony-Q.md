
#### Low Risk and Non-Critical Issues

1. The `_settlementNonce` was not checked if it was valid in `ArbitrumBranchBridgeAgent.sol/_performFallbackCall()`.

   - **File:** [ArbitrumBranchBridgeAgent.sol](https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchBridgeAgent.sol)
   - **Function:** [_performFallbackCall()](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/ArbitrumBranchBridgeAgent.sol#L112)

2. There was no tracking of the amount going in or out of the root chain in `ArbitrumBranchBridgeAgent.sol/withdrawFromPort()`.

   - **File:** [ArbitrumBranchBridgeAgent.sol](https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchBridgeAgent.sol)
   - **Function:** [withdrawFromPort()](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/ArbitrumBranchBridgeAgent.sol#L79C14-L79C30)

3. There is no check on `_recipient` and the `_localAddress` address which might lead to token loss in `ArbitrumBranchPort.sol/_bridgeIn()`.

   - **File:** [ArbitrumBranchPort.sol](https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol)
   - **Function:** [_bridgeIn()](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/ArbitrumBranchPort.sol#L107)

4. There was no check on the `localchainId` validity in the `BranchBridgeAgent` constructor.

   - **File:** [BranchBridgeAgent.sol](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol)
   - **Constructor:** [BranchBridgeAgent()](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L117)

5. `depositNounce` used for core system operations is defined as `uint32` in `BranchBridgeAgent/callOutSystem()`. If it exceeds 4,294,967,295 it could lead to a DOS attack.

   - **File:** [BranchBridgeAgent.sol](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol)
   - **Function:** [callOutSystem()](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L180)

6. The refundee address was not checked in `BranchBridgeAgent/callOutAndBridge`.

   - **File:** [BranchBridgeAgent.sol](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol)
   - **Function:** [callOutAndBridge()](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L209)

7. The `dParams.htokens.length` should be checked before encoding in `BranchBridgeAgent/callOutAndBridgeMultiple`. If the length is more than 255 then it will be downcasted and lead to the loss of value.

   - **File:** [BranchBridgeAgent.sol](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol)
   - **Function:** [callOutAndBridgeMultiple()](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L231)

8. The recipient and router address were not checked in `BranchBridgeAgentExecutor/executeWithSettlementMultiple()`.

   - **File:** [BranchBridgeAgentExecutor.sol](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgentExecutor.sol)
   - **Function:** [executeWithSettlementMultiple()](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgentExecutor.sol#L104)

9. There was no address check in `BranchPort/withdraw`.

    - **File:** [BranchPort.sol](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol)
    - **Function:** [withdraw()](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L226)

10. The addresses were not checked in `CoreRootRouter/initialize()` and `CoreRootRouter/removeBranchBridgeAgent()`.

    - **File:** [CoreRootRouter.sol](https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol)
    - **Function:** [initialize()](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/CoreRootRouter.sol#L83)
    - **Function:** [removeBranchBridgeAgent()](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/CoreRootRouter.sol#L186)

11. `lzendpointaddress` was not checked in the constructor of the `RootBridgeAgentFactory` contract.

    - **File:** [RootBridgeAgentFactory.sol](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L105)
    - **Constructor:** [RootBridgeAgentFactory()](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L105)

12. No address check on `RootBridgeAgentExecutor/executeSystemRequest()`.

    - **File:** [RootBridgeAgentExecutor.sol](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L50)
    - **Function:** [executeSystemRequest()](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgentExecutor.sol#L50)

13. Inconsistent use of `require` and `revert` statements. One of the statements should be used across the board. Revert statements are the recommended approach to save gas, in `initialize()`, `initializeCore()`, `setCoreRouter()` of `RootPort.sol`.

    - **File:** [RootPort.sol](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L147)
    - **Function:** [initialize()](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L129)
    - **Function:** [initializeCore()](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L147)

14. Missing events for core system function in `BranchPort/addBridgeAgent()`.

    - **File:** [BranchPort.sol](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L319)

15. Errors should be prefixed with the contract name for better error handling. E.g:

   `error BranchPort__AlreadyAddedBridgeAgent` instead of `error AlreadyAddedBridgeAgent()`.

   - **File:** [BranchPort.sol](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L319)

Please address these issues to improve the security and robustness of your contract.