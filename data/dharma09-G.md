### **[G-1] `State variables` can be packed into fewer storage slots**

`Save 1 slot`

here we can change position of `_setup` and `_setupCore.` In result we get decent amount of gas saving.

- https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L24C5-L43C47

```solidity
File: src/RootPort.sol
24:    bool internal _setup; //@audit Tite pack state variable
25:
26:    /// @notice True if core setup is still ongoing, false otherwise.
27:    bool internal _setupCore;
28:
    /*///////////////////////////////////////////////////////////////
                        ROOT PORT STATE
    //////////////////////////////////////////////////////////////*/

33:    /// @notice Local Chain Id
34:    uint256 public immutable localChainId;
35:
36:    /// @notice The address of local branch port responsible for handling local transactions.
37:    address public localBranchPortAddress; 
38:
39:    /// @notice The address of the core router in charge of adding new tokens to the system.
40:    address public coreRootRouterAddress;
41:
42:    /// @notice The address of the core router in charge of adding new tokens to the system.
43:    address public coreRootBridgeAgentAddress;
```

```diff
File: src/RootPort.sol
-24:    bool internal _setup; //@audit Tite pack state variable
25:
-26:    /// @notice True if core setup is still ongoing, false otherwise.
-27:    bool internal _setupCore;
28:
    /*///////////////////////////////////////////////////////////////
                        ROOT PORT STATE
    //////////////////////////////////////////////////////////////*/

33:    /// @notice Local Chain Id
34:    uint256 public immutable localChainId;
35:
+        bool internal _setup;
+    /// @notice True if core setup is still ongoing, false otherwise.
+    bool internal _setupCore;
36:    /// @notice The address of local branch port responsible for handling local transactions.
37:    address public localBranchPortAddress; 
38:
39:    /// @notice The address of the core router in charge of adding new tokens to the system.
40:    address public coreRootRouterAddress;
41:
42:    /// @notice The address of the core router in charge of adding new tokens to the system.
43:    address public coreRootBridgeAgentAddress;
```

### [G-02] should use arguments or a value we know instead of state variable

This will save 100 gas because it gets rid of a storage read and instead uses a argument or something cheap that we already have which gives the same result

Use `stack` variable instead of state 

- https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L24C5-L43C47

```solidity
File: src\RootBridgeAgent.sol
1072: hTokens[i] = IPort(localPortAddress).getLocalTokenFromGlobal(_globalAddresses[i], _dstChainId); //@audit Use cache value of dstChainId
1073:            tokens[i] = IPort(localPortAddress).getUnderlyingTokenFromLocal(hTokens[i], _dstChainId);
1074:
1075:            // Avoid stack too deep
1076:            uint16 destChainId = _dstChainId;
```

```diff
File: src\RootBridgeAgent.sol
-1072: hTokens[i] = IPort(localPortAddress).getLocalTokenFromGlobal(_globalAddresses[i], _dstChainId); //@audit Use cache value
-1073:            tokens[i] = IPort(localPortAddress).getUnderlyingTokenFromLocal(hTokens[i], _dstChainId);
+1072: hTokens[i] = IPort(localPortAddress).getLocalTokenFromGlobal(_globalAddresses[i], dstChainId); 
+1073:            tokens[i] = IPort(localPortAddress).getUnderlyingTokenFromLocal(hTokens[i], dstChainId);
1074:
1075:            // Avoid stack too deep
1076:            uint16 destChainId = _dstChainId;
```

### [G-03] Cache Variable to memory which save decent amount of gas

Save `localPortAddress` in memory 

- https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1072C4-L1073C102

```diff
File: src\RootBridgeAgent.sol
+       address _localPortAddress = localPortAddress;
-1072: hTokens[i] = IPort(localPortAddress).getLocalTokenFromGlobal(_globalAddresses[i], _dstChainId); //@audit Use cache value of dstChainId
-1073:            tokens[i] = IPort(localPortAddress).getUnderlyingTokenFromLocal(hTokens[i], _dstChainId);
+1072: hTokens[i] = IPort(_localPortAddress).getLocalTokenFromGlobal(_globalAddresses[i], _dstChainId); //@audit Use cache value of dstChainId
+1073:            tokens[i] = IPort(_localPortAddress).getUnderlyingTokenFromLocal(hTokens[i], _dstChainId);
1074:
1075:            // Avoid stack too deep
1076:            uint16 destChainId = _dstChainId;
```

- https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L112

```diff
File: src\CoreRootRouter.sol
+      address _rootPortAddress = rootPortAddress;
-112: if (msg.sender != IPort(rootPortAddress).getBridgeAgentManager(_rootBridgeAgent)) { //@audit save rootPortAddress to memory
+112: if (msg.sender != IPort(_rootPortAddress).getBridgeAgentManager(_rootBridgeAgent))  
113:            revert UnauthorizedCallerNotManager();
114:        }
115:
116:        // Check if valid chain
-117:        if (!IPort(rootPortAddress).isChainId(_dstChainId)) revert InvalidChainId();
+117:        if (!IPort(_rootPortAddress).isChainId(_dstChainId)) revert InvalidChainId();
```

- https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L414C7-L414C7

```diff
File: src\CoreRootRouter.sol
+      address _rootPortAddress = rootPortAddress;
-414: if (!IPort(rootPortAddress).isGlobalAddress(_globalAddress)) { //@audit save rootPortAddress to memory
+414: if (!IPort(_rootPortAddress).isGlobalAddress(_globalAddress))
415:            revert UnrecognizedGlobalToken();
416:        }
417:
418:        // Verify that it does not exist
-419:        if (IPort(rootPortAddress).isGlobalToken(_globalAddress, _dstChainId)) {
+419:        if (IPort(_rootPortAddress).isGlobalToken(_globalAddress, _dstChainId)
420:            revert TokenAlreadyAdded();
421:        }
```

- https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L461C8-L463C115

```diff
File: src\CoreRootRouter.sol
+      address _rootPortAddress = rootPortAddress;
-461: if (IPort(rootPortAddress).isGlobalAddress(_underlyingAddress)) revert TokenAlreadyAdded(); //@audit
-462:        if (IPort(rootPortAddress).isLocalToken(_underlyingAddress, _srcChainId)) revert TokenAlreadyAdded();
-463:        if (IPort(rootPortAddress).isUnderlyingToken(_underlyingAddress, _srcChainId)) revert TokenAlreadyAdded();
+461: if (IPort(_rootPortAddress).isGlobalAddress(_underlyingAddress)) revert TokenAlreadyAdded(); //@audit
+462:        if (IPort(_rootPortAddress).isLocalToken(_underlyingAddress, _srcChainId)) revert TokenAlreadyAdded();
+463:        if (IPort(_rootPortAddress).isUnderlyingToken(_underlyingAddress, _srcChainId)) revert TokenAlreadyAdded();
```

- https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L481C7-L486C92

```diff
File: src\CoreRootRouter.sol
481: function _setLocalToken(address _globalAddress, address _localAddress, uint16 _dstChainId) internal {
+      address _rootPortAddress = rootPortAddress;
482:        // Verify if the token already added
-483:        if (IPort(rootPortAddress).isLocalToken(_localAddress, _dstChainId)) revert TokenAlreadyAdded(); //@audit
+483:        if (IPort(_rootPortAddress).isLocalToken(_localAddress, _dstChainId)) revert TokenAlreadyAdded(); //@audit
484:
485:        // Set the global token's new branch chain address
-486:        IPort(rootPortAddress).setLocalAddress(_globalAddress, _localAddress, _dstChainId);
+486:        IPort(_rootPortAddress).setLocalAddress(_globalAddress, _localAddress, _dstChainId);
487:    }
487:    }
```

- https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L981C6-L984C116

```diff
File: src\RootBridgeAgent.sol
-981: address localAddress = IPort(localPortAddress).getLocalTokenFromGlobal(_globalAddress, _dstChainId);
+981: address localAddress = IPort(_localPortAddress).getLocalTokenFromGlobal(_globalAddress, _dstChainId);
+      address _localPortAddress = localPortAddress;
982:
983:        // Get Underlying Token Address from Root Port
-984:        address underlyingAddress = IPort(localPortAddress).getUnderlyingTokenFromLocal(localAddress, _dstChainId); //@audit cache local port address to memory
+984:        address underlyingAddress = IPort(_localPortAddress).getUnderlyingTokenFromLocal(localAddress, _dstChainId); 
```

- https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/BranchBridgeAgentFactory.sol#L87C1-L87C1

```diff
File: src\factories\BranchBridgeAgentFactory.sol
87: function initialize(address _coreRootBridgeAgent) external virtual onlyOwner {
88:        require(_coreRootBridgeAgent != address(0), "Core Root Bridge Agent cannot be 0");
+      address _localPortAddress = localPortAddress;
90:        address newCoreBridgeAgent = address(
91:            DeployBranchBridgeAgent.deploy(
92:                rootChainId,
93:                localChainId,
94:                _coreRootBridgeAgent,
95:                lzEndpointAddress,
96:                localCoreBranchRouterAddress,
-97:                localPortAddress //@audit cache
+97:                _localPortAddress
98:            )
99:        );
100:
-101:        IPort(localPortAddress).addBridgeAgent(newCoreBridgeAgent);
+101:   IPort(_localPortAddress).addBridgeAgent(newCoreBridgeAgent);
102:
103:        renounceOwnership();
104:    }
```

- https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/BranchBridgeAgentFactory.sol#L115C4-L140C6

```diff
File: src\factories\BranchBridgeAgentFactory.sol
+      address _localPortAddress = localPortAddress;
128: newBridgeAgent = address(
129:            DeployBranchBridgeAgent.deploy(
130:                rootChainId,
131:                localChainId,
132:                _rootBridgeAgentAddress,
133:                lzEndpointAddress,
134:                _newBranchRouterAddress,
-135:               localPortAddress //@audit save port address to memory => address _localPortAddress = localPortAddress;
+135:               _localPortaddress
136:            )
137:        )
-138:        IPort(localPortAddress).addBridgeAgent(newBridgeAgent);
+138:        IPort(_localPortAddress).addBridgeAgent(newBridgeAgent);
```

- https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/RootBridgeAgentFactory.sol#L48C3-L58C72

```diff
File: src\factories\RootBridgeAgentFactory.sol
48: function createBridgeAgent(address _newRootRouterAddress) external returns (address newBridgeAgent) {
+      address _rootPortAddress = rootPortAddress;
49:        newBridgeAgent = address(
50:            new RootBridgeAgent(                rootChainId, 
51:                lzEndpointAddress, 
-52:                rootPortAddress, //@audit
+52:                _rootPortAddress,
53:                _newRootRouterAddress
54:            )
55:        );
56:
-57:        IRootPort(rootPortAddress).addBridgeAgent(msg.sender, newBridgeAgent);
+57:        IRootPort(_rootPortAddress).addBridgeAgent(msg.sender, newBridgeAgent);
58:    }
59: }
```

### [G-04] Ordering struct members in decreasing size can reduce operations gas cost

Ordering the members of a struct by size can help ensure that they are aligned to word boundaries as much as possible. By placing larger members before smaller members, it can reduce the amount of unused space within each 32-byte word and increase the likelihood that each member will be aligned to a word boundary.

 Number of instances `10` 

example:

```diff
struct Deposit {
-    uint8 status;
-    address owner;
-    address[] hTokens;
-    address[] tokens;
    uint256[] amounts;
    uint256[] deposits;
   mapping(address => CurserRecord) private s_curserRecords;
+  address[] tokens;
+  address[] hTokens;
+  address owner;
+  uint8 status;
}
```

- https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/BridgeAgentStructs.sol

### [G-05] Can Make The Variable Outside The Loop To Save Gas

When you declare a variable inside a loop, Solidity creates a new instance of the variable for each iteration of the loop. This can lead to unnecessary gas costs, especially if the loop is executed frequently or iterates over a large number of elements.

- https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L320

```solidity
File: src\RootBridgeAgent.sol
320: address _hToken = settlement.hTokens[i]; //@audit Outside loop
```

- https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L283

```solidity
File: src\RootBridgeAgentExecutor.sol
283: uint256 currentIterationOffset = PARAMS_START + i; //@audit  variable Outside
```

### [G-06] Don’t cache variable if used only once

If function use variable only once then it doesn’t make sense to cache that variable in memory it just waste of gas.Recommeded avoid such allocation.

- https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1004

```solidity
File: src\RootBridgeAgent.sol
1004:        uint32 cachedSettlementNonce = _settlementNonce; //@audit No need to cache use only once
1005:
1006:        // Get Auxiliary Dynamic Arrays
1007:        address[] memory addressArray = new address[](1);
1008:        uint256[] memory uintArray = new uint256[](1);
1009:
1010:        // Get storage reference for new Settlement
1011:        Settlement storage settlement = getSettlement[cachedSettlementNonce];
```

```diff
File: src\RootBridgeAgent.sol
-1004:        uint32 cachedSettlementNonce = _settlementNonce; 
1005:
1006:        // Get Auxiliary Dynamic Arrays
1007:        address[] memory addressArray = new address[](1);
1008:        uint256[] memory uintArray = new uint256[](1);
1009:
1010:        // Get storage reference for new Settlement
-1011:        Settlement storage settlement = getSettlement[cachedSettlementNonce];
+1011:        Settlement storage settlement = getSettlement[_settlementNonce];
```

### [G-07] Use calldata instead of memory for variable

- https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L441C7-L442C46

```solidity
File: src/RootPort.sol
438: function addNewChain(
439:        address _coreBranchBridgeAgentAddress,
440:        uint256 _chainId,
441:        string memory _wrappedGasTokenName, //@audit Use calldata
442:        string memory _wrappedGasTokenSymbol, //@audit Use calldata
443:        uint8 _wrappedGasTokenDecimals,
444:        address _newLocalBranchWrappedNativeTokenAddress,
445:        address _newUnderlyingBranchWrappedNativeTokenAddress
446:    ) external override onlyOwner {
```