| ID    | Title |
| -------- | ------- |
| 01 | Project Upgrade and Stop Scenario should be added 
| 02 | Missing Event for initialize

## [01] Project Upgrade and Stop Scenario should be added 

At the start of the project, the system may need to be stopped or upgraded. I suggest you have a script beforehand and add it to the documentation. This can also be called an “EMERGENCY STOP (CIRCUIT BREAKER) PATTERN”.

This can be done by adding the ‘pause’ architecture, which is included in many projects, to the critical functions and by authorizing the existing onlyOwner.

https://github.com/maxwoe/solidity_patterns/blob/master/security/EmergencyStop.sol

## [02] Missing Event for initialize

Context:

```solidity

60    function initialize(address _localBridgeAgentAddress) external onlyOwner {

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L60

```solidity

122    function initialize(address _coreBranchRouter, address _bridgeAgentFactory) external virtual onlyOwner {

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L122

```solidity

83    function initialize(address _bridgeAgentAddress, address _hTokenFactory) external onlyOwner {

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L83

```solidity

109    function initialize(address _bridgeAgentAddress) external onlyOwner {

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L109

```solidity

129    function initialize(address _bridgeAgentFactory, address _coreRootRouter) external onlyOwner {

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L129

```solidity
56    function initialize(address _coreRootBridgeAgent) external override onlyOwner {
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ArbitrumBranchBridgeAgentFactory.sol#L56

```solidity
87    function initialize(address _coreRootBridgeAgent) external virtual onlyOwner {
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/BranchBridgeAgentFactory.sol#L87

```solidity
60    function initialize(address _wrappedNativeTokenAddress, address _coreRouter) external onlyOwner {
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenBranchFactory.sol#L60

```solidity
49    function initialize(address _coreRouter) external onlyOwner {
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenRootFactory.sol#L49



Events help non-contract tools to track changes and prevent users from being surprised by changes. Issuing an event-emit during initialization is a detail that many projects skip.

Recommendation
Add an Event-Emit.