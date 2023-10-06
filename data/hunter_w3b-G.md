# Gas Optimization

Gas optimization techniques are critical in smart contract development to reduce transaction costs and make contracts more efficient. The Maia DAO - Ulysses protocol, as observed in the provided contract, can benefit from various gas-saving strategies to enhance its performance. Below is a summary of gas optimization techniques followed by categorized findings within the contracts.

# Summary

| Finding                                                                               | Occurrences |
| :------------------------------------------------------------------------------------ | :---------: |
| Use mappings instead of arrays                                                        |     42      |
| Using Storage instead of memory for structs/arrays saves gas                          |     14      |
| Use assembly to write address storage values                                          |     40      |
| A modifier used only once and not being inherited should be inlined to save gas       |     12      |
| ERC1155 is a cheaper non-fungible token than ERC721                                   |      3      |
| Use one ERC1155 or ERC6909 token instead of several ERC20 tokens                      |      9      |
| Using assembly to revert with an error message                                        |     177     |
| Split revert statements                                                               |      1      |
| Understand the trade-offs when choosing between internal functions and modifiers      |     27      |
| Using > 0 costs more gas than != 0                                                    |     17      |
| Do-While loops are cheaper than for loops                                             |     15      |
| Use assembly to reuse memory space when making more than one external call            |     44      |
| Missing Zero address checks in the constructor                                        |     14      |
| Short-circuit booleans                                                                |      4      |
| 15 Pre-increment and pre-decrement are cheaper than +1 ,-1                            |      4      |
| Use hardcode address instead address(this)                                            |     33      |
| Use bytes.concat() instead of abi.encodePacked(), since this is preferred since 0.8.4 |     46      |
| Shorten the array rather than copying to a new one                                    |     14      |
| Initializers can be marked as payable to save deployment gas                          |      9      |
| Use assembly to validate msg.sender                                                   |     22      |
| Use selfdestruct in the constructor if the contract is one-time use                   |             |
| Avoid contract calls by making the architecture monolithic                            |      8      |
| It is sometimes cheaper to cache calldata                                             |      7      |
| Use ++i instead of i++ to increment                                                   |      4      |
| Always use Named Returns                                                              |     16      |
| Use bitmaps instead of bools when a significant amount of booleans are used           |     10      |
| Empty blocks should be removed or emit something                                      |      5      |

## [G-01] Use mappings instead of arrays

Using mappings instead of arrays to avoid length checks

When storing a list or group of items that you wish to organize in a specific order and fetch with a fixed key/index, it’s common practice to use an array data structure. This works well, but did you know that a trick can be implemented to save 2,000+ gas on each read using a mapping?

See the example below

```solidity


/// get(0) gas cost: 4860
contract Array {
uint256[] a;

    constructor() {
        a.push() = 1;
        a.push() = 2;
        a.push() = 3;
    }

    function get(uint256 index) external view returns(uint256) {
        return a[index];
    }

}

/// get(0) gas cost: 2758
contract Mapping {
mapping(uint256 => uint256) a;

    constructor() {
        a[0] = 1;
        a[1] = 2;
        a[2] = 3;
    }

    function get(uint256 index) external view returns(uint256) {
        return a[index];
    }

}
```

Just by using a mapping, we get a gas saving of 2102 gas. Why? Under the hood when you read the value of an index of an array, solidity adds bytecode that checks that you are reading from a valid index (i.e an index strictly less than the length of the array) else it reverts with a panic error (Panic(0x32) to be precise). This prevents from reading unallocated or worse, allocated storage/memory locations.

Due to the way mappings are (simply a key => value pair), no check like that exists and we are able to read from the a storage slot directly. It’s important to note that when using mappings in this manner, your code should ensure that you are not reading an out of bound index of your canonical array.

```solidity
File: src/BranchPort.sol

35    address[] public bridgeAgents;

45    address[] public bridgeAgentFactories;

55    address[] public strategyTokens;

71    address[] public portStrategies;

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L35

```solidity
File: src/MulticallRootRouter.sol

32    address[] outputTokens;

34    uint256[] amountsOut;

36    uint256[] depositsOut;
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L32

```solidity
File: src/RootPort.sol

67    address[] public bridgeAgents;

80    address[] public bridgeAgentFactories;

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L67

```solidity
File: src/factories/ERC20hTokenBranchFactory.sol

23    ERC20hTokenBranch[] public hTokens;
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenBranchFactory.sol#L23

```solidity
File: src/factories/ERC20hTokenRootFactory.sol

23    ERC20hTokenRoot[] public hTokens;
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenRootFactory.sol#L23

```solidity
File: src/BranchBridgeAgent.sol

507        address[] memory _hTokens = new address[](numOfAssets);
508        address[] memory _tokens = new address[](numOfAssets);
509        uint256[] memory _amounts = new uint256[](numOfAssets);
510        uint256[] memory _deposits = new uint256[](numOfAssets);


827        address[] memory addressArray = new address[](1);
828        uint256[] memory uintArray = new uint256[](1);
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L507-L510

```solidity
File: src/interfaces/BridgeAgentStructs.sol

16    address[] hTokens;
17    address[] tokens;
18    uint256[] amounts;
19    uint256[] deposits;

32    address[] hTokens; //Input Local hTokens Address.
33    address[] tokens; //Input Native / underlying Token Address.
34    uint256[] amounts; //Amount of Local hTokens deposited for interaction.
35    uint256[] deposits; //Amount of native tokens deposited for interaction.

42    address[] hTokens; //Input Local hTokens Address.
43    address[] tokens; //Input Native / underlying Token Address.
44    uint256[] amounts; //Amount of Local hTokens deposited for interaction.
45    uint256[] deposits; //Amount of native tokens deposited for interaction.


62    address[] hTokens; //Input Local hTokens Addresses.
63    address[] tokens; //Input Native / underlying Token Addresses.
64    uint256[] amounts; //Amount of Local hTokens deposited for interaction.
65    uint256[] deposits; //Amount of native tokens deposited for interaction.

75    address[] globalAddresses; //Input Global hTokens Addresses.
76    uint256[] amounts; //Amount of Local hTokens deposited for interaction.
77    uint256[] deposits; //Amount of native tokens deposited for interaction.

93    address[] hTokens; // Input Local hTokens Addresses.
94    address[] tokens; // Input Native / underlying Token Addresses.
95    uint256[] amounts; // Amount of Local hTokens deposited for interaction.
96    uint256[] deposits; // Amount of native tokens deposited for interaction.
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/BridgeAgentStructs.sol#L16-L19

```solidity
File: src/RootBridgeAgent.sol

1007        address[] memory addressArray = new address[](1);
1008        uint256[] memory uintArray = new uint256[](1);

1067        address[] memory hTokens = new address[](_globalAddresses.length);
1068        address[] memory tokens = new address[](_globalAddresses.length);
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1007

```solidity
File: src/RootBridgeAgentExecutor.sol

276        address[] memory hTokens = new address[](numOfAssets);
277        address[] memory tokens = new address[](numOfAssets);
278        uint256[] memory amounts = new uint256[](numOfAssets);
279        uint256[] memory deposits = new uint256[](numOfAssets);
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L276C1-L279C64

## [G-02] Using Storage instead of memory for structs/arrays saves gas

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from
storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array.

If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read.

Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to
be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read.

The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

```solidity
File:  src/RootBridgeAgent.s

1007        address[] memory addressArray = new address[](1);
1008        uint256[] memory uintArray = new uint256[](1);

1067        address[] memory hTokens = new address[](_globalAddresses.length);
1068        address[] memory tokens = new address[](_globalAddresses.length);
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1007

```solidity
File: src/RootBridgeAgentExecutor.sol

276        address[] memory hTokens = new address[](numOfAssets);
277        address[] memory tokens = new address[](numOfAssets);
278        uint256[] memory amounts = new uint256[](numOfAssets);
279        uint256[] memory deposits = new uint256[](numOfAssets);
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L276C1-L279C64

```solidity
File: src/BranchBridgeAgent.sol

507        address[] memory _hTokens = new address[](numOfAssets);
508        address[] memory _tokens = new address[](numOfAssets);
509        uint256[] memory _amounts = new uint256[](numOfAssets);
510        uint256[] memory _deposits = new uint256[](numOfAssets);


827        address[] memory addressArray = new address[](1);
828        uint256[] memory uintArray = new uint256[](1);
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L507-L510

## [G-03] Use assembly to write address storage values

```solidity
File: src/ArbitrumBranchPort.sol

26    address public immutable rootPortAddress;

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol#L26

```solidity
File: src/MulticallRootRouter.sol

65    uint256 public immutable localChainId;

68    address public immutable localPortAddress;

71    address public immutable multicallAddress;

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L65-L71

```solidity
File: src/CoreRootRouter.sol

51    address public immutable rootPortAddress;

    /// @notice Bridge Agent to manage remote execution and cross-chain assets.
54    address payable public bridgeAgentAddress;

    /// @notice Bridge Agent Executor Address.
57    address public bridgeAgentExecutorAddress;

    /// @notice ERC20 hToken Root Factory Address.
60    address public hTokenFactoryAddress;
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L51C1-L60C41

```solidity
File: src/CoreBranchRouter.sol

31        hTokenFactoryAddress = _hTokenFactoryAddress;
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L31

```solidity
File: src/RootBridgeAgent.sol

40    uint16 public immutable localChainId;

    /// @notice Bridge Agent Factory Address.
43    address public immutable factoryAddress;

    /// @notice Local Core Root Router Address
46    address public immutable localRouterAddress;

    /// @notice Local Port Address where funds deposited from this chain are stored.
49    address public immutable localPortAddress;

    /// @notice Local Layer Zero Endpoint Address for cross-chain communication.
52    address public immutable lzEndpointAddress;

    /// @notice Address of Root Bridge Agent Executor.
55    address public immutable bridgeAgentExecutorAddress;
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L40C1-L55C57

```solidity
File: src/VirtualAccount.sol

21    address public immutable override userAddress;

24    address public immutable override localPortAddress;

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L21

```solidity
File:  src/BranchPort.sol

25    address public coreBranchRouterAddress;

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L25

```solidity
File: src/BranchBridgeAgent.sol

60    address public immutable rootBridgeAgentAddress;

67    address public immutable lzEndpointAddress;

70    address public immutable localRouterAddress;

74    address public immutable localPortAddress;

77    address public immutable bridgeAgentExecutorAddress;

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L60

```solidity
File: src/BaseBranchRouter.sol

33    address public localPortAddress;

36    address public override localBridgeAgentAddress;

39    address public override bridgeAgentExecutorAddress;
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L33

```solidity
File: src/RootPort.sol

37    address public localBranchPortAddress;

40    address public coreRootRouterAddress;

43    address public coreRootBridgeAgentAddress;
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L37

```solidity
File: src/token/ERC20hTokenRoot.sol

17    address public immutable override factoryAddress;

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/token/ERC20hTokenRoot.sol#L17

```solidity
File: src/factories/BranchBridgeAgentFactory.sol

27    address public immutable rootBridgeAgentFactoryAddress;

    /// @notice Local Core Branch Router Address.
30    address public immutable localCoreBranchRouterAddress;

    /// @notice Root Port Address.
33    address public immutable localPortAddress;

    /// @notice Local Layer Zero Endpoint for cross-chain communication.
36    address public immutable lzEndpointAddress;
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/BranchBridgeAgentFactory.sol#L27

```solidity
File: src/factories/RootBridgeAgentFactory.sol

16    address public immutable rootPortAddress;

19    address public immutable lzEndpointAddress;

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/RootBridgeAgentFactory.sol#L16

```solidity
File: src/factories/ERC20hTokenBranchFactory.sol

17    address public immutable localPortAddress;

20    address public immutable localCoreRouterAddress;

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenBranchFactory.sol#L17

```solidity
File: src/factories/ERC20hTokenRootFactory.sol

17    address public immutable rootPortAddress;

20    address public immutable coreRootRouterAddress;

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenRootFactory.sol

## [G-04] A modifier used only once and not being inherited should be inlined to save gas

### This `requiresEndpoint` modifier is only used in this function `lzReceiveNonBlocking` it should be save some gas that inlined.

```solidity
File: src/BranchBridgeAgent.sol

930    modifier requiresEndpoint(address _endpoint, bytes calldata _srcAddress) {
931            _requiresEndpoint(_endpoint, _srcAddress);
932            _;
933        }



947    modifier requiresRouter() {
948        if (msg.sender != localRouterAddress) revert UnrecognizedRouter();
949        _;
950    }
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L930-L934

### This `requiresBridgeAgentFactory` modifier is only used in this function `addBridgeAgent` it should be save some gas that inlined.

```solidity
File: src/BranchPort.sol

553    modifier requiresBridgeAgentFactory() {
554        if (!isBridgeAgentFactory[msg.sender]) revert UnrecognizedBridgeAgentFactory();
555        _;
556    }


559    modifier requiresPortStrategy(address _token) {
560        if (!isStrategyToken[_token]) revert UnrecognizedStrategyToken();
561        if (!isPortStrategy[msg.sender][_token]) revert UnrecognizedPortStrategy();
562        _;
563    }
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L553C1-L556C6

```solidity
File: src/RootBridgeAgent.sol

1204    modifier requiresEndpoint(address _endpoint, uint16 _srcChain, bytes calldata _srcAddress) virtual {


1226    modifier requiresPort() {


1232    modifier requiresManager() {
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1204

```solidity
File: src/RootPort.sol

557    modifier requiresBridgeAgentFactory() {

563    modifier requiresBridgeAgent() {

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L557

```solidity
File: src/factories/ERC20hTokenBranchFactory.sol

112    modifier requiresCoreRouter() {
113        if (msg.sender != localCoreRouterAddress) revert UnrecognizedCoreRouter();
114        _;
115    }
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenBranchFactory.sol#L112C14-L112C32

```solidity
File: src/factories/ERC20hTokenRootFactory.sol

96    modifier requiresCoreRouterOrPort() {
97        if (msg.sender != coreRootRouterAddress) {
98            if (msg.sender != rootPortAddress) {
99                revert UnrecognizedCoreRouterOrPort();
100            }
101        }
102        _;
103    }
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenRootFactory.sol#L96

## [G-05] ERC1155 is a cheaper non-fungible token than ERC721

The ERC721 balanceOf function is rarely used in practice but adds a storage overhead whenever a mint and transfer happens. ERC1155 tracks balance per id, and also uses the same balance to track ownership of the id. If the maximum supply for each id is one, then the token becomes non-fungible per each id.

```solidity
File: src/VirtualAccount.sol

7   import {ERC721} from "solmate/tokens/ERC721.sol";

11  import {IERC721Receiver} from "@openzeppelin/contracts/token/ERC721/IERC721Receiver.sol";

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L7

```solidity
File: src/interfaces/IVirtualAccount.sol

4   import {IERC721Receiver} from "@openzeppelin/contracts/token/ERC721/IERC721Receiver.sol";

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/IVirtualAccount.sol#L4

## [G-06] Use one ERC1155 or ERC6909 token instead of several ERC20 tokens

This was the original intent of the ERC1155 token. Each individual token behaves like and ERC20, but only one contract needs to be deployed.

The drawback of this approach is that the tokens will not be compatible with most DeFi swapping primitives.

ERC1155 uses callbacks on all of the transfer methods. If this is not desired, ERC6909 can be used instead.

```solidity
File: src/ArbitrumCoreBranchRouter.sol

4   import {ERC20} from "solmate/tokens/ERC20.sol";

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumCoreBranchRouter.sol#L4

```solidity
File: src/CoreRootRouter.sol

6   import {ERC20} from "solmate/tokens/ERC20.sol";

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L6

```solidity
File: src/CoreBranchRouter.sol

4   import {ERC20} from "solmate/tokens/ERC20.sol";

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L4

```solidity
File: src/BranchPort.sol

8   import {ERC20} from "solmate/tokens/ERC20.sol";
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L8

```solidity
File: src/BaseBranchRouter.sol

7   import {ERC20} from "solmate/tokens/ERC20.sol";
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L7

```solidity
File: src/token/ERC20hTokenRoot.sol

6   import {ERC20} from "solmate/tokens/ERC20.sol";
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/token/ERC20hTokenRoot.sol#L6

```solidity
File: src/token/ERC20hTokenBranch.sol

6   import {ERC20} from "solmate/tokens/ERC20.sol";
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/token/ERC20hTokenBranch.sol#L6

```solidity
File: src/factories/ERC20hTokenBranchFactory.sol

6   import {ERC20} from "../token/ERC20hTokenBranch.sol";

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenBranchFactory.sol#L6

```solidity
File: src/factories/ERC20hTokenRootFactory.sol

6   import {ERC20} from "solmate/tokens/ERC20.sol";
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenRootFactory.sol#L6C1-L7C1

## [G-07] Using assembly to revert with an error message

When reverting in solidity code, it is common practice to use a require or revert statement to revert execution with an error message. This can in most cases be further optimized by using assembly to revert with the error message.

Here’s an example;

```solidity

/// calling restrictedAction(2) with a non-owner address: 24042
contract SolidityRevert {
    address owner;
    uint256 specialNumber = 1;

    constructor() {
        owner = msg.sender;
    }

    function restrictedAction(uint256 num) external {
        require(owner == msg.sender, "caller is not owner");
        specialNumber = num;
    }
}

/// calling restrictedAction(2) with a non-owner address: 23734
    function restrictedAction(uint256 num) external {
        assembly {
            if sub(caller(), sload(owner.slot)) {
                mstore(0x00, 0x20) // store offset to where length of revert message is stored
                mstore(0x20, 0x13) // store length (19)
                mstore(
                    0x40,
                    0x63616c6c6572206973206e6f74206f776e657200000000000000000000000000
                ) // store hex representation of message
                revert(0x00, 0x60) // revert with data
            }
        }
        specialNumber = num;
    }
}

```

From the example above we can see that we get a gas saving of over `300` gas when reverting wth the same error message with assembly against doing so in solidity. This gas savings come from the memory expansion costs and extra type checks the solidity compiler does under the hood.

```solidity
File:  src/ArbitrumBranchBridgeAgent.sol

126        if (msg.sender != address(this)) revert LayerZeroUnauthorizedEndpoint();
127        if (_endpoint != rootBridgeAgentAddress) revert LayerZeroUnauthorizedEndpoint();
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchBridgeAgent.sol#L126

```solidity
File: src/ArbitrumBranchPort.sol

39        require(_rootPortAddress != address(0), "Root Port Address cannot be 0");


63        if (_globalToken == address(0)) revert UnknownGlobalToken();

83        if (!IRootPort(_rootPortAddress).isGlobalToken(_globalAddress, localChainId)) revert UnknownGlobalToken();

90        if (_underlyingAddress == address(0)) revert UnknownUnderlyingToken();

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol

```solidity
File: src/ArbitrumCoreBranchRouter.sol

98            revert UnrecognizedBridgeAgentFactory();

108            revert UnrecognizedBridgeAgent();

169            revert UnrecognizedFunctionId();

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumCoreBranchRouter.sol

```solidity
File: src/CoreRootRouter.sol

84        require(_setup, "Contract is already initialized");

113            revert UnauthorizedCallerNotManager();


117        if (!IPort(rootPortAddress).isChainId(_dstChainId)) revert InvalidChainId();

120        if (IBridgeAgent(_rootBridgeAgent).getBranchBridgeAgent(_dstChainId) != address(0)) revert InvalidChainId();

123        if (!IBridgeAgent(_rootBridgeAgent).isBranchBridgeAgentAllowed(_dstChainId)) revert UnauthorizedChainId();

164            revert UnrecognizedBridgeAgentFactory();

278        require(msg.sender == rootPortAddress, "Only root port can call");

327            revert UnrecognizedFunctionId();

245            revert UnrecognizedFunctionId();

412        if (_dstChainId == rootChainId) revert InvalidChainId();

415            revert UnrecognizedGlobalToken();

420            revert TokenAlreadyAdded();

461        if (IPort(rootPortAddress).isGlobalAddress(_underlyingAddress)) revert TokenAlreadyAdded();

462        if (IPort(rootPortAddress).isLocalToken(_underlyingAddress, _srcChainId)) revert TokenAlreadyAdded();

463        if (IPort(rootPortAddress).isUnderlyingToken(_underlyingAddress, _srcChainId)) revert TokenAlreadyAdded();

483        if (IPort(rootPortAddress).isLocalToken(_localAddress, _dstChainId)) revert TokenAlreadyAdded();

512        if (msg.sender != bridgeAgentExecutorAddress) revert UnrecognizedBridgeAgentExecutor();

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L6

```solidity
File: src/CoreBranchRouter.sol

212            revert UnrecognizedBridgeAgentFactory();

222            revert UnrecognizedBridgeAgent();

268        if (!IPort(_localPortAddress).isBridgeAgent(_branchBridgeAgent)) revert UnrecognizedBridgeAgent();

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L212

```solidity
File: src/RootBridgeAgent.sol

244        if (settlementReference.owner == address(0)) revert SettlementRetryUnavailable();

249                revert NotSettlementOwner();

282        if (settlementOwner == address(0)) revert SettlementRetrieveUnavailable();

287                revert NotSettlementOwner();

307        if (settlement.status == STATUS_SUCCESS) revert SettlementRedeemUnavailable();

308        if (settlementOwner == address(0)) revert SettlementRedeemUnavailable();

313                revert NotSettlementOwner();

357        if (_dParams.amount < _dParams.deposit) revert InvalidInputParams();

365                revert InvalidInputParams();

372                revert InvalidInputParams();

396        if (length > MAX_TOKENS_LENGTH) revert InvalidInputParams();

430        if (!success) if (msg.sender == getBranchBridgeAgent[localChainId]) revert ExecutionFailure();

450                revert AlreadyExecutedTransaction();

473                revert AlreadyExecutedTransaction();

496                revert AlreadyExecutedTransaction();

519                revert AlreadyExecutedTransaction();

545                revert AlreadyExecutedTransaction();

583                revert AlreadyExecutedTransaction();

623                revert AlreadyExecutedTransaction();

670            if (settlementReference.owner == address(0)) revert SettlementRetryUnavailable();

675                    revert NotSettlementOwner();

705                revert AlreadyExecutedTransaction();

757        if (!success) revert ExecutionFailure();

818        if (callee == address(0)) revert UnrecognizedBridgeAgent();

871        if (_hTokens.length == 0) revert SettlementRetryUnavailableUseCallout();

910        if (callee == address(0)) revert UnrecognizedBridgeAgent();

1057        if (_globalAddresses.length > MAX_TOKENS_LENGTH) revert InvalidInputParamsLength();

1060        if (_globalAddresses.length != _amounts.length) revert InvalidInputParamsLength();

1061        if (_amounts.length != _deposits.length) revert InvalidInputParamsLength();

1141        if (_amount == 0 && _deposit == 0) revert InvalidInputParams();

1142        if (_amount < _deposit) revert InvalidInputParams();

1145        if (_localAddress == address(0)) revert UnrecognizedLocalAddress();

1146        if (_underlyingAddress == address(0)) if (_deposit > 0) revert UnrecognizedUnderlyingAddress();

1171        if (getBranchBridgeAgent[_branchChainId] != address(0)) revert AlreadyAddedBridgeAgent();

1199        if (msg.sender != localRouterAddress) revert UnrecognizedRouter();

1205        if (msg.sender != address(this)) revert LayerZeroUnauthorizedEndpoint();

1208            if (_endpoint != lzEndpointAddress) revert LayerZeroUnauthorizedEndpoint();

1210            if (_srcAddress.length != 40) revert LayerZeroUnauthorizedCaller();

1221        if (msg.sender != bridgeAgentExecutorAddress) revert UnrecognizedExecutor();


1227        if (msg.sender != localPortAddress) revert UnrecognizedPort();

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L244C1-L245C1

```solidity
File: src/VirtualAccount.sol

76            if (!success) revert CallFailed();

103            if (!success) revert CallFailed();

111        if (msg.value != valAccumulator) revert CallFailed();

163                revert UnauthorizedCaller();

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L76C18-L76C25

```solidity
File: src/BranchPort.sol

109        require(_owner != address(0), "Owner is zero address");

123        require(coreBranchRouterAddress == address(0), "Contract already initialized");

124        require(!isBridgeAgentFactory[_bridgeAgentFactory], "Contract already initialized");

126        require(_coreBranchRouter != address(0), "CoreBranchRouter is zero address");

127        require(_bridgeAgentFactory != address(0), "BridgeAgentFactory is zero address");

181        require(ERC20(_token).balanceOf(address(this)) - currBalance == _amount, "Port Strategy Withdraw Failed");

213        require(

299        if (length > 255) revert InvalidInputArrays();

300        if (length != _underlyingAddresses.length) revert InvalidInputArrays();

301        if (_underlyingAddresses.length != _amounts.length) revert InvalidInputArrays();

302        if (_amounts.length != _deposits.length) revert InvalidInputArrays();

320        if (isBridgeAgent[_bridgeAgent]) revert AlreadyAddedBridgeAgent();

339        if (isBridgeAgentFactory[_newBridgeAgentFactory]) revert AlreadyAddedBridgeAgentFactory();

364            revert InvalidMinimumReservesRatio();

387        if (!isStrategyToken[_token]) revert UnrecognizedStrategyToken();

332        require(coreBranchRouterAddress != address(0), "CoreRouter address is zero");

333        require(_newCoreRouter != address(0), "New CoreRouter address is zero");

542        if (msg.sender != coreBranchRouterAddress) revert UnrecognizedCore();

548        if (!isBridgeAgent[msg.sender]) revert UnrecognizedBridgeAgent();

554        if (!isBridgeAgentFactory[msg.sender]) revert UnrecognizedBridgeAgentFactory();

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L299C1-L302C78

```solidity
File:   src/BranchBridgeAgent.sol

125        require(_rootBridgeAgentAddress != address(0), "Root Bridge Agent Address cannot be the zero address.");

126        require(

130        require(_localRouterAddress != address(0), "Local Router Address cannot be the zero address.");

131        require(_localPortAddress != address(0), "Local Port Address cannot be the zero address.");

354        if (deposit.owner != msg.sender) revert NotDepositOwner();

412        if (payload.length == 0) revert DepositRetryUnavailableUseCallout();

424        if (getDeposit[_depositNonce].owner != msg.sender) revert NotDepositOwner();

439        if (deposit.status == STATUS_SUCCESS) revert DepositRedeemUnavailable();

440        if (deposit.owner == address(0)) revert DepositRedeemUnavailable();

441        if (deposit.owner != msg.sender) revert NotDepositOwner();

604            if (executionState[nonce] != STATUS_READY) revert AlreadyExecutedTransaction();

624            if (executionState[nonce] != STATUS_READY) revert AlreadyExecutedTransaction();

646            if (executionState[nonce] != STATUS_READY) revert AlreadyExecutedTransaction();

672                revert AlreadyExecutedTransaction();

720        if (!success) revert ExecutionFailure();

869        if (_hTokens.length > MAX_TOKENS_LENGTH) revert InvalidInput();

870        if (_hTokens.length != _tokens.length) revert InvalidInput();

871        if (_tokens.length != _amounts.length) revert InvalidInput();

872        if (_amounts.length != _deposits.length) revert InvalidInput();

938        if (msg.sender != address(this)) revert LayerZeroUnauthorizedEndpoint();

939        if (_endpoint != lzEndpointAddress) revert LayerZeroUnauthorizedEndpoint();

942        if (_srcAddress.length != 40) revert LayerZeroUnauthorizedCaller();

943        if (rootBridgeAgentAddress != address(uint160(bytes20(_srcAddress[20:])))) revert LayerZeroUnauthorizedCaller();

948        if (msg.sender != localRouterAddress) revert UnrecognizedRouter();

954        if (msg.sender != bridgeAgentExecutorAddress) revert UnrecognizedBridgeAgentExecutor();

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L354C1-L355C1

```solidity
File: src/RootPort.sol

130        require(_bridgeAgentFactory != address(0), "Bridge Agent Factory cannot be 0 address.");

131        require(_coreRootRouter != address(0), "Core Root Router cannot be 0 address.");

132        require(_setup, "Setup ended.");

152        require(_coreRootBridgeAgent != address(0), "Core Root Bridge Agent cannot be 0 address.");

153        require(_coreLocalBranchBridgeAgent != address(0), "Core Local Branch Bridge Agent cannot be 0 address.");

154        require(_localBranchPortAddress != address(0), "Local Branch Port Address cannot be 0 address.");

155        require(isBridgeAgent[_coreRootBridgeAgent], "Core Bridge Agent doesn't exist.");

156        require(_setupCore, "Core Setup ended.");

245        if (_globalAddress == address(0)) revert InvalidGlobalAddress();

246        if (_localAddress == address(0)) revert InvalidLocalAddress();

247        if (_underlyingAddress == address(0)) revert InvalidUnderlyingAddres

264        if (_localAddress == address(0)) revert InvalidLocalAddress();

282        if (!isGlobalAddress[_hToken]) revert UnrecognizedToken();

290        if (_deposit > 0) if (!ERC20hTokenRoot(_hToken).mint(_recipient, _deposit, _srcChainId)) revert UnableToMint();

309        if (!isGlobalAddress[_hToken]) revert UnrecognizedToken();

320        if (!isGlobalAddress[_hToken]) revert UnrecognizedToken();

330        if (!isGlobalAddress[_hToken]) revert UnrecognizedToken();

341        if (!isGlobalAddress[_hToken]) revert UnrecognizedToken();

342        if (!ERC20hTokenRoot(_hToken).mint(_to, _amount, localChainId)) revert UnableToMint();

360        if (_user == address(0)) revert InvalidUserAddress();

383        if (isBridgeAgent[_bridgeAgent]) revert AlreadyAddedBridgeAgent();

422        if (isBridgeAgentFactory[_bridgeAgentFactory]) revert AlreadyAddedBridgeAgentFactory();

448        if (isChainId[_chainId]) revert AlreadyAddedChain();

485        if (isGlobalAddress[_ecoTokenGlobalAddress]) revert AlreadyAddedEcosystemToken();

510       if (_coreRootRouter == address(0)) revert InvalidCoreRootRouter();

511        if (_coreRootBridgeAgent == address(0)) revert InvalidCoreRootBridgeAgent();

528        if (_coreBranchRouter == address(0)) revert InvalidCoreBranchRouter();

529        if (_coreBranchBridgeAgent == address(0)) revert InvalidCoreBrancBridgeAgent();

544        if (_coreBranchRouter == address(0)) revert InvalidCoreBranchRouter();

545        if (_coreBranchBridgeAgent == address(0)) revert InvalidCoreBrancBridgeAgent();
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L130C1-L132C41

## [G-08] Split revert statements

Similar to splitting require statements, you will usually save some gas by not having a boolean operator in the if statement.

```solidity

contract CustomErrorBoolLessEfficient {
    error BadValue();

    function requireGood(uint256 x) external pure {
        if (x < 10 || x > 20) {
            revert BadValue();
        }
    }
}

contract CustomErrorBoolEfficient {
    error TooLow();
    error TooHigh();

    function requireGood(uint256 x) external pure {
        if (x < 10) {
            revert TooLow();
        }
        if (x > 20) {
            revert TooHigh();
        }
    }
}

```

```solidity
File: src/BranchPort.sol

363        if (_minimumReservesRatio >= DIVISIONER || _minimumReservesRatio < MIN_RESERVE_RATIO) {
364            revert InvalidMinimumReservesRatio();
365        }
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L363C1-L365C10

## [G-09] Understand the trade-offs when choosing between internal functions and modifiers

Modifiers inject its implementation bytecode where it is used while internal functions jump to the location in the runtime code where the its implementation is. This brings certain trade-offs to both options.

Using modifiers more than once means repetitiveness and increase in size of the runtime code but reduces gas cost because of the absence of jumping to the internal function execution offset and jumping back to continue. This means that if runtime gas cost matter most to you, then modifiers should be your choice but if deployment gas cost and/or reducing the size of the creation code is most important to you then using internal functions will be best.

However, modifiers have the tradeoff that they can only be executed at the start or end of a functon. This means executing it at the middle of a function wouldn’t be directly possible, at least not without internal functions which kill the original purpose. This affects it’s flexibility. Internal functions however can be called at any point in a function.

Example showing difference in gas cost using modifiers and an internal function

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.19;

/\*_ deployment gas cost: 195435
gas per call:
restrictedAction1: 28367
restrictedAction2: 28377
restrictedAction3: 28411
_/
contract Modifier {
address owner;
uint256 val;

    constructor() {
        owner = msg.sender;
    }

    modifier onlyOwner() {
        require(msg.sender == owner);
        _;
    }

    function restrictedAction1() external onlyOwner {
        val = 1;
    }

    function restrictedAction2() external onlyOwner {
        val = 2;
    }

    function restrictedAction3() external onlyOwner {
        val = 3;
    }

}

/\*_ deployment gas cost: 159309
gas per call:
restrictedAction1: 28391
restrictedAction2: 28401
restrictedAction3: 28435
_/
contract InternalFunction {
address owner;
uint256 val;

    constructor() {
        owner = msg.sender;
    }

    function onlyOwner() internal view {
        require(msg.sender == owner);
    }

    function restrictedAction1() external {
        onlyOwner();
        val = 1;
    }

    function restrictedAction2() external {
        onlyOwner();
        val = 2;
    }

    function restrictedAction3() external {
        onlyOwner();
        val = 3;
    }

}
```

| Operation          | Deployment | restrictedAction1 | restrictedAction2 | restrictedAction3 |
| ------------------ | ---------- | ----------------- | ----------------- | ----------------- |
| Modifiers          | 195,435    | 28,367            | 28,377            | 28,411            |
| Internal Functions | 159,309    | 28,391            | 28,401            | 28,435            |

From the table above, we can see that the contract that uses modifiers cost more than 35k gas more than the contract using internal functions when deploying it due to repetition of the onlyOwner functionality in 3 functions.

During runtime, we can see that each function using modifiers cost a fixed 24 gas less than the functions using internal functions.

```solidity
File: src/MulticallRootRouter.sol

// @audit  this lock modifier use 4 times
590    modifier lock() {

598    modifier requiresExecutor() {

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L590C1-L591C1

```solidity
File: src/CoreRootRouter.sol

511    modifier requiresExecutor() {

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L511

```solidity
File: src/RootBridgeAgent.sol


1190    modifier lock() {

1198    modifier requiresRouter() {

1204    modifier requiresEndpoint(address _endpoint, uint16 _srcChain, bytes calldata _srcAddress) virtual {

1220    modifier requiresAgentExecutor() {

1226    modifier requiresPort() {

1232    modifier requiresManager() {

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1220

```solidity
File: src/VirtualAccount.sol

160    modifier requiresApprovedCaller() {

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L160

```solidity
File: src/BranchPort.sol

541    modifier requiresCoreRouter() {

547    modifier requiresBridgeAgent() {

553    modifier requiresBridgeAgentFactory() {

559    modifier requiresPortStrategy(address _token) {

566    modifier lock() {

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L553

```solidity
File: src/BranchBridgeAgent.sol

922    modifier lock() {

930    modifier requiresEndpoint(address _endpoint, bytes calldata _srcAddress) {

947    modifier requiresRouter() {

953    modifier requiresAgentExecutor() {

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L922

```solidity
File: src/BaseBranchRouter.sol

206    modifier requiresAgentExecutor() {

212    modifier lock() {

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L206

```solidity
File: src/RootPort.sol

557    modifier requiresBridgeAgentFactory() {

563    modifier requiresBridgeAgent() {

569    modifier requiresCoreRootRouter() {

575    modifier requiresLocalBranchPort() {

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L557

```solidity
File: src/factories/ERC20hTokenBranchFactory.sol


112    modifier requiresCoreRouter() {

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenBranchFactory.sol#L112

```solidity
File: src/factories/ERC20hTokenRootFactory.sol

96    modifier requiresCoreRouterOrPort() {

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenRootFactory.sol#L96

## [G-10] Using > 0 costs more gas than != 0

```solidity
File: src/ArbitrumBranchPort.sol

127        if (_deposit > 0) {

132        if (_amount - _deposit > 0) {
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol#L127

```solidity
File: src/RootBridgeAgent.sol

363        if (_dParams.amount > 0) {

370        if (_dParams.deposit > 0) {

1146        if (_underlyingAddress == address(0)) if (_deposit > 0) revert UnrecognizedUnderlyingAddress();

1149        if (_amount - _deposit > 0) {

1156        if (_deposit > 0) {

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L363

```solidity
File: src/BranchPort.sol

259            if (_amounts[i] - _deposits[i] > 0) {

266            if (_deposits[i] > 0) {

525        if (_hTokenAmount > 0) {

531        if (_deposit > 0) {

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L259

```solidity
File: src/BranchBridgeAgent.sol

906        if (_amount - _deposit > 0) {

912        if (_deposit > 0) {

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L906

```solidity
File: src/BaseBranchRouter.sol

165        if (_amount - _deposit > 0) {

173        if (_deposit > 0) {

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L165

```solidity
File: src/RootPort.sol

284        if (_amount - _deposit > 0) {

290        if (_deposit > 0) if (!ERC20hTokenRoot(_hToken).mint(_recipient, _deposit, _srcChainId)) revert UnableToMint();

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L284

## [G-11] Do-While loops are cheaper than for loops

If you want to push optimization at the expense of creating slightly unconventional code, Solidity do-while loops are more gas efficient than for loops, even if you add an if-condition check for the case where the loop doesn’t execute at all.

```solidity

// SPDX-License-Identifier: MIT
pragma solidity 0.8.20;

// times == 10 in both tests
contract Loop1 {
    function loop(uint256 times) public pure {
        for (uint256 i; i < times; ) {
            unchecked {
                ++i;
            }
        }
    }
}

contract Loop2 {
    function loop(uint256 times) public pure {
        if (times == 0) {
            return;
        }

        uint256 i;

        do {
            unchecked {
                ++i;
            }
        } while (i < times);
    }
}

```

```solidity
File:  src/MulticallRootRouter.sol

278            for (uint256 i = 0; i < outputParams.outputTokens.length;) {

367            for (uint256 i = 0; i < outputParams.outputTokens.length;) {

455            for (uint256 i = 0; i < outputParams.outputTokens.length;) {

557        for (uint256 i = 0; i < outputTokens.length;) {

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L278

```solidity
File: src/RootBridgeAgentExecutor.sol

281        for (uint256 i = 0; i < uint256(uint8(numOfAssets));) {

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L281

```solidity
File: src/RootBridgeAgent.sol

318        for (uint256 i = 0; i < settlement.hTokens.length;) {

399        for (uint256 i = 0; i < length;) {

1070        for (uint256 i = 0; i < hTokens.length;) {

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L318

```solidity
File: src/VirtualAccount.sol

70        for (uint256 i = 0; i < length;) {

90        for (uint256 i = 0; i < length;) {

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L70

```solidity
File: src/BranchPort.sol

257        for (uint256 i = 0; i < length;) {

305        for (uint256 i = 0; i < length;) {

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L257

```solidity
File: src/BranchBridgeAgent.sol

447        for (uint256 i = 0; i < deposit.tokens.length;) {

513        for (uint256 i = 0; i < numOfAssets;) {

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L447

```solidity
File: src/BaseBranchRouter.sol

192        for (uint256 i = 0; i < _hTokens.length;) {

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L192

## [G-12] Use assembly to reuse memory space when making more than one external call

An operation that causes the solidity compiler to expand memory is making external calls. When making external calls the compiler has to encode the function signature of the function it wishes to call on the external contract alongside it’s arguments in memory. As we know, solidity does not clear or reuse memory memory so it’ll have to store these data in the next free memory pointer which expands memory further.

With inline assembly, we can either use the scratch space and free memory pointer offset to store this data (as above) if the function arguments do not take up more than 96 bytes in memory. Better still, if we are making more than one external call we can reuse the same memory space as the first calls to store the new arguments in memory without expanding memory unnecessarily. Solidity in this scenario would expand memory by as much as the returned data length is. This is because the returned data is stored in memory (in most cases). If the return data is less than 96 bytes, we can use the scratch space to store it to prevent expanding memory.

See the example below;

```solidity
contract Called {
    function add(uint256 a, uint256 b) external pure returns (uint256) {
        return a + b;
    }
}

contract Solidity {
    // cost: 7262
    function call(address calledAddress) external pure returns (uint256) {
        Called called = Called(calledAddress);
        uint256 res1 = called.add(1, 2);
        uint256 res2 = called.add(3, 4);

        uint256 res = res1 + res2;
        return res;
    }
}

contract Assembly {
    // cost: 5281
    function call(address calledAddress) external view returns (uint256) {
        assembly {
            // check that calledAddress has code deployed to it
            if iszero(extcodesize(calledAddress)) {
                revert(0x00, 0x00)
            }

            // first call
            mstore(0x00, hex"771602f7")
            mstore(0x04, 0x01)
            mstore(0x24, 0x02)
            let success := staticcall(
                gas(),
                calledAddress,
                0x00,
                0x44,
                0x60,
                0x20
            )
            if iszero(success) {
                revert(0x00, 0x00)
            }
            let res1 := mload(0x60)

            // second call
            mstore(0x04, 0x03)
            mstore(0x24, 0x4)
            success := staticcall(gas(), calledAddress, 0x00, 0x44, 0x60, 0x20)
            if iszero(success) {
                revert(0x00, 0x00)
            }
            let res2 := mload(0x60)

            // add results
            let res := add(res1, res2)

            // return data
            mstore(0x60, res)
            return(0x60, 0x20)
        }
    }
}
```

We save approximately 2,000 gas by using the scratch space to store the function selector and it’s arguments and also reusing the same memory space for the second call while storing the returned data in the zero slot thus not expanding memory.

If the arguments of the external function you wish to call is above 64 bytes and if you are making one external call, it wouldn’t save any significant gas writing it in assembly. However, if making more than one call. You can still save gas by reusing the same memory slot for the 2 calls using inline assembly.

Note: Always remember to update the free memory pointer if the offset it points to is already used, to avoid solidity overriding the data stored there or using the value stored there in an unexpected way.

Also note to avoid overwriting the zero slot (0x60 memory offset) if you have undefined dynamic memory values within that call stack. An alternative is to explicitly define dynamic memory values or if used, to set the slot back to 0x00 before exiting the assembly block.

```solidity
File: src/factories/ERC20hTokenBranchFactory.sol

66            ERC20(_wrappedNativeTokenAddress).name(),
67            ERC20(_wrappedNativeTokenAddress).symbol(),
68            ERC20(_wrappedNativeTokenAddress).decimals(),
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenBranchFactory.sol#L66C2-L68C58

```solidity
File: src/ArbitrumBranchPort.sol

83        if (!IRootPort(_rootPortAddress).isGlobalToken(_globalAddress, localChainId)) revert UnknownGlobalToken();

87            IRootPort(_rootPortAddress).getUnderlyingTokenFromLocal(_globalAddress, localChainId);

92        IRootPort(_rootPortAddress).burnFromLocalBranch(_depositor, _globalAddress, _amount);

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol#L83

```solidity
File: src/ArbitrumCoreBranchRouter.sol

97        if (!IPort(_localPortAddress).isBridgeAgentFactory(_branchBridgeAgentFactory)) {

102        address newBridgeAgent = IBridgeAgentFactory(_branchBridgeAgentFactory).createBridgeAgent(

107        if (!IPort(_localPortAddress).isBridgeAgent(newBridgeAgent)) {

118        IBridgeAgent(localBridgeAgentAddress).callOutSystem(payable(_refundee), payload, _gParams);

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumCoreBranchRouter.sol#L97

```solidity
File: src/BaseBranchRouter.sol

168                ERC20(_hToken).approve(_localPortAddress, _amount - _deposit);

175            ERC20(_token).approve(_localPortAddress, _deposit);

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L168

```solidity
File: src/BranchBridgeAgent.sol

908                IPort(localPortAddress).bridgeIn(_recipient, _hToken, _amount - _deposit);

913            IPort(localPortAddress).withdraw(_recipient, _token, _deposit);

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L908

```solidity
File: src/CoreBranchRouter.sol


211        if (!IPort(_localPortAddress).isBridgeAgentFactory(_branchBridgeAgentFactory)) {
221        if (!IPort(_localPortAddress).isBridgeAgent(newBridgeAgent)) {
216        address newBridgeAgent = IBridgeAgentFactory(_branchBridgeAgentFactory).createBridgeAgent(
232        IBridgeAgent(localBridgeAgentAddress).callOutSystem{value: msg.value}(payable(_refundee), payload, _gParams);



249        if (IPort(_localPortAddress).isBridgeAgentFactory(_newBridgeAgentFactoryAddress)) {
251            IPort(_localPortAddress).toggleBridgeAgentFactory(_newBridgeAgentFactoryAddress);
254            IPort(_localPortAddress).addBridgeAgentFactory(_newBridgeAgentFactoryAddress);



268        if (!IPort(_localPortAddress).isBridgeAgent(_branchBridgeAgent)) revert UnrecognizedBridgeAgent();
271        IPort(_localPortAddress).toggleBridgeAgent(_branchBridgeAgent);



289        if (IPort(_localPortAddress).isStrategyToken(_underlyingToken)) {
291            IPort(_localPortAddress).toggleStrategyToken(_underlyingToken);
394            IPort(_localPortAddress).addStrategyToken(_underlyingToken, _minimumReservesRatio);



317        if (!IPort(_localPortAddress).isPortStrategy(_portStrategy, _underlyingToken)) {
319            IPort(_localPortAddress).addPortStrategy(_portStrategy, _underlyingToken, _dailyManagementLimit);
322            IPort(_localPortAddress).updatePortStrategy(_portStrategy, _underlyingToken, _dailyManagementLimit);
325            IPort(_localPortAddress).togglePortStrategy(_portStrategy, _underlyingToken);
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L317

```solidity
File: src/CoreRootRouter.sol

112        if (msg.sender != IPort(rootPortAddress).getBridgeAgentManager(_rootBridgeAgent)) {
117        if (!IPort(rootPortAddress).isChainId(_dstChainId)) revert InvalidChainId();
120        if (IBridgeAgent(_rootBridgeAgent).getBranchBridgeAgent(_dstChainId) != address(0)) revert InvalidChainId();
123        if (!IBridgeAgent(_rootBridgeAgent).isBranchBridgeAgentAllowed(_dstChainId)) revert UnauthorizedChainId();


461        if (IPort(rootPortAddress).isGlobalAddress(_underlyingAddress)) revert TokenAlreadyAdded();
462        if (IPort(rootPortAddress).isLocalToken(_underlyingAddress, _srcChainId)) revert TokenAlreadyAdded();
463        if (IPort(rootPortAddress).isUnderlyingToken(_underlyingAddress, _srcChainId)) revert TokenAlreadyAdded();

469        IPort(rootPortAddress).setAddresses(

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L461

```solidity
File: src/MulticallRootRouter.sol

239            IVirtualAccount(userAccount).call(calls);
248            IVirtualAccount(userAccount).call(calls);
251            IVirtualAccount(userAccount).withdrawERC20(outputParams.outputToken, outputParams.amountOut);
255                IVirtualAccount(userAccount).userAddress(),
275            IVirtualAccount(userAccount).call(calls);

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L239

## [G-13] Missing Zero address checks in the constructor

Missing checks for zero-addresses may lead to infunctional protocol, if the variable addresses are updated incorrectly. It also wast gas as it requires the redeployment of the contract.

```solidity
File: src/CoreRootRouter.sol

71    constructor(uint256 _rootChainId, address _rootPortAddress) {
72        rootChainId = _rootChainId;
73        rootPortAddress = _rootPortAddress;
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L73

```solidity
File: src/CoreBranchRouter.sol

30    constructor(address _hTokenFactoryAddress) BaseBranchRouter() {
31        hTokenFactoryAddress = _hTokenFactoryAddress;
32    }
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L31

```solidity
File: src/VirtualAccount.sol

35    constructor(address _userAddress, address _localPortAddress) {
36        userAddress = _userAddress;
37        localPortAddress = _localPortAddress;
38    }
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L36

## [G-14] Short-circuit booleans

In Solidity, when you evaluate a boolean expression (e.g the || (logical or) or && (logical and) operators), in the case of || the second expression will only be evaluated if the first expression evaluates to false and in the case of && the second expression will only be evaluated if the first expression evaluates to true. This is called short-circuiting.

For example, the expression require(msg.sender == owner || msg.sender == manager) will pass if the first expression msg.sender == owner evaluates to true. The second expression msg.sender == manager will not be evaluated at all.

However, if the first expression msg.sender == owner evaluates to false, the second expression msg.sender == manager will be evaluated to determine whether the overall expression is true or false. Here, by checking the condition that is most likely to pass firstly, we can avoid checking the second condition thereby saving gas in majority of successful calls.

This is similar for the expression require(msg.sender == owner && msg.sender == manager). If the first expression msg.sender == owner evaluates to false, the second expression msg.sender == manager will not be evaluated because the overall expression cannot be true. For the overall statement to be true, both side of the expression must evaluate to true. Here, by checking the condition that is most likely to fail firstly, we can avoid checking the second condition thereby saving gas in majority of call reverts.

Short-circuiting is useful and it’s recommended to place the less expensive expression first, as the more costly one might be bypassed. If the second expression is more important than the first, it might be worth reversing their order so that the cheaper one gets evaluated first.

```solidity
File: src/BranchPort.sol

363        if (_minimumReservesRatio >= DIVISIONER || _minimumReservesRatio < MIN_RESERVE_RATIO) {

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L363

```solidity
File: src/BranchBridgeAgent.sol

127            _lzEndpointAddress != address(0) || _rootChainId == _localChainId,

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L127

```solidity
File: src/factories/BranchBridgeAgentFactory.sol

63            _lzEndpointAddress != address(0) || _rootChainId == _localChainId,

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/BranchBridgeAgentFactory.sol#L63

```solidity
File: src/RootBridgeAgent.sol

1141        if (_amount == 0 && _deposit == 0) revert InvalidInputParams();

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1141

## [G-15] Pre-increment and pre-decrement are cheaper than +1 ,-1

```solidity
File: src/RootBridgeAgent.sol

978        settlementNonce = _settlementNonce + 1;

1064        settlementNonce = _settlementNonce + 1;

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L978

```solidity
File: src/BranchBridgeAgent.sol

821        depositNonce = _depositNonce + 1;

875        depositNonce = _depositNonce + 1;

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L821

## [G-16] Use hardcode address instead address(this)

It can be more gas-efficient to use a hardcoded address instead of the address(this) expression, especially if you need to use the same address multiple times in your contract.

The reason for this, is that using address(this) requires an additional EXTCODESIZE operation to retrieve the contract’s address from its bytecode, which can increase the gas cost of your contract. By pre-calculating and using a hardcoded address, you can avoid this additional operation and reduce the overall gas cost of your contract.

Here’s an example of how you can use a hardcoded address instead of address(this):

```solidity
contract MyContract {
    address public myAddress = 0x1234567890123456789012345678901234567890;

    function doSomething() public {
        // Use myAddress instead of address(this)
        require(msg.sender == myAddress, "Caller is not authorized");

        // Do something
    }
}

```

In the above example, we have a contract (MyContract) with a public address variable myAddress. Instead of using address(this) to retrieve the contract’s address, we have pre-calculated and hardcoded the address in the variable. This can help to reduce the gas cost of our contract and make our code more efficient.

[Reference](https://book.getfoundry.sh/reference/forge-std/compute-create-address)

```solidity
File: src/BranchPort.sol

175        uint256 currBalance = ERC20(_token).balanceOf(address(this));

178        IPortStrategy(msg.sender).withdraw(address(this), _token, _amount);

181        require(ERC20(_token).balanceOf(address(this)) - currBalance == _amount, "Port Strategy Withdraw Failed");

193        uint256 currBalance = ERC20(_token).balanceOf(address(this));

210        IPortStrategy(_strategy).withdraw(address(this), _token, amountToWithdraw);

214            ERC20(_token).balanceOf(address(this)) - currBalance == amountToWithdraw, "Port Strategy Withdraw Failed"

436        uint256 currBalance = ERC20(_token).balanceOf(address(this));

526            _localAddress.safeTransferFrom(_depositor, address(this), _hTokenAmount);

532            _underlyingAddress.safeTransferFrom(_depositor, address(this), _deposit);

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L175

```solidity
File: src/ArbitrumBranchBridgeAgent.sol

126        if (msg.sender != address(this)) revert LayerZeroUnauthorizedEndpoint();

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchBridgeAgent.sol#L126

```solidity
File: src/ArbitrumBranchPort.sol

66        _underlyingAddress.safeTransferFrom(_depositor, address(this), _deposit);

128            _underlyingAddress.safeTransferFrom(_depositor, address(this), _deposit);

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol#L66

```solidity
File: src/RootBridgeAgent.sol

120        bridgeAgentExecutorAddress = DeployRootBridgeAgentExecutor.deploy(address(this));

148            address(this),

424        (bool success,) = address(this).excessivelySafeCall(

695                address(this).balance

754        (bool success,) = bridgeAgentExecutorAddress.call{value: address(this).balance}(_calldata);
779        (bool success,) = bridgeAgentExecutorAddress.call{value: address(this).balance}(_calldata);

940        ILayerZeroEndpoint(lzEndpointAddress).send{value: address(this).balance}(

1182        getBranchBridgeAgentPath[_branchChainId] = abi.encodePacked(_newBranchBridgeAgent, address(this));

1205        if (msg.sender != address(this)) revert LayerZeroUnauthorizedEndpoint();

1233        if (msg.sender != IPort(localPortAddress).getBridgeAgentManager(address(this))) {
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L120

```solidity
File: src/BranchBridgeAgent.sol

142        rootBridgeAgentPath = abi.encodePacked(_rootBridgeAgentAddress, address(this));

168            address(this),

579        address(this).excessivelySafeCall(

717        (bool success,) = bridgeAgentExecutorAddress.call{value: address(this).balance}(_calldata);

737        (bool success,) = bridgeAgentExecutorAddress.call{value: address(this).balance}(_calldata);

787        ILayerZeroEndpoint(lzEndpointAddress).send{value: address(this).balance}(

938        if (msg.sender != address(this)) revert LayerZeroUnauthorizedEndpoint();
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L142

```solidity
File: src/BaseBranchRouter.sol

167                _hToken.safeTransferFrom(msg.sender, address(this), _amount - _deposit);

174            _token.safeTransferFrom(msg.sender, address(this), _deposit);
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L167

```solidity
File: src/RootPort.sol

301        _hToken.safeTransferFrom(_from, address(this), _amount);

362        newAccount = new VirtualAccount{salt: keccak256(abi.encode(_user))}(_user, address(this));
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L301

```solidity
File:
```

## [G-17] Use bytes.concat() instead of abi.encodePacked(), since this is preferred since 0.8.4

It's recommended to use the bytes.concat() function instead of abi.encodePacked() for concatenating byte arrays. bytes.concat() is a more gas-efficient and safer way to concatenate byte arrays, and it's considered a best practice in newer Solidity versions.

```solidity
File: src/ArbitrumBranchBridgeAgent.sol

115            rootChainId, "", 0, abi.encodePacked(bytes1(0x09), _settlementNonce)

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchBridgeAgent.sol#L115

```solidity
File: src/ArbitrumCoreBranchRouter.sol

62        bytes memory payload = abi.encodePacked(bytes1(0x02), params);

115        bytes memory payload = abi.encodePacked(bytes1(0x04), data);
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumCoreBranchRouter.sol#L62

```solidity
File: src/CoreRootRouter.sol

136        bytes memory payload = abi.encodePacked(bytes1(0x02), params);

171        bytes memory payload = abi.encodePacked(bytes1(0x03), params);

196        bytes memory payload = abi.encodePacked(bytes1(0x04), params);

223        bytes memory payload = abi.encodePacked(bytes1(0x05), params);

254        bytes memory payload = abi.encodePacked(bytes1(0x06), params);

284        bytes memory payload = abi.encodePacked(bytes1(0x07), params);

434        bytes memory payload = abi.encodePacked(bytes1(0x01), params);
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L136

```solidity
File: src/CoreBranchRouter.sol

51        bytes memory payload = abi.encodePacked(bytes1(0x01), params);

75        bytes memory payload = abi.encodePacked(bytes1(0x02), params);

181        bytes memory payload = abi.encodePacked(bytes1(0x03), params);

229        bytes memory payload = abi.encodePacked(bytes1(0x04), params);
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L51

```solidity
File: src/RootBridgeAgent.sol

151            abi.encodePacked(uint16(2), _gasLimit, _remoteBranchExecutionGas, getBranchBridgeAgent[_dstChainId])

168        bytes memory payload = abi.encodePacked(bytes1(0x00), _recipient, settlementNonce++, _params);

292        bytes memory payload = abi.encodePacked(bytes1(0x03), settlementOwner, _settlementNonce);

829                abi.encodePacked(uint16(2), _gParams.gasLimit, _gParams.remoteBranchExecutionGas, callee)

879            payload = abi.encodePacked(

893            payload = abi.encodePacked(

921                abi.encodePacked(uint16(2), _gParams.gasLimit, _gParams.remoteBranchExecutionGas, callee)

943            abi.encodePacked(bytes1(0x04), _depositNonce),

992        _payload = abi.encodePacked(

1089        _payload = abi.encodePacked(

1182        getBranchBridgeAgentPath[_branchChainId] = abi.encodePacked(_newBranchBridgeAgent, address(this));
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L151

```solidity
File: src/BranchBridgeAgent.sol

142        rootBridgeAgentPath = abi.encodePacked(_rootBridgeAgentAddress, address(this));

171            abi.encodePacked(uint16(2), _gasLimit, _remoteBranchExecutionGas, rootBridgeAgentAddress)

188        bytes memory payload = abi.encodePacked(bytes1(0x00), depositNonce++, _params);

202        bytes memory payload = abi.encodePacked(bytes1(0x01), depositNonce++, _params);

219        bytes memory payload = abi.encodePacked(

241        bytes memory payload = abi.encodePacked(

269        bytes memory payload = abi.encodePacked(bytes1(0x04), msg.sender, depositNonce++, _params);

287        bytes memory payload = abi.encodePacked(

317        bytes memory payload = abi.encodePacked(

362                payload = abi.encodePacked(

373                payload = abi.encodePacked(

386                payload = abi.encodePacked(

398                payload = abi.encodePacked(

427        bytes memory payload = abi.encodePacked(bytes1(0x08), msg.sender, _depositNonce);

474        bytes memory payload = abi.encodePacked(_hasFallbackToggled ? bytes1(0x87) : bytes1(0x07), params);

776            abi.encodePacked(uint16(2), _gParams.gasLimit, _gParams.remoteBranchExecutionGas, rootBridgeAgentAddress)

790            abi.encodePacked(bytes1(0x09), _settlementNonce),

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L142

## [G-18] Shorten the array rather than copying to a new one

Inline-assembly can be used to shorten the array by changing the length slot, so that the entries don't have to be copied to a new, shorter array

1. Shorten the array rather than copying to a new one:

   This suggests that instead of creating a new array with fewer elements (by copying data from the original array to the new one), it's more efficient to modify the original array to have a shorter length.

2. Inline-assembly can be used to shorten the array by changing the length slot:

   Use inline assembly to directly manipulate the length of an array.

3. So that the entries don't have to be copied to a new, shorter array:

   When you shorten an array by changing its length directly using inline assembly, you avoid the overhead of copying the array's elements to a new one. This is more gas-efficient, especially when dealing with large arrays, as it reduces the computational and gas costs associated with copying data.

```solidity
File: src/RootBridgeAgentExecutor.sol

276        address[] memory hTokens = new address[](numOfAssets);
277        address[] memory tokens = new address[](numOfAssets);
278        uint256[] memory amounts = new uint256[](numOfAssets);
279        uint256[] memory deposits = new uint256[](numOfAssets);
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L276C1-L279C64

```solidity
File: src/RootBridgeAgent.sol

1007        address[] memory addressArray = new address[](1);
1008        uint256[] memory uintArray = new uint256[](1);

1067        address[] memory hTokens = new address[](_globalAddresses.length);
1068        address[] memory tokens = new address[](_globalAddresses.length);
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1007C1-L1009C1

```solidity
File: src/BranchBridgeAgent.sol

507        address[] memory _hTokens = new address[](numOfAssets);
508        address[] memory _tokens = new address[](numOfAssets);
509        uint256[] memory _amounts = new uint256[](numOfAssets);
510        uint256[] memory _deposits = new uint256[](numOfAssets);

827        address[] memory addressArray = new address[](1);
828        uint256[] memory uintArray = new uint256[](1);
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L507C1-L510C65

## [G-19] Initializers can be marked as payable to save deployment gas

```solidity
File: src/MulticallRootRouter.sol

109    function initialize(address _bridgeAgentAddress) external onlyOwner {

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L109

```solidity
File: src/CoreRootRouter.sol

83    function initialize(address _bridgeAgentAddress, address _hTokenFactory) external onlyOwner {

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L83

```solidity
File: src/BranchPort.sol

122    function initialize(address _coreBranchRouter, address _bridgeAgentFactory) external virtual onlyOwner {

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L122

```solidity
File: src/BaseBranchRouter.sol

60    function initialize(address _localBridgeAgentAddress) external onlyOwner {

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L60

```solidity
File: src/RootPort.sol

129    function initialize(address _bridgeAgentFactory, address _coreRootRouter) external onlyOwner {

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L129

```solidity
File:  src/factories/BranchBridgeAgentFactory.sol

87    function initialize(address _coreRootBridgeAgent) external virtual onlyOwner {

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/BranchBridgeAgentFactory.sol#L87

```solidity
File: src/factories/ERC20hTokenBranchFactory.sol

60    function initialize(address _wrappedNativeTokenAddress, address _coreRouter) external onlyOwner {

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenBranchFactory.sol#L60

```solidity
File: src/factories/ERC20hTokenRootFactory.sol

49    function initialize(address _coreRouter) external onlyOwner {

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenRootFactory.sol#L49

```solidity
File: src/factories/ArbitrumBranchBridgeAgentFactory.sol

56    function initialize(address _coreRootBridgeAgent) external override onlyOwner {
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ArbitrumBranchBridgeAgentFactory.sol#L56

## [G-20] Use assembly to validate msg.sender

Using assembly to validate msg.sender means that you can manually inspect and manipulate the value of msg.sender using inline assembly code to enforce custom validation rules or conditions.

For-Example

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract SenderValidationExample {
    address public owner;

    constructor() {
        owner = msg.sender;
    }

    function validateSender() public view returns (bool) {
        bool isOwner;

        assembly {
            // Load the current sender (msg.sender) into a temporary variable
            isOwner := eq(sender, sload(owner.slot))
        }

        return isOwner;
    }
}

```

```solidity
File: src/ArbitrumBranchBridgeAgent.sol

126        if (msg.sender != address(this)) revert LayerZeroUnauthorizedEndpoint();

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchBridgeAgent.sol#L126

```solidity
File: src/MulticallRootRouter.sol

605        if (msg.sender != bridgeAgentExecutorAddress) revert UnrecognizedBridgeAgentExecutor();
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L605

```solidity
File: src/CoreRootRouter.sol

112        if (msg.sender != IPort(rootPortAddress).getBridgeAgentManager(_rootBridgeAgent)) {

278        require(msg.sender == rootPortAddress, "Only root port can call");

512        if (msg.sender != bridgeAgentExecutorAddress) revert UnrecognizedBridgeAgentExecutor();

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L112

```solidity
File: src/RootBridgeAgent.sol

247        if (msg.sender != settlementReference.owner) {

285        if (msg.sender != settlementOwner) {

286            if (msg.sender != address(IPort(localPortAddress).getUserAccount(settlementOwner))) {

311        if (msg.sender != settlementOwner) {

312            if (msg.sender != address(IPort(localPortAddress).getUserAccount(settlementOwner))) {

430        if (!success) if (msg.sender == getBranchBridgeAgent[localChainId]) revert ExecutionFailure();

1199        if (msg.sender != localRouterAddress) revert UnrecognizedRouter();

1205        if (msg.sender != address(this)) revert LayerZeroUnauthorizedEndpoint();

1221        if (msg.sender != bridgeAgentExecutorAddress) revert UnrecognizedExecutor();

1227        if (msg.sender != localPortAddress) revert UnrecognizedPort();

1233        if (msg.sender != IPort(localPortAddress).getBridgeAgentManager(address(this))) {

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L247

```solidity
File: src/BranchBridgeAgent.sol

354        if (deposit.owner != msg.sender) revert NotDepositOwner();

424        if (getDeposit[_depositNonce].owner != msg.sender) revert NotDepositOwner();

441        if (deposit.owner != msg.sender) revert NotDepositOwner();

938        if (msg.sender != address(this)) revert LayerZeroUnauthorizedEndpoint();

948        if (msg.sender != localRouterAddress) revert UnrecognizedRouter();

954        if (msg.sender != bridgeAgentExecutorAddress) revert UnrecognizedBridgeAgentExecutor();

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L354

## [G-21] Use selfdestruct in the constructor if the contract is one-time use

Sometimes, contracts are used to deploy several contracts in one transaction, which necessitates doing it in the constructor.

If the contract’s only use is the code in the constructor, then selfdestructing at the end of the operation will save gas.

Although selfdestruct is set for removal in an upcoming hardfork, it will still be supported in the constructor per EIP 6780

## [G-22] Avoid contract calls by making the architecture monolithic

Contract calls are expensive, and the best way to save gas on them is by not using them at all. There is a natural tradeoff with this, but having several contracts that talk to each other can sometimes increase gas and complexity rather than manage it.

```solidity
File: src/ArbitrumBranchBridgeAgent.sol

103        _rootBridgeAgentAddress.call{value: msg.value}("");

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchBridgeAgent.sol#L103

```solidity
File: src/RootBridgeAgent.sol

754        (bool success,) = bridgeAgentExecutorAddress.call{value: address(this).balance}(_calldata);

779        (bool success,) = bridgeAgentExecutorAddress.call{value: address(this).balance}(_calldata);

835            callee.call{value: msg.value}("");

927            callee.call{value: _value}("");

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L754

```solidity
File: src/VirtualAccount.sol

101            if (isContract(_call.target)) (success, returnData[i]) = _call.target.call{value: val}(_call.callData);

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L101

```solidity
File: src/BranchBridgeAgent.sol

717        (bool success,) = bridgeAgentExecutorAddress.call{value: address(this).balance}(_calldata);

737        (bool success,) = bridgeAgentExecutorAddress.call{value: address(this).balance}(_calldata);

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L717

## [G-23] It is sometimes cheaper to cache calldata

Although the calldataload instruction is a cheap opcode, the solidity compiler will sometimes output cheaper code if you cache calldataload. This will not always be the case, so you should test both possibilities.

```solidity
contract LoopSum {
    function sumArr(uint256[] calldata arr) public pure returns (uint256 sum) {
        uint256 len = arr.length;
        for (uint256 i = 0; i < len; ) {
            sum += arr[i];
            unchecked {
                ++i;
            }
        }
    }
}

```

```solidity
File: src/ArbitrumBranchBridgeAgent.sol

89    function retrySettlement(uint32, bytes calldata, GasParams[2] calldata, bool) external payable override lock {}

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchBridgeAgent.sol#L89

```solidity
File: src/CoreRootRouter.sol

109        GasParams[2] calldata _gParams
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L109

```solidity
File: src/CoreBranchRouter.sol

43    function addGlobalToken(address _globalAddress, uint256 _dstChainId, GasParams[3] calldata _gParams)

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L43

```solidity
File: src/VirtualAccount.sol

66    function call(Call[] calldata calls) external override requiresApprovedCaller returns (bytes[] memory returnData) {

85    function payableCall(PayableCall[] calldata calls) public payable returns (bytes[] memory returnData) {

134    function onERC1155BatchReceived(address, address, uint256[] calldata, uint256[] calldata, bytes calldata)
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L66

```solidity
File: src/BranchBridgeAgent.sol

467        GasParams[2] calldata _gParams,

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L467

## [G-24] Use ++i instead of i++ to increment

The reason behind this is in way ++i and i++ are evaluated by the compiler.

i++ returns i(its old value) before incrementing i to a new value. This means that 2 values are stored on the stack for usage whether you wish to use it or not. ++i on the other hand, evaluates the ++ operation on i (i.e it increments i) then returns i (its incremented value) which means that only one item needs to be stored on the stack.

```solidity
File: src/RootBridgeAgent.sol

// @audit settlementNonce++
168        bytes memory payload = abi.encodePacked(bytes1(0x00), _recipient, settlementNonce++, _params);

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L168

```solidity
File: src/BranchBridgeAgent.sol

188        bytes memory payload = abi.encodePacked(bytes1(0x00), depositNonce++, _params);

202        bytes memory payload = abi.encodePacked(bytes1(0x01), depositNonce++, _params);

269        bytes memory payload = abi.encodePacked(bytes1(0x04), msg.sender, depositNonce++, _params);
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L188

## [G-25] Always use Named Returns

The solidity compiler outputs more efficient code when the variable is declared in the return statement. There seem to be very few exceptions to this in practice, so if you see an anonymous return, you should test it with a named return instead to determine which case is most efficient.

```solidity

// SPDX-License-Identifier: MIT
pragma solidity 0.8.20;

contract NamedReturn {
    function myFunc1(uint256 x, uint256 y) external pure returns (uint256) {
        require(x > 0);
        require(y > 0);

        return x * y;
    }
}

contract NamedReturn2 {
    function myFunc2(uint256 x, uint256 y) external pure returns (uint256 z) {
        require(x > 0);
        require(y > 0);

        z = x * y;
    }
}

```

```solidity
File: src/ArbitrumBranchBridgeAgent.sol

15        return new ArbitrumBranchBridgeAgent(
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchBridgeAgent.sol#L15

```solidity
File: src/MulticallRootRouter.sol

582        return data;
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L582

```solidity
File: src/BranchPort.sol

440            return currBalance > minReserves ? currBalance - minReserves : 0;

473        return ((_currBalance + _strategyTokenDebt) * getMinimumTokenReserveRatio[_token]) / DIVISIONER;

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L440

```solidity
File: src/MulticallRootRouterLibZip.sol

38        return data.cdDecompress();

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouterLibZip.sol#L38

```solidity
File: src/BranchBridgeAgent.sol

32        return new BranchBridgeAgent(

157        return getDeposit[_depositNonce];

570        return SettlementMultipleParams(numOfAssets, _recipient, nonce, _hTokens, _tokens, _amounts, _deposits);

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L32

```solidity
File: src/BaseBranchRouter.sol

75        return IBridgeAgent(localBridgeAgentAddress).getDepositEntry(_depositNonce);

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L75

```solidity
File: src/RootPort.sol

181        return _getLocalToken(_localAddress, _srcChainId, _dstChainId);

196        return getLocalTokenFromGlobal[globalAddress][_dstChainId];

207        return getUnderlyingTokenFromLocal[localAddress][_srcChainId];

212        return getLocalTokenFromGlobal[_globalAddress][_srcChainId] != address(0);

217        return getGlobalTokenFromLocal[_localAddress][_srcChainId] != address(0);

226        return _getLocalToken(_localAddress, _srcChainId, _dstChainId) != address(0);

231        return getLocalTokenFromUnderlying[_underlyingToken][_srcChainId] != address(0);

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L181

## [G-26] Use bitmaps instead of bools when a significant amount of booleans are used

A common pattern, especially in airdrops, is to mark an address as “already used” when claiming the airdrop or NFT mint.

However, since it only takes one bit to store this information, and each slot is 256 bits, that means one can store a 256 flags/booleans with one storage slot.

```solidity
File: src/RootBridgeAgent.sol

68    mapping(uint256 chainId => bool allowed) public isBranchBridgeAgentAllowed;

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L68

```solidity
File: src/BranchPort.sol

32    mapping(address bridgeAgent => bool isActiveBridgeAgent) public isBridgeAgent;

42    mapping(address bridgeAgentFactory => bool isActiveBridgeAgentFactory) public isBridgeAgentFactory;

52    mapping(address token => bool allowsStrategies) public isStrategyToken;

68    mapping(address strategy => mapping(address token => bool isActiveStrategy)) public isPortStrategy;

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L32

```solidity
File: src/RootPort.sol

54    mapping(VirtualAccount acount => mapping(address router => bool allowed)) public isRouterApproved;

61    mapping(uint256 chainId => bool isActive) public isChainId;

64    mapping(address bridgeAgent => bool isActive) public isBridgeAgent;

77    mapping(address bridgeAgentFactory => bool isActive) public isBridgeAgentFactory;

87    mapping(address token => bool isGlobalToken) public isGlobalAddress;

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L54

## [G-27] Empty blocks should be removed or emit something

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting.
If the contract is meant to be extended, the contract should be abstract and the function signatures be added without any default implementation.

If the block is an empty if-statement block to avoid doing subsequent checks in the else-if/else conditions, the else-if/else conditions should be
nested under the negation of the if-statement, because they involve different classes of checks, which may lead to the introduction of errors when
the code is later modified (if( x ) {}else if(y){ . . . }else{ . . . } => if ( !x) {if(y) { . . . }else{ . . .}}) .
Empty receive()/fallback() payable functions that are not used, can be removed to save deployment gas.

```solidity
File: src/ArbitrumBranchBridgeAgent.sol

57    {}

89    function retrySettlement(uint32, bytes calldata, GasParams[2] calldata, bool) external payable override lock {}
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchBridgeAgent.sol#L57

```solidity
File: src/ArbitrumCoreBranchRouter.sol

44    constructor() CoreBranchRouter(address(0)) {}
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumCoreBranchRouter.sol#L44

```solidity
File: src/MulticallRootRouterLibZip.sol

31    {}
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouterLibZip.sol#L31

```solidity
File: src/factories/ArbitrumBranchBridgeAgentFactory.sol

46    {}
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ArbitrumBranchBridgeAgentFactory.sol#L46
