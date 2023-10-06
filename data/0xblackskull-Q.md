### Contracts that lock funds
1. BranchBridgeAgent.sol
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L149
This contract receive ethers becoz `receive() external payable {}` not no way to withdraw funds because no withdraw function

