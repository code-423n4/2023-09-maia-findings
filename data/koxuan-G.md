# Report


## Gas Optimizations


| |Issue|Instances|
|-|:-|:-:|
| [GAS-1](#GAS-1) | Cache array length outside of loop | 8 |
| [GAS-2](#GAS-2) | Replace ERC721 with ERC721A | 1 |
| [GAS-3](#GAS-3) | Long revert strings | 16 |
| [GAS-4](#GAS-4) | Use mappings over arrays | 50 |
### <a name="GAS-1"></a>[GAS-1] Cache array length outside of loop
If not cached, the solidity compiler will always read the length of the array during each iteration. That is, if it is a storage array, this is an extra sload operation (100 additional extra gas for each iteration except for the first) and if it is a memory array, this is an extra mload operation (3 additional gas for each iteration except for the first).

*Instances (8)*:
```solidity
File: BaseBranchRouter.sol

192:         for (uint256 i = 0; i < _hTokens.length;) {

```

```solidity
File: BranchBridgeAgent.sol

447:         for (uint256 i = 0; i < deposit.tokens.length;) {

```

```solidity
File: MulticallRootRouter.sol

278:             for (uint256 i = 0; i < outputParams.outputTokens.length;) {

367:             for (uint256 i = 0; i < outputParams.outputTokens.length;) {

455:             for (uint256 i = 0; i < outputParams.outputTokens.length;) {

557:         for (uint256 i = 0; i < outputTokens.length;) {

```

```solidity
File: RootBridgeAgent.sol

318:         for (uint256 i = 0; i < settlement.hTokens.length;) {

1070:         for (uint256 i = 0; i < hTokens.length;) {

```

### <a name="GAS-2"></a>[GAS-2] Replace ERC721 with ERC721A
ERC721A is more gas efficient when minting a lot of NFTs at the same time. Consider replacing ERC721A with ERC721.

*Instances (1)*:
```solidity
File: VirtualAccount.sol

7: import {ERC721} from "solmate/tokens/ERC721.sol";

```

### <a name="GAS-3"></a>[GAS-3] Long revert strings

*Instances (16)*:
```solidity
File: BranchBridgeAgent.sol

125:         require(_rootBridgeAgentAddress != address(0), "Root Bridge Agent Address cannot be the zero address.");

130:         require(_localRouterAddress != address(0), "Local Router Address cannot be the zero address.");

131:         require(_localPortAddress != address(0), "Local Port Address cannot be the zero address.");

```

```solidity
File: BranchPort.sol

127:         require(_bridgeAgentFactory != address(0), "BridgeAgentFactory is zero address");

```

```solidity
File: RootBridgeAgent.sol

111:         require(_lzEndpointAddress != address(0), "Layerzero Enpoint Address cannot be zero address");

112:         require(_localPortAddress != address(0), "Port Address cannot be zero address");

113:         require(_localRouterAddress != address(0), "Router Address cannot be zero address");

```

```solidity
File: RootPort.sol

130:         require(_bridgeAgentFactory != address(0), "Bridge Agent Factory cannot be 0 address.");

131:         require(_coreRootRouter != address(0), "Core Root Router cannot be 0 address.");

152:         require(_coreRootBridgeAgent != address(0), "Core Root Bridge Agent cannot be 0 address.");

153:         require(_coreLocalBranchBridgeAgent != address(0), "Core Local Branch Bridge Agent cannot be 0 address.");

154:         require(_localBranchPortAddress != address(0), "Local Branch Port Address cannot be 0 address.");

```

```solidity
File: factories/ArbitrumBranchBridgeAgentFactory.sol

57:         require(_coreRootBridgeAgent != address(0), "Core Root Bridge Agent Address cannot be 0");

```

```solidity
File: factories/BranchBridgeAgentFactory.sol

61:         require(_rootBridgeAgentFactoryAddress != address(0), "Root Bridge Agent Factory Address cannot be 0");

66:         require(_localCoreBranchRouterAddress != address(0), "Core Branch Router Address cannot be 0");

88:         require(_coreRootBridgeAgent != address(0), "Core Root Bridge Agent cannot be 0");

```

### <a name="GAS-4"></a>[GAS-4] Use mappings over arrays
Arrays uses more gas than mappings. Consider using mappings whenever possible.

*Instances (50)*:
```solidity
File: BaseBranchRouter.sol

187:         address[] memory _hTokens,

188:         address[] memory _tokens,

```

```solidity
File: BranchBridgeAgent.sol

507:         address[] memory _hTokens = new address[](numOfAssets);

507:         address[] memory _hTokens = new address[](numOfAssets);

508:         address[] memory _tokens = new address[](numOfAssets);

508:         address[] memory _tokens = new address[](numOfAssets);

827:         address[] memory addressArray = new address[](1);

827:         address[] memory addressArray = new address[](1);

863:         address[] memory _hTokens,

864:         address[] memory _tokens,

```

```solidity
File: BranchPort.sol

35:     address[] public bridgeAgents;

45:     address[] public bridgeAgentFactories;

55:     address[] public strategyTokens;

71:     address[] public portStrategies;

248:         address[] memory _localAddresses,

249:         address[] memory _underlyingAddresses,

290:         address[] memory _localAddresses,

291:         address[] memory _underlyingAddresses,

```

```solidity
File: MulticallRootRouter.sol

32:     address[] outputTokens;

547:         address[] memory outputTokens,

```

```solidity
File: RootBridgeAgent.sol

858:         address[] memory _hTokens,

859:         address[] memory _tokens,

1007:         address[] memory addressArray = new address[](1);

1007:         address[] memory addressArray = new address[](1);

1050:         address[] memory _globalAddresses,

1067:         address[] memory hTokens = new address[](_globalAddresses.length);

1067:         address[] memory hTokens = new address[](_globalAddresses.length);

1068:         address[] memory tokens = new address[](_globalAddresses.length);

1068:         address[] memory tokens = new address[](_globalAddresses.length);

```

```solidity
File: RootBridgeAgentExecutor.sol

276:         address[] memory hTokens = new address[](numOfAssets);

276:         address[] memory hTokens = new address[](numOfAssets);

277:         address[] memory tokens = new address[](numOfAssets);

277:         address[] memory tokens = new address[](numOfAssets);

```

```solidity
File: RootPort.sol

67:     address[] public bridgeAgents;

80:     address[] public bridgeAgentFactories;

```

```solidity
File: interfaces/BridgeAgentStructs.sol

16:     address[] hTokens;

17:     address[] tokens;

32:     address[] hTokens; //Input Local hTokens Address.

33:     address[] tokens; //Input Native / underlying Token Address.

42:     address[] hTokens; //Input Local hTokens Address.

43:     address[] tokens; //Input Native / underlying Token Address.

62:     address[] hTokens; //Input Local hTokens Addresses.

63:     address[] tokens; //Input Native / underlying Token Addresses.

75:     address[] globalAddresses; //Input Global hTokens Addresses.

93:     address[] hTokens; // Input Local hTokens Addresses.

94:     address[] tokens; // Input Native / underlying Token Addresses.

```

```solidity
File: interfaces/IBranchPort.sol

105:         address[] memory _localAddresses,

106:         address[] memory _underlyingAddresses,

137:         address[] memory _localAddresses,

138:         address[] memory _underlyingAddresses,

```

