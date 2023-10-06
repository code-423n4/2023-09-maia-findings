## Summary

### Low Risk Issues

| |Issue|Instances|
|-|:-|:-:|
| [[L&#x2011;01](#l01-missing-events)] | Missing Event for critical functions or parameters change | 4 |
| [[L&#x2011;02](#l02-missing-access-control)] | Missing access control in `RootBridgeAgentFactory.sol#createBridgeAgent` | 1 |

### Non-critical Issues

| |Issue|Instances|
|-|:-|:-:|
| [[N&#x2011;01](#n01-parameter-not-used)] |  `_rootBridgeAgentFactoryAddress` parameter not used in `ArbitrumBranchBridgeAgentFactory.sol#createBridgeAgent` function | 2 |
| [[N&#x2011;02](#n02-address-canot-be-zero)] | Use `Address cannot be the zero address` instead of `Address cannot be 0` in require statements error messages | 23 |
| [[N&#x2011;03](#n03-typos)] | Typos | 1 |
| [[N&#x2011;04](#n03-unused-error)] | Unused error definition | 4 |
| [[N&#x2011;05](#n03-internal-function-standard)] | Follow Solidity standard naming conventions (internal function style rule) for functions | 2 |


## Low Risk Issues

### [L&#x2011;01] Missing Event for critical functions or parameters change

Events help non-contract tools to track changes, and events prevent users from being surprised by changes.

*There is 5 instance of this issue:*

```solidity
File: src/factories/ERC20hTokenBranchFactory.sol

96:    function createToken(string memory _name, string memory _symbol, uint8 _decimals, bool _addPrefix)

```

*GitHub*: [96](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/factories/ERC20hTokenBranchFactory.sol#L96C1-L96C103)

```solidity
File: src/factories/ERC20hTokenRootFactory.sol

76:    function createToken(string memory _name, string memory _symbol, uint8 _decimals)

```

*GitHub*: [76](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/factories/ERC20hTokenRootFactory.sol#L76)

```solidity
File: src/VirtualAccount.sol

51:    function withdrawNative(uint256 _amount) external override requiresApprovedCaller {
52:        msg.sender.safeTransferETH(_amount);
53:    }

56:    function withdrawERC20(address _token, uint256 _amount) external override requiresApprovedCaller {
57:        _token.safeTransfer(msg.sender, _amount);
58:    }

61:    function withdrawERC721(address _token, uint256 _tokenId) external override requiresApprovedCaller {
62:        ERC721(_token).transferFrom(address(this), msg.sender, _tokenId);
63    }

```

*GitHub*: [51, 56, 61](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/VirtualAccount.sol#L51C5-L63C6)

### [L&#x2011;02] Missing access control in `RootBridgeAgentFactory.sol#createBridgeAgent`

Due to the lack of access control mechanism in `RootBridgeAgentFactory.sol#createBridgeAgent` Anybody can create bridge agents, and this cause the `bridgeAgents` to grow endlessly.

*There is 1 instance of this issue:*

```solidity
File: src/factories/RootBridgeAgentFactory.sol

48:    function createBridgeAgent(address _newRootRouterAddress) external returns (address newBridgeAgent) {

```

*GitHub*: [48](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/factories/RootBridgeAgentFactory.sol#L48)

## Non-critical Issues

### [N&#x2011;01] `_rootBridgeAgentFactoryAddress` parameter not used in `ArbitrumBranchBridgeAgentFactory.sol#createBridgeAgent` function

The `_rootBridgeAgentFactoryAddress` parameter is not used in the function logic and its not makes any changes in the logic. Its only used for a checking the state variable `rootBridgeAgentFactoryAddress`. So this can be removed, its not make any effect on the business logic.

*There are 2 instances of this issue:*

```solidity
File: src/factories/ArbitrumBranchBridgeAgentFactory.sol

79:    function createBridgeAgent(
80:        address _newBranchRouterAddress,
81:        address _rootBridgeAgentAddress,
82:  @>    address _rootBridgeAgentFactoryAddress
83:    ) external virtual override returns (address newBridgeAgent) {
84:        require(
85:            msg.sender == localCoreBranchRouterAddress, "Only the Core Branch Router can create a new Bridge Agent."
86:        );
87:        require(
88:   @>       _rootBridgeAgentFactoryAddress == rootBridgeAgentFactoryAddress,
89:            "Root Bridge Agent Factory Address does not match."
90:        );
```

*GitHub*: [79](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/factories/ArbitrumBranchBridgeAgentFactory.sol#L79C1-L90C11)


```solidity
File: src/factories/BranchBridgeAgentFactory.sol
116:    function createBridgeAgent(
117:        address _newBranchRouterAddress,
118:        address _rootBridgeAgentAddress,
119:  @>    address _rootBridgeAgentFactoryAddress
120:    ) external virtual returns (address newBridgeAgent) {
121:        require(
122:            msg.sender == localCoreBranchRouterAddress, "Only the Core Branch Router can create a new Bridge Agent."
123:        );
124:        require(
125:  @>        _rootBridgeAgentFactoryAddress == rootBridgeAgentFactoryAddress,
126:            "Root Bridge Agent Factory Address does not match."
127:        );
```

*GitHub*: [116](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/factories/BranchBridgeAgentFactory.sol#L115C1-L126C11)


### [N&#x2011;02] Use `Address cannot be the zero address` instead of `Address cannot be 0` in require statements error messages

Contract sometimes uses `Address cannot be the zero address` and sometimes `Address cannot be 0`. To achieve a standard, its better to use `Address cannot be the zero address` instead of `Address cannot be 0` because its more meaningful and readable

*There are 23 instances of this issue:*

<details>
<summary>see instances</summary>

```solidity
File: src/ArbitrumBranchPort.sol

39:         require(_rootPortAddress != address(0), "Root Port Address cannot be 0");

```

*GitHub*: [39](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/ArbitrumBranchPort.sol#L39)

```solidity
File: src/BaseBranchRouter.sol

61:         require(_localBridgeAgentAddress != address(0), "Bridge Agent address cannot be 0");

```

*GitHub*: [61](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BaseBranchRouter.sol#L61)

```solidity
File: src/MulticallRootRouter.sol

93:        require(_localPortAddress != address(0), "Local Port Address cannot be 0");
94:        require(_multicallAddress != address(0), "Multicall Address cannot be 0");

110:       require(_bridgeAgentAddress != address(0), "Bridge Agent Address cannot be 0");

```

*GitHub*: [93](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/MulticallRootRouter.sol#L93), [94](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/MulticallRootRouter.sol#L94), [110](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/MulticallRootRouter.sol#L110)

```solidity
File: src/RootPort.sol

130:        require(_bridgeAgentFactory != address(0), "Bridge Agent Factory cannot be 0 address.");
131:        require(_coreRootRouter != address(0), "Core Root Router cannot be 0 address.");

152:        require(_coreRootBridgeAgent != address(0), "Core Root Bridge Agent cannot be 0 address.");
153:        require(_coreLocalBranchBridgeAgent != address(0), "Core Local Branch Bridge Agent cannot be 0 address.");
154:        require(_localBranchPortAddress != address(0), "Local Branch Port Address cannot be 0 address.");

```

*GitHub*: [130](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L130), [131](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L131), 
[152](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L152), [153](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L153), [154](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L154)

```solidity
File: src/factories/ArbitrumBranchBridgeAgentFactory.sol

57:         require(_coreRootBridgeAgent != address(0), "Core Root Bridge Agent Address cannot be 0");

```

*GitHub*: [57](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/factories/ArbitrumBranchBridgeAgentFactory.sol#L57)

```solidity
File: src/factories/BranchBridgeAgentFactory.sol

61:        require(_rootBridgeAgentFactoryAddress != address(0), "Root Bridge Agent Factory Address cannot be 0");

66:        require(_localCoreBranchRouterAddress != address(0), "Core Branch Router Address cannot be 0");
67:        require(_localPortAddress != address(0), "Port Address cannot be 0");
68:        require(_owner != address(0), "Owner cannot be 0");

88:        require(_coreRootBridgeAgent != address(0), "Core Root Bridge Agent cannot be 0");

```

*GitHub*: [61](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/factories/BranchBridgeAgentFactory.sol#L61C1-L61C112), [66](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/factories/BranchBridgeAgentFactory.sol#L66), [67](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/factories/BranchBridgeAgentFactory.sol#L67), [68](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/factories/BranchBridgeAgentFactory.sol#L68), [88](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/factories/BranchBridgeAgentFactory.sol#L88)


```solidity
File: src/factories/ERC20hTokenBranchFactory.sol

43:        require(_localPortAddress != address(0), "Port address cannot be 0");

61:        require(_coreRouter != address(0), "CoreRouter address cannot be 0");

```

*GitHub*: [43](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/factories/ERC20hTokenBranchFactory.sol#L43), [61](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/factories/ERC20hTokenBranchFactory.sol#L61)

```solidity
File: src/factories/ERC20hTokenRootFactory.sol

35:        require(_rootPortAddress != address(0), "Root Port Address cannot be 0");

50:        require(_coreRouter != address(0), "CoreRouter address cannot be 0");

```

*GitHub*: [35](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/factories/ERC20hTokenRootFactory.sol#L35), [50](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/factories/ERC20hTokenRootFactory.sol#L50)

```solidity
File: src/factories/RootBridgeAgentFactory.sol

32:        require(_rootPortAddress != address(0), "Root Port Address cannot be 0");

```

*GitHub*: [32](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/factories/RootBridgeAgentFactory.sol#L32)

```solidity
File: src/token/ERC20hTokenRoot.sol

39:        require(_rootPortAddress != address(0), "Root Port Address cannot be 0");
40:        require(_factoryAddress != address(0), "Factory Address cannot be 0");

```

*GitHub*: [39](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/token/ERC20hTokenRoot.sol#L39), [40](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/token/ERC20hTokenRoot.sol#L40)

</details>


### [N&#x2011;03] Typos

*There are 1 instances of this issue:*

```solidity
File: src/interfaces/IBranchBridgeAgent.sol

// @audit `callOutSystem()` is an external function not internal 
130:     * @notice Internal function performs call to Layerzero Enpoint Contract for cross-chain messaging.

```
*GitHub*: [130](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/interfaces/IBranchBridgeAgent.sol#L130)

### [N&#x2011;04] Unused error definition
Note that there may be cases where an error superficially appears to be used, but this is only because there are multiple definitions of the error in different files. In such cases, the error definition should be moved into a separate file. The instances below are the unused definitions.

There are 4 instances of this issue:

```solidity

File: src/interfaces/IBranchPort.sol

255:    error NotEnoughDebtToRepay();

```
*GitHub*: [255](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/interfaces/IBranchPort.sol#L255)

```solidity

File: src/interfaces/IERC20hTokenBranchFactory.sol

34:    error UnrecognizedPort();

```
*GitHub*: [34](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/interfaces/IERC20hTokenBranchFactory.sol#L34)

```solidity

File: src/interfaces/IERC20hTokenRoot.sol

57:    error UnrecognizedPort();

```
*GitHub*: [57](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/interfaces/IERC20hTokenRoot.sol#L57)

```solidity

File: src/interfaces/IPortStrategy.sol

29:     error UnrecognizedPort();

```
*GitHub*: [29](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/interfaces/IPortStrategy.sol#L29)

### [N&#x2011;05] Follow Solidity standard naming conventions (internal function style rule) for functions

The below codes don’t follow Solidity’s standard naming convention,

internal functions : the mixedCase format starting with an underscore (_mixedCase starting with an underscore)

*There are 2 instances of this issue:*

```solidity
File: src/VirtualAccount.sol

147:    function isContract(address addr) internal view returns (bool) {

```
*GitHub*: [147](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/VirtualAccount.sol#L147)

```solidity
File: src/RootPort.sol

359:    function addVirtualAccount(address _user) internal returns (VirtualAccount newAccount) {

```
*GitHub*: [359](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L359)