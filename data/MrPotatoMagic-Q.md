# Quality Assurance Report

| QA     | Issue                                                                                                                                                                          | Instances |
|--------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------|
| [N-01] | Missing event emission for critical state changes                                                                                                                              | 21        |
| [N-02] | Avoid naming mappings with `get` in the beginning                                                                                                                              | 2         |
| [N-03] | Shift ERC721 receiver import to `IVirtualAccount.sol` to avoid duplicating ERC721 receiver import                                                                              | 1         |
| [N-04] | Typo error in comments                                                                                                                                                         | 9         |
| [N-05] | No need to limit `settlementNonce` input to uint32                                                                                                                             | 2         |
| [N-06] | Setting `deposit.owner = address(0);` is not required                                                                                                                          | 1         |
| [L-01] | Consider using ERC1155Holder instead of ERC1155Receiver due to OpenZeppelin's latest v5.0 release candidate changes                                                            | 1         |
| [L-02] | Mapping key-value pair names are reversed                                                                                                                                      | 4         |
| [L-03] | Do not hardcode `_zroPaymentAddress` field to address(0) to allow future ZRO fee payments and prevent Bridge Agents from falling apart incase LayerZero makes breaking changes | 2         |
| [L-04] | Do not hardcode `_payInZRO` field to false to allow future ZRO fee payment estimation for payloads                                                                             | 2         |
| [L-05] | Leave some degree of configurability for extra parameters in `_adapterParams` to allow for feature extensions                                                                  | 2         |
| [L-06] | Do not hardcode LayerZero's proprietary chainIds                                                                                                                               | 2         |
| [L-07] | Array entry not deleted when removing bridge agent                                                                                                                             | 1         |
| [L-08] | Double entries in `strategyTokens`, `portStrategies`, `bridgeAgents` and `bridgeAgentFactories` arrays are not prevented                                                       | 4         |

### Total Non-Critical issues: 36 instances across 6 issues

### Total Low-severity issues: 18 instances across 8 issues

### Total issues: 54 instances across 14 issues

## [N-01] Missing event emission for critical state changes

There are 21 instances of this issue:

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/token/ERC20hTokenRoot.sol#L39C6-L44C44

Missing event emission for state variables `localChainId` and `factoryAddress`
```solidity
File: ERC20hTokenRoot.sol
31:     constructor(
32:         uint16 _localChainId,
33:         address _factoryAddress,
34:         address _rootPortAddress,
35:         string memory _name,
36:         string memory _symbol,
37:         uint8 _decimals
38:     ) ERC20(string(string.concat(_name)), string(string.concat(_symbol)), _decimals) {
39:         require(_rootPortAddress != address(0), "Root Port Address cannot be 0");
40:         require(_factoryAddress != address(0), "Factory Address cannot be 0");
41:        
42:         localChainId = _localChainId;
43:         factoryAddress = _factoryAddress;
44:         _initializeOwner(_rootPortAddress);
45:         //@audit NC - missing event emission
46:     }
```

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/factories/ERC20hTokenRootFactory.sol#L34C4-L40C1

Missing event emission for state variables `localChainId` and `rootPortAddress`
```solidity
File: ERC20hTokenRootFactory.sol
34:     constructor(uint16 _localChainId, address _rootPortAddress) {
35:         require(_rootPortAddress != address(0), "Root Port Address cannot be 0");
36:         localChainId = _localChainId;
37:         rootPortAddress = _rootPortAddress;
38:         _initializeOwner(msg.sender);
39:         //@audit NC - missing event emission
40:     }
```

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/factories/ERC20hTokenRootFactory.sol#L49C5-L54C1

Missing event emission for state variable `coreRootRouterAddress`
```solidity
File: ERC20hTokenRootFactory.sol
50:     function initialize(address _coreRouter) external onlyOwner {
51:         require(_coreRouter != address(0), "CoreRouter address cannot be 0");
52:         coreRootRouterAddress = _coreRouter; //@audit NC - missing event emission
53:         renounceOwnership();
54:     }
```

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/factories/ERC20hTokenBranchFactory.sol#L42C5-L49C6

Missing event emission for state variables from Line 44 to 47
```solidity
File: ERC20hTokenBranchFactory.sol
42:     constructor(uint16 _localChainId, address _localPortAddress, string memory _chainName, string memory _chainSymbol) {
43:         require(_localPortAddress != address(0), "Port address cannot be 0");
44:         chainName = string.concat(_chainName, " Ulysses");
45:         chainSymbol = string.concat(_chainSymbol, "-u");
46:         localChainId = _localChainId;
47:         localPortAddress = _localPortAddress;
48:         _initializeOwner(msg.sender);
49:         //@audit NC - missing event emission
50:     }
```

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/factories/ERC20hTokenBranchFactory.sol#L60C1-L77C6

Missing event emission for state changes on Line 73 and 75
```solidity
File: ERC20hTokenBranchFactory.sol
61:     function initialize(address _wrappedNativeTokenAddress, address _coreRouter) external onlyOwner {
62:         require(_coreRouter != address(0), "CoreRouter address cannot be 0");
63: 
64:         ERC20hTokenBranch newToken = new ERC20hTokenBranch(
65:             chainName,
66:             chainSymbol,
67:             ERC20(_wrappedNativeTokenAddress).name(),
68:             ERC20(_wrappedNativeTokenAddress).symbol(),
69:             ERC20(_wrappedNativeTokenAddress).decimals(),
70:             localPortAddress
71:         );
72: 
73:         hTokens.push(newToken);
74: 
75:         localCoreRouterAddress = _coreRouter;
76: 
77:         renounceOwnership();
78:         //@audit NC - missing event emission
79:     }
```

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/factories/BranchBridgeAgentFactory.sol#L52C1-L77C6

Missing event emission for state variables from Line 70 to 75
```solidity
File: BranchBridgeAgentFactory.sol
52:     constructor(
53:         uint16 _localChainId,
54:         uint16 _rootChainId,
55:         address _rootBridgeAgentFactoryAddress,
56:         address _lzEndpointAddress,
57:         address _localCoreBranchRouterAddress,
58:         address _localPortAddress,
59:         address _owner
60:     ) {
61:         require(_rootBridgeAgentFactoryAddress != address(0), "Root Bridge Agent Factory Address cannot be 0");
62:         require(
63:             _lzEndpointAddress != address(0) || _rootChainId == _localChainId,
64:             "Layerzero Endpoint Address cannot be the zero address."
65:         );
66:         require(_localCoreBranchRouterAddress != address(0), "Core Branch Router Address cannot be 0");
67:         require(_localPortAddress != address(0), "Port Address cannot be 0");
68:         require(_owner != address(0), "Owner cannot be 0");
69: 
70:         localChainId = _localChainId;
71:         rootChainId = _rootChainId;
72:         rootBridgeAgentFactoryAddress = _rootBridgeAgentFactoryAddress;
73:         lzEndpointAddress = _lzEndpointAddress;
74:         localCoreBranchRouterAddress = _localCoreBranchRouterAddress;
75:         localPortAddress = _localPortAddress;
76:         _initializeOwner(_owner);
77:         //@audit NC - missing event emission
78:     }
```

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/VirtualAccount.sol#L35C1-L38C6

Missing event emission for state variables `userAddress` and `localPortAddress`
```solidity
File: VirtualAccount.sol
35:     constructor(address _userAddress, address _localPortAddress) {
36:         userAddress = _userAddress;
37:         localPortAddress = _localPortAddress;
38:         //@audit NC - missing event emission
39:     }
```

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L111C1-L118C6

Missing event emission for state variables on Lines 114,115,118 and 119
```solidity
File: RootPort.sol
113:     constructor(uint256 _localChainId) {
114:         localChainId = _localChainId;
115:         isChainId[_localChainId] = true;
116: 
117:         _initializeOwner(msg.sender);
118:         _setup = true;
119:         _setupCore = true;
120:         //@audit NC - missing event emission
121:     }
```

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L129C1-L139C6

Missing event emission for state changes from Line 136 to 141
```solidity
File: RootPort.sol
132:     function initialize(address _bridgeAgentFactory, address _coreRootRouter) external onlyOwner {
133:         require(_bridgeAgentFactory != address(0), "Bridge Agent Factory cannot be 0 address.");
134:         require(_coreRootRouter != address(0), "Core Root Router cannot be 0 address.");
135:         require(_setup, "Setup ended.");
136:         _setup = false; 
137:      
138:         isBridgeAgentFactory[_bridgeAgentFactory] = true;
139:         bridgeAgentFactories.push(_bridgeAgentFactory);
140: 
141:         coreRootRouterAddress = _coreRootRouter;
142:         //@audit NC - missing event emission
143:     }
```

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L147C1-L163C6

Missing event emission for state changes from Line 161 to 166
```solidity
File: RootPort.sol
151:     function initializeCore(
152:         address _coreRootBridgeAgent,
153:         address _coreLocalBranchBridgeAgent,
154:         address _localBranchPortAddress
155:     ) external onlyOwner {
156:         require(_coreRootBridgeAgent != address(0), "Core Root Bridge Agent cannot be 0 address.");
157:         require(_coreLocalBranchBridgeAgent != address(0), "Core Local Branch Bridge Agent cannot be 0 address.");
158:         require(_localBranchPortAddress != address(0), "Local Branch Port Address cannot be 0 address.");
159:         require(isBridgeAgent[_coreRootBridgeAgent], "Core Bridge Agent doesn't exist.");
160:         require(_setupCore, "Core Setup ended.");
161:         _setupCore = false;
162:     
163:         coreRootBridgeAgentAddress = _coreRootBridgeAgent;
164:         localBranchPortAddress = _localBranchPortAddress;
165:         IBridgeAgent(_coreRootBridgeAgent).syncBranchBridgeAgent(_coreLocalBranchBridgeAgent, localChainId);
166:         getBridgeAgentManager[_coreRootBridgeAgent] = owner();
167:         //@audit NC - missing event emission
168:     }
```

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L105C1-L122C6

Missing event emission for state changes from Line 118 to 124
```solidity
File: RootBridgeAgent.sol
108:     constructor(
109:         uint16 _localChainId,
110:         address _lzEndpointAddress,
111:         address _localPortAddress,
112:         address _localRouterAddress
113:     ) {
114:         require(_lzEndpointAddress != address(0), "Layerzero Enpoint Address cannot be zero address");
115:         require(_localPortAddress != address(0), "Port Address cannot be zero address");
116:         require(_localRouterAddress != address(0), "Router Address cannot be zero address");
117: 
118:         factoryAddress = msg.sender;
119:         localChainId = _localChainId;
120:         lzEndpointAddress = _lzEndpointAddress;
121:         localPortAddress = _localPortAddress;
122:         localRouterAddress = _localRouterAddress;
123:         bridgeAgentExecutorAddress = DeployRootBridgeAgentExecutor.deploy(address(this));
124:         settlementNonce = 1;
125:         //@audit NC - missing event emission
126:     }
```

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/MulticallRootRouter.sol#L92C1-L100C6

Missing event emission for state variables from Line 96 to 98
```solidity
File: MulticallRootRouter.sol
092:     constructor(uint256 _localChainId, address _localPortAddress, address _multicallAddress) {
093:         require(_localPortAddress != address(0), "Local Port Address cannot be 0");
094:         require(_multicallAddress != address(0), "Multicall Address cannot be 0");
095: 
096:         localChainId = _localChainId;
097:         localPortAddress = _localPortAddress;
098:         multicallAddress = _multicallAddress;
099:         _initializeOwner(msg.sender);
100:         //@audit NC - missing event emission
101:     }
```

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/MulticallRootRouter.sol#L109C1-L115C6

Missing event emission for state variables on Lines 113 and 114
```solidity
File: MulticallRootRouter.sol
110:     function initialize(address _bridgeAgentAddress) external onlyOwner {
111:         require(_bridgeAgentAddress != address(0), "Bridge Agent Address cannot be 0");
112: 
113:         bridgeAgentAddress = payable(_bridgeAgentAddress);
114:         bridgeAgentExecutorAddress = IBridgeAgent(_bridgeAgentAddress).bridgeAgentExecutorAddress();
115:         renounceOwnership();
116:         //@audit NC - missing event emission
117:     }
```

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/CoreRootRouter.sol#L71C1-L77C6

Missing event emission for state variables on Lines 72,73 and 76
```solidity
File: CoreRootRouter.sol
71:     constructor(uint256 _rootChainId, address _rootPortAddress) {
72:         rootChainId = _rootChainId;
73:         rootPortAddress = _rootPortAddress;
74: 
75:         _initializeOwner(msg.sender);
76:         _setup = true;
77:         //@audit NC - missing event emission
78:     }
```

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/CoreBranchRouter.sol#L30C1-L32C6

Missing event emission for hTokenFactoryAddress state variable
```solidity
File: CoreBranchRouter.sol
30:     constructor(address _hTokenFactoryAddress) BaseBranchRouter() {
31:         hTokenFactoryAddress = _hTokenFactoryAddress;//@audit NC - missing event emission
32:     }
```

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L122C1-L132C6

Missing event emission for state variables from Line 129 to 131
```solidity
File: BranchPort.sol
122:     function initialize(address _coreBranchRouter, address _bridgeAgentFactory) external virtual onlyOwner {
123:         require(coreBranchRouterAddress == address(0), "Contract already initialized");
124:         require(!isBridgeAgentFactory[_bridgeAgentFactory], "Contract already initialized");
125: 
126:         require(_coreBranchRouter != address(0), "CoreBranchRouter is zero address");
127:         require(_bridgeAgentFactory != address(0), "BridgeAgentFactory is zero address");
128: 
129:         coreBranchRouterAddress = _coreBranchRouter;
130:         isBridgeAgentFactory[_bridgeAgentFactory] = true;
131:         bridgeAgentFactories.push(_bridgeAgentFactory);
132:         //@audit NC - missing event emission
133:     }
```

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L319C1-L324C6

Missing event emission for state changes on lines 323, 324
```solidity
File: BranchPort.sol
320:     function addBridgeAgent(address _bridgeAgent) external override requiresBridgeAgentFactory {
321:         if (isBridgeAgent[_bridgeAgent]) revert AlreadyAddedBridgeAgent();
322: 
323:         isBridgeAgent[_bridgeAgent] = true;
324:         bridgeAgents.push(_bridgeAgent); 
325:         //@audit NC - missing event emission
326:     }
```

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L331C1-L335C6

Missing event emission for state variable on Line 336
```solidity
File: BranchPort.sol
333:     function setCoreRouter(address _newCoreRouter) external override requiresCoreRouter {
334:         require(coreBranchRouterAddress != address(0), "CoreRouter address is zero");
335:         require(_newCoreRouter != address(0), "New CoreRouter address is zero");
336:         coreBranchRouterAddress = _newCoreRouter;
337:         //@audit NC - missing event emission
338:     }
```

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L117C1-L143C6

Missing event emission for state variables from Lines 134 to 143
```solidity
File: BranchBridgeAgent.sol
117:      */
118:     constructor(
119:         uint16 _rootChainId,
120:         uint16 _localChainId,
121:         address _rootBridgeAgentAddress,
122:         address _lzEndpointAddress,
123:         address _localRouterAddress,
124:         address _localPortAddress
125:     ) {
126:         require(_rootBridgeAgentAddress != address(0), "Root Bridge Agent Address cannot be the zero address.");
127:         require(
128:             _lzEndpointAddress != address(0) || _rootChainId == _localChainId,
129:             "Layerzero Endpoint Address cannot be the zero address."
130:         );
131:         require(_localRouterAddress != address(0), "Local Router Address cannot be the zero address.");
132:         require(_localPortAddress != address(0), "Local Port Address cannot be the zero address.");
133: 
134:         localChainId = _localChainId;
135:         rootChainId = _rootChainId;
136:         rootBridgeAgentAddress = _rootBridgeAgentAddress;
137:         lzEndpointAddress = _lzEndpointAddress;
138:         localRouterAddress = _localRouterAddress;
139:         localPortAddress = _localPortAddress;
140:         bridgeAgentExecutorAddress = DeployBranchBridgeAgentExecutor.deploy();
141:         depositNonce = 1;
142: 
143:         rootBridgeAgentPath = abi.encodePacked(_rootBridgeAgentAddress, address(this));
144:         //@audit NC - missing event emission
145:     }
```

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BaseBranchRouter.sol#L60C1-L67C6

Missing event emission for state variables from Lines 62 to 64
```solidity
File: BaseBranchRouter.sol
60:     function initialize(address _localBridgeAgentAddress) external onlyOwner {
61:         require(_localBridgeAgentAddress != address(0), "Bridge Agent address cannot be 0");
62:         localBridgeAgentAddress = _localBridgeAgentAddress;
63:         localPortAddress = IBridgeAgent(_localBridgeAgentAddress).localPortAddress();
64:         bridgeAgentExecutorAddress = IBridgeAgent(_localBridgeAgentAddress).bridgeAgentExecutorAddress();
65:         //@audit NC - missing event emission
66:         renounceOwnership();
67:     }
```

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/ArbitrumBranchPort.sol#L38C1-L43C6

Missing event emission for state variables on Lines 41 and 42
```solidity
File: ArbitrumBranchPort.sol
38:     constructor(uint16 _localChainId, address _rootPortAddress, address _owner) BranchPort(_owner) {
39:         require(_rootPortAddress != address(0), "Root Port Address cannot be 0");
40: 
41:         localChainId = _localChainId;
42:         rootPortAddress = _rootPortAddress;
43:         //@audit NC - missing event emission
44:     }
```

## [N-02] Avoid naming mappings with `get` in the beginning

Mapping names starting with "get" can be misleading since "get" is usually used for getters that do no make any state changes and only read state. Thus, if we have a statement like `getTokenBalance[chainId] += amount;`, it can be potentially misleading since we make state changes to a mapping which seems like a getter on first sight.

There are 2 instances of this issue (Note: Most bridgeAgent and Port contracts have this issue as well but I have not mentioned them here explicitly):

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/token/ERC20hTokenRoot.sol#L58

```solidity
File: src/token/ERC20hTokenRoot.sol
58:    getTokenBalance[chainId] += amount;
```

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/token/ERC20hTokenRoot.sol#L70

```solidity
File: src/token/ERC20hTokenRoot.sol
58:    getTokenBalance[chainId] -= amount;
```

## [N-03] Shift ERC721 receiver import to `IVirtualAccount.sol` to avoid duplicating ERC721 receiver import

Shift all ERC721 and ERC1155 receiver imports to interface `IVirtualAccount.sol` to avoid duplicating ERC721 receiver import and ensure code maintainability.

There is 1 instance of this issue:

We can see below that both VirtualAccount.sol and IVirtualAccount.sol have imported the IERC721Receiver interface. 

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/VirtualAccount.sol#L8C1-L12C1

```solidity
File: src/VirtualAccount.sol
9:  import {ERC1155Receiver} from "@openzeppelin/contracts/token/ERC1155/utils/ERC1155Receiver.sol";
10: import {IERC1155Receiver} from "@openzeppelin/contracts/token/ERC1155/IERC1155Receiver.sol";
11: import {IERC721Receiver} from "@openzeppelin/contracts/token/ERC721/IERC721Receiver.sol";
```

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/interfaces/IVirtualAccount.sol#L4

```solidity
File: src/interfaces/IVirtualAccount.sol
4: import {IERC721Receiver} from "@openzeppelin/contracts/token/ERC721/IERC721Receiver.sol";
```

## [N-04] Typo error in comments

There are 9 instances of this issue:

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/interfaces/IRootBridgeAgent.sol#L50C1-L51C68

Correct "singned" to "signed"
```solidity
File: IRootBridgeAgent.sol
50:  *       0x04 | Call to Root Router without Deposit + singned message.
51:  *       0x05 | Call to Root Router with Deposit + singned message.
```

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L42C1-L44C1

Correct "core router" to "core root bridge agent"
```solidity
File: RootPort.sol
43:     /// @notice The address of the core router in charge of adding new tokens to the system.
44:     address public coreRootBridgeAgentAddress;
```

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L60

Correct "Mapping from address to BridgeAgent" to "Mapping from chainId to IsActive (bool)"
```solidity
File: RootPort.sol
61:     /// @notice Mapping from address to Bridge Agent. 
62:     mapping(uint256 chainId => bool isActive) public isChainId;
```

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L64

Correct "Layerzer Zero" to "LayerZero"
```solidity
File: RootBridgeAgent.sol
65:     /// @notice Message Path for each connected Branch Bridge Agent as bytes for Layzer Zero interaction = localAddress + destinationAddress abi.encodePacked()
```

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/CoreRootRouter.sol#L153

Correct the below statement to "@param _dstChainId Chain Id of the branch chain for the bridge agent to be toggled"
```solidity
File: src/CoreRootRouter.sol
153:      * @param _dstChainId Chain Id of the branch chain where the new Bridge Agent will be deployed.
```

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/CoreRootRouter.sol#L183

Correct below comment to "@param _dstChainId Chain Id of the branch chain for the bridge agent to be removed"
```solidity
File: src/CoreRootRouter.sol
183:      * @param _dstChainId Chain Id of the branch chain where the new Bridge Agent will be deployed.
```

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L177

Correct "startegy" to "strategy"
```solidity
File: BranchPort.sol
178:  // Withdraw tokens from startegy 
```

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L86C1-L87C76

Correct "deposits hash" to "deposits nonce"
```solidity
File: src/BranchBridgeAgent.sol
86:     /// @notice Mapping from Pending deposits hash to Deposit Struct.
87:     mapping(uint256 depositNonce => Deposit depositInfo) public getDeposit;
```

## [N-05] No need to limit `settlementNonce` input to uint32

There are 2 instances of this issue:

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L135C1-L137C6

Mapping getSettlement supports type(uint256).max - 1 number of nonces while in the function getSettlementEntry below we limit _settlementNonce input only till type(uint32).max - 1. There is no need to limit this input to uint32. Although uint32 in itself is quite large, there does not seem to be a problem making this uint256.

```solidity
File: RootBridgeAgent.sol
139:     function getSettlementEntry(uint32 _settlementNonce) external view override returns (Settlement memory) {
140:         return getSettlement[_settlementNonce];
141:     }
```

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BaseBranchRouter.sol#L74

```solidity
File: BaseBranchRouter.sol
75:     function getDepositEntry(uint32 _depositNonce) external view override returns (Deposit memory) {
76:         return IBridgeAgent(localBridgeAgentAddress).getDepositEntry(_depositNonce);
77:     }
```

## [N-06] Setting `deposit.owner = address(0);` is not required

There is 1 instance of this issue:

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L444

Setting deposit.owner to address(0) is not required on Line 444 since we anyways delete the deposit info for that _depositNonce on Line 456.
```solidity
File: src/BranchBridgeAgent.sol
444: deposit.owner = address(0);
456: delete getDeposit[_depositNonce];
```

## [L-01] Consider using ERC1155Holder instead of ERC1155Receiver due to OpenZeppelin's latest v5.0 release candidate changes

View OpenZeppelin's v5.0 release candidate changes [here](https://github.com/OpenZeppelin/openzeppelin-contracts/releases/tag/v5.0.0-rc.0)

There is 1 instance of this issue:

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/VirtualAccount.sol#L9C1-L9C97

```solidity
File: src/VirtualAccount.sol
9: import {ERC1155Receiver} from "@openzeppelin/contracts/token/ERC1155/utils/ERC1155Receiver.sol";
```

## [L-02] Mapping key-value pair names are reversed

Keys are named with value names and Values are named with key names. This can be difficult to read and maintain as keys and values are referenced using their names. 

There are 4 instances of this:

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L86C1-L101C37

Below in the 4 instances of the mappings, we can see that the key-value pair names are reversed. For example, in mapping `getGlobalTokenFromLocal`, the first key is named `address chainId` while the second key is named `uint256 localAddress`. As we know, chainIds cannot be addresses and localAddress cannot be an uint256. 
```solidity
File: RootPort.sol
091:     /// @notice ChainId -> Local Address -> Global Address
092:     mapping(address chainId => mapping(uint256 localAddress => address globalAddress)) public getGlobalTokenFromLocal;
093: 
094:     /// @notice ChainId -> Global Address -> Local Address
095:     mapping(address chainId => mapping(uint256 globalAddress => address localAddress)) public getLocalTokenFromGlobal;
096: 
097:     /// @notice ChainId -> Underlying Address -> Local Address
098:     mapping(address chainId => mapping(uint256 underlyingAddress => address localAddress)) public
099:         getLocalTokenFromUnderlying;
100: 
101:     /// @notice Mapping from Local Address to Underlying Address.
102:     mapping(address chainId => mapping(uint256 localAddress => address underlyingAddress)) public
103:         getUnderlyingTokenFromLocal;
104: 
```
Additionally, if we check this getter function in the same contract, we can further prove that the namings are reversed in the original mappings above. (Note: The contracts function as expected since only names are reversed)
```solidity
File: RootPort.sol
195:     function _getLocalToken(address _localAddress, uint256 _srcChainId, uint256 _dstChainId)
196:         internal
197:         view
198:         returns (address)
199:     {
200:         address globalAddress = getGlobalTokenFromLocal[_localAddress][_srcChainId];
201:         return getLocalTokenFromGlobal[globalAddress][_dstChainId];
202:     }
```
**Solution:**
```solidity
File: RootPort.sol
091:     /// @notice Local Address -> ChainId -> Global Address
092:     mapping(address localAddress => mapping(uint256 chainId => address globalAddress)) public getGlobalTokenFromLocal;
093: 
094:     /// @notice Global Address -> ChainId -> Local Address
095:     mapping(address globalAddress => mapping(uint256 chainId => address localAddress)) public getLocalTokenFromGlobal;
096: 
097:     /// @notice Underlying Address -> ChainId -> Local Address
098:     mapping(address underlyingAddress => mapping(uint256 chainId => address localAddress)) public
099:         getLocalTokenFromUnderlying;
100: 
101:     /// @notice Mapping from Local Address to Underlying Address.
102:     mapping(address localAddress => mapping(uint256 chainId => address underlyingAddress)) public
103:         getUnderlyingTokenFromLocal;
104: 
```

## [L-03] Do not hardcode `_zroPaymentAddress` field to address(0) to allow future ZRO fee payments and prevent Bridge Agents from falling apart incase LayerZero makes breaking changes

Hardcoding the [_zroPaymentAddress](https://github.com/LayerZero-Labs/LayerZero/blob/43ab0aed0fbcd123bcac3d089e74898e25b86c0a/contracts/interfaces/ILayerZeroEndpoint.sol#L13) field to address(0) disallows the protocol from using ZRO token as a fee payment option in the future (ZRO might be launching in the coming year). Consider passing the _zroPaymentAddress field as an input parameter to allow flexibility of future fee payments using ZRO tokens.

We can also see point 5 in this [integration checklist](https://layerzero.gitbook.io/docs/evm-guides/layerzero-integration-checklist) provided by the LayerZero team to ensure maximum flexibility in fee payment options is achieved. Here is the point:
```
Do not hardcode address zero (address(0)) as zroPaymentAddress when estimating fees and sending messages. Pass it as a parameter instead.
```

Check out this recent discussion between the 10xKelly and I on hardcoding _zroPaymentAddress (Note: In our case, the **contracts that would be difficult to handle changes or updates are the BridgeAgent contracts**):
Check out this transcript in case image fails to render - https://tickettool.xyz/direct?url=https://cdn.discordapp.com/attachments/1155911737397223496/1155911741725757531/transcript-tx-mrpotatomagic.html

![](https://ipfs.filebase.io/ipfs/QmdaySs3uaoiegvF37Nx2U4WmMe5JXZJ8oJa8PMo3n5riH)

There are 2 instances of this issue:

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L828

This is even more important in our contracts since function [_performCall()](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L808) is the exit point for most cross-chain calls being made from the RootBridgeAgent.sol. Thus, if any updates are made from the LayerZero team, there are chances of the protocol core functionality breaking down.
```solidity
File: src/RootBridgeAgent.sol
823:             ILayerZeroEndpoint(lzEndpointAddress).send{value: msg.value}(
824:                 _dstChainId,
825:                 getBranchBridgeAgentPath[_dstChainId],
826:                 _payload,
827:                 _refundee,
828:                 address(0),
829:                 abi.encodePacked(uint16(2), _gParams.gasLimit, _gParams.remoteBranchExecutionGas, callee)
830:             );
```

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L775

Check Line 778:
```solidity
File: BranchBridgeAgent.sol
768:     function _performCall(address payable _refundee, bytes memory _payload, GasParams calldata _gParams)
769:         internal
770:         virtual
771:     {
772:         //Sends message to LayerZero messaging layer
773:         ILayerZeroEndpoint(lzEndpointAddress).send{value: msg.value}(
774:             rootChainId,
775:             rootBridgeAgentPath,
776:             _payload,
777:             payable(_refundee),
778:             address(0),//@audit Integration issue - Do not hardcode address 0 as it may cause future upgrades difficulty
779:             abi.encodePacked(uint16(2), _gParams.gasLimit, _gParams.remoteBranchExecutionGas, rootBridgeAgentAddress)
780:         );
781:     }
```

## [L-04] Do not hardcode `_payInZRO` field to false to allow future ZRO fee payment estimation for payloads

Hardcoding the [_payInZRO](https://github.com/LayerZero-Labs/LayerZero/blob/43ab0aed0fbcd123bcac3d089e74898e25b86c0a/contracts/interfaces/ILayerZeroEndpoint.sol#L39C15-L39C24) field disallows the protocol from estimating fees when using ZRO tokens as a fee payment option (ZRO might be launching in the coming year). Consider passing the _payInZRO field as an input parameter to allow flexibility of future fee payments using ZRO. **(Note: Although in the docs [here](https://layerzero.gitbook.io/docs/evm-guides/code-examples/estimating-message-fees) ,they've mentioned to set _payInZRO to false, it is only temporarily to avoid incorrect fee estimations. Providing _payInZRO as an input parameter does not affect this since bool value by default is false)**

We can also see point 6 in this [integration checklist](https://layerzero.gitbook.io/docs/evm-guides/layerzero-integration-checklist) provided by the LayerZero team to ensure maximum flexibility in fee payment options is achieved. Here is the point (useZro is now changed to _payInZRO):
```
Do not hardcode useZro to false when estimating fees and sending messages. Pass it as a parameter instead.
```

There are 2 instances of this issue:

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L150

Check Line 154 below:
```solidity
File: RootBridgeAgent.sol
144:     function getFeeEstimate(
145:         uint256 _gasLimit,
146:         uint256 _remoteBranchExecutionGas,
147:         bytes calldata _payload,
148:         uint16 _dstChainId
149:     ) external view returns (uint256 _fee) {
150:         (_fee,) = ILayerZeroEndpoint(lzEndpointAddress).estimateFees(
151:             _dstChainId,
152:             address(this),
153:             _payload,
154:             false, //@audit Low - Do not hardcode this to false, instead pass it as parameter to allow future payments using ZRO
155:             abi.encodePacked(uint16(2), _gasLimit, _remoteBranchExecutionGas, getBranchBridgeAgent[_dstChainId])
156:         );
157:     }
```

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L161C1-L173C6

Check Line 172 below:
```solidity
File: BranchBridgeAgent.sol
162:     /// @inheritdoc IBranchBridgeAgent
163:     function getFeeEstimate(uint256 _gasLimit, uint256 _remoteBranchExecutionGas, bytes calldata _payload)
164:         external
165:         view
166:         returns (uint256 _fee)
167:     {
168:         (_fee,) = ILayerZeroEndpoint(lzEndpointAddress).estimateFees(
169:             rootChainId,
170:             address(this),
171:             _payload,
172:             false,//@audit Low - do not set this to false 
173:             abi.encodePacked(uint16(2), _gasLimit, _remoteBranchExecutionGas, rootBridgeAgentAddress)
174:         );
175:     }
```

## [L-05] Leave some degree of configurability for extra parameters in `_adapterParams` to allow for feature extensions

As recommended by LayerZero [here on the last line of Message Adapter Parameters para](https://layerzero.gitbook.io/docs/faq/messaging-properties#message-adapter-parameters), the team should leave some degree of configurability when packing various variables into [_adapterParams](https://github.com/LayerZero-Labs/LayerZero/blob/43ab0aed0fbcd123bcac3d089e74898e25b86c0a/contracts/interfaces/ILayerZeroEndpoint.sol#L14). This can allow the Maia team to support feature extensions that might be provided by the LayerZero team in the future.

There are 2 instances of this:

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L829C1-L829C106

Below Line 829 represents the _adapterParams. Leaving an extra `bytes calldata param` field when packing the variables using abi.encodePacked can help support any feature extensions by LayerZero in the future.
```solidity
File: src/RootBridgeAgent.sol
823:             ILayerZeroEndpoint(lzEndpointAddress).send{value: msg.value}(
824:                 _dstChainId,
825:                 getBranchBridgeAgentPath[_dstChainId],
826:                 _payload,
827:                 _refundee,
828:                 address(0),
829:                 abi.encodePacked(uint16(2), _gParams.gasLimit, _gParams.remoteBranchExecutionGas, callee)
830:             );
```

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L776

Check Line 779:
```solidity
File: BranchBridgeAgent.sol
773:         ILayerZeroEndpoint(lzEndpointAddress).send{value: msg.value}(
774:             rootChainId,
775:             rootBridgeAgentPath,
776:             _payload,
777:             payable(_refundee),
778:             address(0),
779:             abi.encodePacked(uint16(2), _gParams.gasLimit, _gParams.remoteBranchExecutionGas, rootBridgeAgentAddress)
780:         );
781:     }
```

## [L-06] Do not hardcode LayerZero's proprietary chainIds

As stated by LayerZero [here](https://layerzero.gitbook.io/docs/technical-reference/mainnet/supported-chain-ids):
```
ChainId values are not related to EVM ids. Since LayerZero will span EVM & non-EVM chains the chainId are proprietary to our Endpoints.
```

Since the chainIds are proprietary, they are subject to change. As recommended by LayerZero on point 4 [here](https://layerzero.gitbook.io/docs/evm-guides/layerzero-integration-checklist), use admin restricted setters for changing these chainIds.

Additionally, in the current Maia contracts most chainIds have been marked immutable. If LayerZero does change the chainIds, migrating to a new version would be quite cumbersome all because of this trivial chainId problem (if not handled).

There are 2 instances of this issue (Note: Most bridgeAgent and Port contracts have this issue as well but I have not mentioned them here explicitly):

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L40

As we can see below, currently the chainId is immutable. Consider removing immutable to ensure future chainId changes compatibility.
```solidity
File: src/RootBridgeAgent.sol
40: uint16 public immutable localChainId;
```

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/CoreRootRouter.sol#L47

As we can see below, currently the chainId is immutable, Consider removing immutable to ensure future chainId changes compatibility.
```solidity
File: src/CoreRootRouter.sol
46:     /// @notice Root Chain Layer Zero Identifier.
47:     uint256 public immutable rootChainId;
```

## [L-07] Array entry not deleted when removing bridge agent

There is 1 instance of this issue:

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L355

The function toggleBridgeAgent() is only called from [_removeBranchBridgeAgent()](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/CoreBranchRouter.sol#L263) in the CoreBranchRouter contract. Thus, the function should delete the _bridgeAgent entry from the bridgeAgents array as well to remove stale state.
```solidity
File: BranchPort.sol
359:     function toggleBridgeAgent(address _bridgeAgent) external override requiresCoreRouter {
360:         isBridgeAgent[_bridgeAgent] = !isBridgeAgent[_bridgeAgent];
361: 
362:         emit BridgeAgentToggled(_bridgeAgent);
363:     }
```

## [L-08] Double entries in `strategyTokens`, `portStrategies`, `bridgeAgents` and `bridgeAgentFactories` arrays are not prevented

There are 4 instances of this issue:

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L362C1-L380C1

### 1. Double entry of a strategy token

Let's look at the execution path of how strategy tokens are managed:

**Root chain to EP:**
manageStrategyToken => callOut => _performCall => send

**EP to Branch chain:**
receivePayload => lzReceive => lzReceiveNonBlocking => _execute => executeNoSettlement (Executor) => executeNoSettlement (Router) => _manageStrategyToken => either toggleStrategyToken or addStrategyToken

As we can see in the chain of calls above, when toggling off a strategy token we reach the function toggleStrategyToken(), which toggles of the token as below:
```solidity
File: BranchPort.sol
380:     function toggleStrategyToken(address _token) external override requiresCoreRouter {
381:         isStrategyToken[_token] = !isStrategyToken[_token];
382: 
383:         emit StrategyTokenToggled(_token);
384:     }
```
Now when we try to toggle it back on, according to the chain of calls we reach the function addStrategyToken(), which does the following:
 - On Line 372, we push the token to strategyTokens again. This is what causes the double entry
 - On Line 373, there is a chance of overwriting the _minimumReservesRatio as well.
```solidity
File: BranchPort.sol
367:     function addStrategyToken(address _token, uint256 _minimumReservesRatio) external override requiresCoreRouter {
368:         if (_minimumReservesRatio >= DIVISIONER || _minimumReservesRatio < MIN_RESERVE_RATIO) {
369:             revert InvalidMinimumReservesRatio();
370:         }
371: 
372:         strategyTokens.push(_token);
373:         getMinimumTokenReserveRatio[_token] = _minimumReservesRatio;
374:         isStrategyToken[_token] = true;
375: 
376:         emit StrategyTokenAdded(_token, _minimumReservesRatio);
377:     }
```

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L382C1-L401C1

### 2. Double entry of a Port Strategy

Let's look at the execution path of how Port Strategies are managed:

**Root chain to EP:**
managePortStrategy() => callOut => _performCall => send

**EP to Branch chain:**
receivePayload => lzReceive => lzReceiveNonBlocking => _execute => executeNoSettlement (Executor) => executeNoSettlement (Router) => _managePortStrategy => either addPortStrategy or togglePortStrategy (excluding updatePortStrategy since not important here)

As we can see in the chain of calls above, when toggling off a port strategy we reach the function togglePortStrategy(), which toggles of the strategy as below:
```solidity
File: BranchPort.sol
402:     function togglePortStrategy(address _portStrategy, address _token) external override requiresCoreRouter {
403:         isPortStrategy[_portStrategy][_token] = !isPortStrategy[_portStrategy][_token];
404: 
405:         emit PortStrategyToggled(_portStrategy, _token);
406:     }
```
Now when we try to toggle it back on, according to the chain of calls we reach the function addPortStrategy(), which does the following:
 - On Line 394, we push the token to portStrategies again. This is what causes the double entry
 - On Line 395, there is a chance of overwriting the _dailyManagementLimit as well.
```solidity
File: BranchPort.sol
388:     function addPortStrategy(address _portStrategy, address _token, uint256 _dailyManagementLimit)
389:         external
390:         override
391:         requiresCoreRouter
392:     {
393:         if (!isStrategyToken[_token]) revert UnrecognizedStrategyToken();
394:         portStrategies.push(_portStrategy);
395:         strategyDailyLimitAmount[_portStrategy][_token] = _dailyManagementLimit;
396:         isPortStrategy[_portStrategy][_token] = true;
397: 
398:         emit PortStrategyAdded(_portStrategy, _token, _dailyManagementLimit);
399:     }
```

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L414C1-L424C6

### 3. Double entry of a Core Branch Bridge Agent

Let's look at the execution path of how a Core Branch Router is set:

**Root chain to EP:**
setCoreBranch() => callOut => _performCall => send

**EP to Branch chain:**
receivePayload => lzReceive => lzReceiveNonBlocking => _execute => executeNoSettlement (Executor) => executeNoSettlement (Router) => setCoreBranchRouter (Port)

As we can see in the chain of calls above, when set a Core Branch Router we reach the function setCoreBranchRouter(), which does the following:
 - On Line 425 and 427 respectively, we set the coreBranchRouterAddress to the same address again and push the already existing coreBranchBridgeAgent to the bridgeAgents array again. This is what causes the double entry.
```solidity
File: BranchPort.sol
420:     function setCoreBranchRouter(address _coreBranchRouter, address _coreBranchBridgeAgent)
421:         external
422:         override
423:         requiresCoreRouter
424:     {   //@audit Low - If caller sets coreBranchRouterAddress to the same address again, this can cause a double entry in the bridgeAgents array. To prevent this ensure that the coreBranchRouterAddress cannot be set to the existing address. Although admin error, this check can help prevent double entries 
425:         coreBranchRouterAddress = _coreBranchRouter;
426:         isBridgeAgent[_coreBranchBridgeAgent] = true;
427:         bridgeAgents.push(_coreBranchBridgeAgent);
428: 
429:         emit CoreBranchSet(_coreBranchRouter, _coreBranchBridgeAgent);
430:     }
```
**Mitigation for above (check added on Line 425 to prevent resetting to the same router address again):**
```solidity
File: BranchPort.sol
420:     function setCoreBranchRouter(address _coreBranchRouter, address _coreBranchBridgeAgent)
421:         external
422:         override
423:         requiresCoreRouter
424:     {   
425:         if (coreBranchRouterAddress == _coreBranchRouter) revert SomeError(); 
426:         coreBranchRouterAddress = _coreBranchRouter;
427:         isBridgeAgent[_coreBranchBridgeAgent] = true;
428:         bridgeAgents.push(_coreBranchBridgeAgent);
429: 
430:         emit CoreBranchSet(_coreBranchRouter, _coreBranchBridgeAgent);
431:     }
```

### 4. Double entry of a Bridge Agent Factory

Let's look at the execution path of how Bridge Agent Factories are managed:

**Root chain to EP:**
toggleBranchBridgeAgentFactory() => callOut => _performCall => send

**EP to Branch chain:**
receivePayload => lzReceive => lzReceiveNonBlocking => _execute => executeNoSettlement (Executor) => executeNoSettlement (Router) => _toggleBranchBridgeAgentFactory => either toggleBridgeAgentFactory or addBridgeAgentFactory

As we can see in the chain of calls above, when toggling off a branch bridge agent factory we reach the function toggleBridgeAgentFactory(), which toggles of the bridge agent factory as below:
```solidity
File: BranchPort.sol
348:     function toggleBridgeAgentFactory(address _newBridgeAgentFactory) external override requiresCoreRouter {
349:         isBridgeAgentFactory[_newBridgeAgentFactory] = !isBridgeAgentFactory[_newBridgeAgentFactory];
350: 
351:         emit BridgeAgentFactoryToggled(_newBridgeAgentFactory);
352:     }

```
Now when we try to toggle it back on, according to the chain of calls we reach the function addBridgeAgentFactory(), which does the following:
 - On Line 342, we push the token to bridgeAgentFactories array again. This is what causes the double entry
```solidity
File: BranchPort.sol
338:     function addBridgeAgentFactory(address _newBridgeAgentFactory) external override requiresCoreRouter {
339:         if (isBridgeAgentFactory[_newBridgeAgentFactory]) revert AlreadyAddedBridgeAgentFactory();
340: 
341:         isBridgeAgentFactory[_newBridgeAgentFactory] = true;
342:         bridgeAgentFactories.push(_newBridgeAgentFactory);
343: 
344:         emit BridgeAgentFactoryAdded(_newBridgeAgentFactory);
345:     }

```