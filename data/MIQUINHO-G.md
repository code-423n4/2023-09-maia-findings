### [G-01] Use != costs less gas than >
- BranchBridgeAgent.sol:911
- BranchBridgeAgent.sol:918
- BaseBranchRouter.sol:165
- BaseBranchRouter.sol:174
- VirtualAccount.sol:155
- ArbitrumBranchPort.sol:127
- ArbitrumBranchPort.sol:133
- BranchPort.sol:533
- BranchPort.sol:541
- RootBridgeAgent.sol:365
- RootBridgeAgent.sol:373
- RootBridgeAgent.sol:1153
- RootBridgeAgent.sol:1161
### [G-02] Use uint256 costs less gas than bool
- RootPort.sol:24
- RootPort.sol:29
- RootPort.sol:67
- RootPort.sol:72
- RootPort.sol:87
- RootPort.sol:99
- BranchPort.sol:32
- BranchPort.sol:43
- BranchPort.sol:54
- BranchPort.sol:71
- RootBridgeAgent.sol:68
### [G-03] Use hardcode address instead of address(this)
- BranchBridgeAgentExecutor.sol:92
- BranchBridgeAgentExecutor.sol:127
- BranchBridgeAgent.sol:142
- BranchBridgeAgent.sol:169
- BranchBridgeAgent.sol:580
- BranchBridgeAgent.sol:719
- BranchBridgeAgent.sol:740
- BranchBridgeAgent.sol:791
- BranchBridgeAgent.sol:945
- BranchPort.sol:535
- BranchPort.sol:543
### [G-04] Use x = x + y costs less gas than x += y
- VirtualAccount.sol:98
- BranchPort.sol:161
- BranchPort.sol:174
- BranchPort.sol:178
### [G-05] Use >= costs less gas than >
- BranchPort.sol:306