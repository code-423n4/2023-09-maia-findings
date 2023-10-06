https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L85C5-L85C5  

 mapping(uint256 chainId => mapping(uint256 nonce => uint256 state)) public executionState;
here we can change unit256 state to bool as it will save many gas and we are using this for many palace.

https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L72
 Call calldata _call = calls[i];
we can define _call before for loop.

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L318C9-L318C62
for (uint256 i = 0; i < settlement.hTokens.length;) {
calculate earlier settlement.hTokens.length .

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1067C9-L1067C75

address[] memory hTokens = new address[](_globalAddresses.length);
_globalAddresses.length has been used many times. We can calculate this constant earlier.


https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1070C33-L1070C47
hTokens.length   can be used as contant.

https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L557
 for (uint256 i = 0; i < outputTokens.length;) {
outputTokens.length can be used as constant.

https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L455
 for (uint256 i = 0; i < outputParams.outputTokens.length;) {
outputParams.outputTokens.length can be used as constant.


https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L157
 getPortStrategyTokenDebt[msg.sender][_token] += _amount;
+= takes more gas.

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L169
 getPortStrategyTokenDebt[msg.sender][_token] -= _amount;
-= takes more gas.

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L172
getStrategyTokenDebt[_token] -= _amount;

-= takes more gas.