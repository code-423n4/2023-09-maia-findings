## 1. Layerzero Gas Estimate not Implemented Properly
According to the Layerzero documentation checklists, “useZro” should not be hardcoded to false, it should be passed as a parameter Instead.
https://layerzero.gitbook.io/docs/evm-guides/layerzero-integration-checklist


Affected Codes:
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L150

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L170


## 2. Layerzero ChainId not implemented properly
According to the Layerzero documentation checklists, chain ID should not be hardcoded, “admin restricted setters” should be used instead.

https://layerzero.gitbook.io/docs/evm-guides/layerzero-integration-checklist

Implementation of ChainId (localChainId) throughout this protocol, is made immutable without any way to securely update it, this could make these contracts inoperable if Layerzero updates their chain IDs.

Affected Files:

https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol#L22

https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L65

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L40


