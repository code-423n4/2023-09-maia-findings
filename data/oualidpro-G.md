
## Gas Optimizations


| |Issue|Instances|Gas Saved|
|-|:-|:-:|-:|
| [GAS-1](#GAS-1-1693311540) | Multiple accesses of a mapping/array should use a local variable cache | 3 | 126 |
| [GAS-2](#GAS-2-1693311663) | Use assembly in place of `abi.decode` to extract `calldata` values more efficiently | 1 | - |
| [GAS-3](#GAS-3-1693311695) | Use `assembly` to write address storage values | 48 | - |
| [GAS-4](#GAS-4-1694877823) | Use `do while` loops instead of `for` loops | 15 | 1815 |
| [GAS-5](#GAS-5-1693312253) | Use `ERC721A` instead `ERC721` | 1 | - |
| [GAS-6](#GAS-6-1693312386) | Don't initialize `bool` variable with default value | 2 | - |
| [GAS-7](#GAS-7-1694875070) | State variables can be packed into fewer storage slots | 3 | 6000 |
| [GAS-8](#GAS-8-1693313026) | The use of a logical AND in place of double if is slightly less gas efficient in instances where there isn't a corresponding else statement for the given if statement | 1 | 15 |
| [GAS-9](#GAS-9-1693313275) | Use assembly to validate `msg.sender` | 41 | 492 |
| [GAS-10](#GAS-10-1694794906) | Using mappings instead of arrays to avoid length checks save gas | 8 | - |
| [GAS-11](#GAS-11-1693313317) | Can make the variable outside the loop to save gas | 9 | - |
### <a name="GAS-1-1693311540"></a>[GAS-1] Multiple accesses of a mapping/array should use a local variable cache
The instances below point to the second+ access of a value inside a mapping/array, within a function. Caching a mapping's value in a local `storage` or `calldata` variable when the value is accessed [multiple times](https://gist.github.com/IllIllI000/ec23a57daa30a8f8ca8b9681c8ccefb0), saves **~42 gas per access** due to not having to recalculate the key's keccak256 hash (Gkeccak256 - **30 gas**) and that calculation's associated stack operations. Caching an array's struct avoids recalculating the array offsets into memory/calldata

*Instances (3)*:
<details><summary> see instances </summary>

```solidity
File: src/BranchPort.sol

// @audit isPortStrategy[_portStrategy]
397:         isPortStrategy[_portStrategy][_token] = !isPortStrategy[_portStrategy][_token];

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L397)

```solidity
File: src/RootBridgeAgent.sol

// @audit executionState[_srcChainId]
786:                 executionState[_srcChainId][_depositNonce] = STATUS_RETRIEVE;

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L786)

```solidity
File: src/RootPort.sol

// @audit isRouterApproved[_userAccount]
374:         isRouterApproved[_userAccount][_router] = !isRouterApproved[_userAccount][_router];

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L374)

</details>

### <a name="GAS-2-1693311663"></a>[GAS-2] Use assembly in place of `abi.decode` to extract `calldata` values more efficiently
Instead of using abi.decode, we can use assembly to decode our desired calldata values directly. This will allow us to avoid decoding calldata values that we will not use.
For more details on how to implement this, check the following [report](https://code4rena.com/reports/2023-05-juicebox#g-04-use-assembly-in-place-of-abidecode-to-extract-calldata-values-more-efficiently)

*Instances (1)*:
<details><summary> see instances </summary>

```solidity
File: src/ArbitrumCoreBranchRouter.sol

128:             (
                     address newBranchRouter,
                     address branchBridgeAgentFactory,
                     address rootBridgeAgent,
                     address rootBridgeAgentFactory,
                     address refundee,
                 ) = abi.decode(_data[1:], (address, address, address, address, address, GasParams));

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumCoreBranchRouter.sol#L128-L134)

</details>

### <a name="GAS-3-1693311695"></a>[GAS-3] Use `assembly` to write address storage values

*Instances (48)*:
<details><summary> see instances </summary>

```solidity
File: src/ArbitrumBranchPort.sol

42:         rootPortAddress = _rootPortAddress;

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol#L42-L42)

```solidity
File: src/BaseBranchRouter.sol

62:         localBridgeAgentAddress = _localBridgeAgentAddress;

63:         localPortAddress = IBridgeAgent(_localBridgeAgentAddress).localPortAddress();

64:         bridgeAgentExecutorAddress = IBridgeAgent(_localBridgeAgentAddress).bridgeAgentExecutorAddress();

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L64-L64)

```solidity
File: src/BranchBridgeAgent.sol

135:         rootBridgeAgentAddress = _rootBridgeAgentAddress;

136:         lzEndpointAddress = _lzEndpointAddress;

137:         localRouterAddress = _localRouterAddress;

138:         localPortAddress = _localPortAddress;

139:         bridgeAgentExecutorAddress = DeployBranchBridgeAgentExecutor.deploy();

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L139-L139)

```solidity
File: src/BranchPort.sol

129:         coreBranchRouterAddress = _coreBranchRouter;

334:         coreBranchRouterAddress = _newCoreRouter;

419:         coreBranchRouterAddress = _coreBranchRouter;

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L419-L419)

```solidity
File: src/CoreBranchRouter.sol

31:         hTokenFactoryAddress = _hTokenFactoryAddress;

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L31-L31)

```solidity
File: src/CoreRootRouter.sol

73:         rootPortAddress = _rootPortAddress;

86:         bridgeAgentAddress = payable(_bridgeAgentAddress);

87:         bridgeAgentExecutorAddress = IBridgeAgent(_bridgeAgentAddress).bridgeAgentExecutorAddress();

88:         hTokenFactoryAddress = _hTokenFactory;

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L88-L88)

```solidity
File: src/MulticallRootRouter.sol

97:         localPortAddress = _localPortAddress;

98:         multicallAddress = _multicallAddress;

112:         bridgeAgentAddress = payable(_bridgeAgentAddress);

113:         bridgeAgentExecutorAddress = IBridgeAgent(_bridgeAgentAddress).bridgeAgentExecutorAddress();

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L113-L113)

```solidity
File: src/RootBridgeAgent.sol

115:         factoryAddress = msg.sender;

117:         lzEndpointAddress = _lzEndpointAddress;

118:         localPortAddress = _localPortAddress;

119:         localRouterAddress = _localRouterAddress;

120:         bridgeAgentExecutorAddress = DeployRootBridgeAgentExecutor.deploy(address(this));

1181:         getBranchBridgeAgent[_branchChainId] = _newBranchBridgeAgent;

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1181-L1181)

```solidity
File: src/RootPort.sol

138:         coreRootRouterAddress = _coreRootRouter;

159:         coreRootBridgeAgentAddress = _coreRootBridgeAgent;

160:         localBranchPortAddress = _localBranchPortAddress;

162:         getBridgeAgentManager[_coreRootBridgeAgent] = owner();

386:         getBridgeAgentManager[_bridgeAgent] = _manager;

513:         coreRootRouterAddress = _coreRootRouter;

514:         coreRootBridgeAgentAddress = _coreRootBridgeAgent;

515:         getBridgeAgentManager[_coreRootBridgeAgent] = owner();

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L515-L515)

```solidity
File: src/VirtualAccount.sol

36:         userAddress = _userAddress;

37:         localPortAddress = _localPortAddress;

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L37-L37)

```solidity
File: src/factories/BranchBridgeAgentFactory.sol

72:         rootBridgeAgentFactoryAddress = _rootBridgeAgentFactoryAddress;

73:         lzEndpointAddress = _lzEndpointAddress;

74:         localCoreBranchRouterAddress = _localCoreBranchRouterAddress;

75:         localPortAddress = _localPortAddress;

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/BranchBridgeAgentFactory.sol#L75-L75)

```solidity
File: src/factories/ERC20hTokenBranchFactory.sol

47:         localPortAddress = _localPortAddress;

74:         localCoreRouterAddress = _coreRouter;

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenBranchFactory.sol#L74-L74)

```solidity
File: src/factories/ERC20hTokenRootFactory.sol

37:         rootPortAddress = _rootPortAddress;

51:         coreRootRouterAddress = _coreRouter;

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenRootFactory.sol#L51-L51)

```solidity
File: src/factories/RootBridgeAgentFactory.sol

35:         lzEndpointAddress = _lzEndpointAddress;

36:         rootPortAddress = _rootPortAddress;

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/RootBridgeAgentFactory.sol#L36-L36)

```solidity
File: src/token/ERC20hTokenRoot.sol

43:         factoryAddress = _factoryAddress;

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/token/ERC20hTokenRoot.sol#L43-L43)

</details>

### <a name="GAS-4-1694877823"></a>[GAS-4] Use `do while` loops instead of `for` loops
A `do while` loop will cost less gas since the condition is not being checked for the first iteration, Check my example on [github](https://github.com/he110-1/gasOptimization/blob/main/forToDoWhileOptimizationProof.sol). Actually, `do while` alwayse cast less gas compared to `For` check my second example [github](https://github.com/he110-1/gasOptimization/blob/main/forToDoWhileOptimizationProof2.sol)

*Instances (15)*:
<details><summary> see instances </summary>

```solidity
File: src/BaseBranchRouter.sol

192:         for (uint256 i = 0; i < _hTokens.length;) {

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L192)

```solidity
File: src/BranchBridgeAgent.sol

447:         for (uint256 i = 0; i < deposit.tokens.length;) {

513:         for (uint256 i = 0; i < numOfAssets;) {

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L513)

```solidity
File: src/BranchPort.sol

257:         for (uint256 i = 0; i < length;) {

305:         for (uint256 i = 0; i < length;) {

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L305)

```solidity
File: src/MulticallRootRouter.sol

278:             for (uint256 i = 0; i < outputParams.outputTokens.length;) {

367:             for (uint256 i = 0; i < outputParams.outputTokens.length;) {

455:             for (uint256 i = 0; i < outputParams.outputTokens.length;) {

557:         for (uint256 i = 0; i < outputTokens.length;) {

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L557)

```solidity
File: src/RootBridgeAgent.sol

318:         for (uint256 i = 0; i < settlement.hTokens.length;) {

399:         for (uint256 i = 0; i < length;) {

1070:         for (uint256 i = 0; i < hTokens.length;) {

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1070)

```solidity
File: src/RootBridgeAgentExecutor.sol

281:         for (uint256 i = 0; i < uint256(uint8(numOfAssets));) {

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L281)

```solidity
File: src/VirtualAccount.sol

70:         for (uint256 i = 0; i < length;) {

90:         for (uint256 i = 0; i < length;) {

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L90)

</details>

### <a name="GAS-5-1693312253"></a>[GAS-5] Use `ERC721A` instead `ERC721`
`ERC721A` is an improvement standard for `ERC721` tokens. It was proposed by the Azuki team and used for developing their NFT collection. Compared with `ERC721`, `ERC721A` is a more gas-efficient standard to mint a lot of of NFTs simultaneously. It allows developers to mint multiple NFTs at the same gas price. This has been a great improvement due to Ethereum’s sky-rocketing gas fee. Reference: https://nextrope.com/erc721-vs-erc721a-2/.

*Instances (1)*:
<details><summary> see instances </summary>

```solidity
File: src/VirtualAccount.sol

7: import {ERC721} from "solmate/tokens/ERC721.sol";

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L7)

</details>

### <a name="GAS-6-1693312386"></a>[GAS-6] Don't initialize `bool` variable with default value
The default value for variables with type bool is false, so initializing them to false is superfluous.

*Instances (2)*:
<details><summary> see instances </summary>

```solidity
File: src/CoreRootRouter.sol

85:         _setup = false;

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L85)

```solidity
File: src/RootPort.sol

133:         _setup = false;

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L133)

</details>

### <a name="GAS-7-1694875070"></a>[GAS-7] State variables can be packed into fewer storage slots
If variables occupying the same slot are both written the same function or by the constructor, avoids a separate Gsset (**20000 gas**). Reads of the variables can also be cheaper

*Instances (3)*:
<details><summary> see instances </summary>

```solidity
File: src/CoreRootRouter.sol

// @audit from 6 to 5 you need to change the structure elements order to: , uint256, address, address, address, address, bool
38: contract CoreRootRouter is IRootRouter, Ownable {
        /*///////////////////////////////////////////////////////////////
                        CORE ROOT ROUTER STATE
        //////////////////////////////////////////////////////////////*/
    
        /// @notice Boolean to indicate if the contract is in set up mode.
        bool internal _setup;
    
        /// @notice Root Chain Layer Zero Identifier.
        uint256 public immutable rootChainId;
    
        /// @notice Address for Local Port Address where funds deposited from this chain are kept
        ///         managed and supplied to different Port Strategies.
        address public immutable rootPortAddress;
    
        /// @notice Bridge Agent to manage remote execution and cross-chain assets.
        address payable public bridgeAgentAddress;
    
        /// @notice Bridge Agent Executor Address.
        address public bridgeAgentExecutorAddress;
    
        /// @notice ERC20 hToken Root Factory Address.
        address public hTokenFactoryAddress;
    
        /*///////////////////////////////////////////////////////////////
                                CONSTRUCTOR
        //////////////////////////////////////////////////////////////*/
    
        /**
         * @notice Constructor for Core Root Router.
         * @param _rootChainId layer zero root chain id.
         * @param _rootPortAddress address of the Root Port.
         */
        constructor(uint256 _rootChainId, address _rootPortAddress) {
            rootChainId = _rootChainId;
            rootPortAddress = _rootPortAddress;
    
            _initializeOwner(msg.sender);
            _setup = true;
        }
    
        /*///////////////////////////////////////////////////////////////
                        INITIALIZATION FUNCTIONS
        //////////////////////////////////////////////////////////////*/
    
        function initialize(address _bridgeAgentAddress, address _hTokenFactory) external onlyOwner {
            require(_setup, "Contract is already initialized");
            _setup = false;
            bridgeAgentAddress = payable(_bridgeAgentAddress);
            bridgeAgentExecutorAddress = IBridgeAgent(_bridgeAgentAddress).bridgeAgentExecutorAddress();
            hTokenFactoryAddress = _hTokenFactory;
        }
    
        /*///////////////////////////////////////////////////////////////
                        BRIDGE AGENT MANAGEMENT FUNCTIONS
        //////////////////////////////////////////////////////////////*/
    
        /**
         * @notice Add a new Chain (Branch Bridge Agent and respective Router) to a Root Bridge Agent.
         * @param _branchBridgeAgentFactory Address of the branch Bridge Agent Factory.
         * @param _newBranchRouter Address of the new branch router.
         * @param _refundee Address of the excess gas receiver.
         * @param _dstChainId Chain Id of the branch chain where the new Bridge Agent will be deployed.
         * @param _gParams Gas parameters for remote execution.
         */
        function addBranchToBridgeAgent(
            address _rootBridgeAgent,
            address _branchBridgeAgentFactory,
            address _newBranchRouter,
            address _refundee,
            uint16 _dstChainId,
            GasParams[2] calldata _gParams
        ) external payable {
            // Check if msg.sender is the Bridge Agent Manager
            if (msg.sender != IPort(rootPortAddress).getBridgeAgentManager(_rootBridgeAgent)) {
                revert UnauthorizedCallerNotManager();
            }
    
            // Check if valid chain
            if (!IPort(rootPortAddress).isChainId(_dstChainId)) revert InvalidChainId();
    
            // Check if chain already added to bridge agent
            if (IBridgeAgent(_rootBridgeAgent).getBranchBridgeAgent(_dstChainId) != address(0)) revert InvalidChainId();
    
            // Check if Branch Bridge Agent is allowed by Root Bridge Agent
            if (!IBridgeAgent(_rootBridgeAgent).isBranchBridgeAgentAllowed(_dstChainId)) revert UnauthorizedChainId();
    
            // Encode CallData
            bytes memory params = abi.encode(
                _newBranchRouter,
                _branchBridgeAgentFactory,
                _rootBridgeAgent,
                IBridgeAgent(_rootBridgeAgent).factoryAddress(),
                _refundee,
                _gParams[1]
            );
    
            // Pack funcId into data
            bytes memory payload = abi.encodePacked(bytes1(0x02), params);
    
            //Add new global token to branch chain
            IBridgeAgent(bridgeAgentAddress).callOut{value: msg.value}(
                payable(_refundee), _refundee, _dstChainId, payload, _gParams[0]
            );
        }
    
        /*///////////////////////////////////////////////////////////////
                    GOVERNANCE / ADMIN EXTERNAL FUNCTIONS
        ///////////////////////////////////////////////////////////////*/
    
        /**
         * @notice Add or Remove a Branch Bridge Agent Factory.
         * @param _rootBridgeAgentFactory Address of the root Bridge Agent Factory.
         * @param _branchBridgeAgentFactory Address of the branch Bridge Agent Factory.
         * @param _refundee Receiver of any leftover execution gas upon reaching the destination network.
         * @param _dstChainId Chain Id of the branch chain where the new Bridge Agent will be deployed.
         * @param _gParams Gas parameters for remote execution.
         */
        function toggleBranchBridgeAgentFactory(
            address _rootBridgeAgentFactory,
            address _branchBridgeAgentFactory,
            address _refundee,
            uint16 _dstChainId,
            GasParams calldata _gParams
        ) external payable onlyOwner {
            if (!IPort(rootPortAddress).isBridgeAgentFactory(_rootBridgeAgentFactory)) {
                revert UnrecognizedBridgeAgentFactory();
            }
    
            // Encode CallData
            bytes memory params = abi.encode(_branchBridgeAgentFactory);
    
            // Pack funcId into data
            bytes memory payload = abi.encodePacked(bytes1(0x03), params);
    
            //Add new global token to branch chain
            IBridgeAgent(bridgeAgentAddress).callOut{value: msg.value}(
                payable(_refundee), _refundee, _dstChainId, payload, _gParams
            );
        }
    
        /**
         * @notice Remove a Branch Bridge Agent.
         * @param _branchBridgeAgent Address of the Branch Bridge Agent to be updated.
         * @param _refundee Receiver of any leftover execution gas upon reaching destination network.
         * @param _dstChainId Chain Id of the branch chain where the new Bridge Agent will be deployed.
         * @param _gParams Gas parameters for remote execution.
         */
        function removeBranchBridgeAgent(
            address _branchBridgeAgent,
            address _refundee,
            uint16 _dstChainId,
            GasParams calldata _gParams
        ) external payable onlyOwner {
            //Encode CallData
            bytes memory params = abi.encode(_branchBridgeAgent);
    
            // Pack funcId into data
            bytes memory payload = abi.encodePacked(bytes1(0x04), params);
    
            //Add new global token to branch chain
            IBridgeAgent(bridgeAgentAddress).callOut{value: msg.value}(
                payable(_refundee), _refundee, _dstChainId, payload, _gParams
            );
        }
    
        /**
         * @notice Add or Remove a Strategy Token.
         * @param _underlyingToken Address of the underlying token to be added for use in Branch strategies.
         * @param _minimumReservesRatio Minimum Branch Port reserves ratio for the underlying token.
         * @param _refundee Receiver of any leftover execution gas upon reaching destination network.
         * @param _dstChainId Chain Id of the branch chain where the new Bridge Agent will be deployed.
         * @param _gParams Gas parameters for remote execution.
         */
        function manageStrategyToken(
            address _underlyingToken,
            uint256 _minimumReservesRatio,
            address _refundee,
            uint16 _dstChainId,
            GasParams calldata _gParams
        ) external payable onlyOwner {
            // Encode CallData
            bytes memory params = abi.encode(_underlyingToken, _minimumReservesRatio);
    
            // Pack funcId into data
            bytes memory payload = abi.encodePacked(bytes1(0x05), params);
    
            //Add new global token to branch chain
            IBridgeAgent(bridgeAgentAddress).callOut{value: msg.value}(
                payable(_refundee), _refundee, _dstChainId, payload, _gParams
            );
        }
    
        /**
         * @notice Add, Remove or update a Port Strategy.
         * @param _portStrategy Address of the Port Strategy to be added for use in Branch strategies.
         * @param _underlyingToken Address of the underlying token to be added for use in Branch strategies.
         * @param _dailyManagementLimit Daily management limit of the given token for the Port Strategy.
         * @param _isUpdateDailyLimit Boolean to safely indicate if the Port Strategy is being updated and not deactivated.
         * @param _refundee Receiver of any leftover execution gas upon reaching destination network.
         * @param _dstChainId Chain Id of the branch chain where the new Bridge Agent will be deployed.
         * @param _gParams Gas parameters for remote execution.
         */
        function managePortStrategy(
            address _portStrategy,
            address _underlyingToken,
            uint256 _dailyManagementLimit,
            bool _isUpdateDailyLimit,
            address _refundee,
            uint16 _dstChainId,
            GasParams calldata _gParams
        ) external payable onlyOwner {
            // Encode CallData
            bytes memory params = abi.encode(_portStrategy, _underlyingToken, _dailyManagementLimit, _isUpdateDailyLimit);
    
            // Pack funcId into data
            bytes memory payload = abi.encodePacked(bytes1(0x06), params);
    
            //Add new global token to branch chain
            IBridgeAgent(bridgeAgentAddress).callOut{value: msg.value}(
                payable(_refundee), _refundee, _dstChainId, payload, _gParams
            );
        }
    
        /**
         * @notice Set the Core Branch Router and Bridge Agent.
         * @param _refundee Receiver of any leftover execution gas upon reaching destination network.
         * @param _coreBranchRouter Address of the Core Branch Router.
         * @param _coreBranchBridgeAgent Address of the Core Branch Bridge Agent.
         * @param _dstChainId Chain Id of the branch chain where the new Bridge Agent will be deployed.
         * @param _gParams Gas parameters for remote execution.
         */
        function setCoreBranch(
            address _refundee,
            address _coreBranchRouter,
            address _coreBranchBridgeAgent,
            uint16 _dstChainId,
            GasParams calldata _gParams
        ) external payable {
            // Check caller is root port
            require(msg.sender == rootPortAddress, "Only root port can call");
    
            // Encode CallData
            bytes memory params = abi.encode(_coreBranchRouter, _coreBranchBridgeAgent);
    
            // Pack funcId into data
            bytes memory payload = abi.encodePacked(bytes1(0x07), params);
    
            //Add new global token to branch chain
            IBridgeAgent(bridgeAgentAddress).callOut{value: msg.value}(
                payable(_refundee), _refundee, _dstChainId, payload, _gParams
            );
        }
    
        /*///////////////////////////////////////////////////////////////
                            LAYERZERO FUNCTIONS
        ///////////////////////////////////////////////////////////////*/
    
        /// @inheritdoc IRootRouter
        function executeResponse(bytes calldata _encodedData, uint16 _srcChainId)
            external
            payable
            override
            requiresExecutor
        {
            // Parse funcId
            bytes1 funcId = _encodedData[0];
    
            ///  FUNC ID: 2 (_addLocalToken)
            if (funcId == 0x02) {
                (address underlyingAddress, address localAddress, string memory name, string memory symbol, uint8 decimals)
                = abi.decode(_encodedData[1:], (address, address, string, string, uint8));
    
                _addLocalToken(underlyingAddress, localAddress, name, symbol, decimals, _srcChainId);
    
                /// FUNC ID: 3 (_setLocalToken)
            } else if (funcId == 0x03) {
                (address globalAddress, address localAddress) = abi.decode(_encodedData[1:], (address, address));
    
                _setLocalToken(globalAddress, localAddress, _srcChainId);
    
                /// FUNC ID: 4 (_syncBranchBridgeAgent)
            } else if (funcId == 0x04) {
                (address newBranchBridgeAgent, address rootBridgeAgent) = abi.decode(_encodedData[1:], (address, address));
    
                _syncBranchBridgeAgent(newBranchBridgeAgent, rootBridgeAgent, _srcChainId);
    
                /// Unrecognized Function Selector
            } else {
                revert UnrecognizedFunctionId();
            }
        }
    
        /// @inheritdoc IRootRouter
        function execute(bytes calldata _encodedData, uint16) external payable override requiresExecutor {
            // Parse funcId
            bytes1 funcId = _encodedData[0];
    
            /// FUNC ID: 1 (_addGlobalToken)
            if (funcId == 0x01) {
                (address refundee, address globalAddress, uint16 dstChainId, GasParams[2] memory gasParams) =
                    abi.decode(_encodedData[1:], (address, address, uint16, GasParams[2]));
    
                _addGlobalToken(refundee, globalAddress, dstChainId, gasParams);
    
                /// Unrecognized Function Selector
            } else {
                revert UnrecognizedFunctionId();
            }
        }
    
        /// @inheritdoc IRootRouter
        function executeDepositSingle(bytes memory, DepositParams memory, uint16)
            external
            payable
            override
            requiresExecutor
        {
            revert();
        }
    
        /// @inheritdoc IRootRouter
        function executeDepositMultiple(bytes calldata, DepositMultipleParams memory, uint16)
            external
            payable
            override
            requiresExecutor
        {
            revert();
        }
    
        /// @inheritdoc IRootRouter
        function executeSigned(bytes memory, address, uint16) external payable override requiresExecutor {
            revert();
        }
    
        /// @inheritdoc IRootRouter
        function executeSignedDepositSingle(bytes memory, DepositParams memory, address, uint16)
            external
            payable
            override
            requiresExecutor
        {
            revert();
        }
    
        /// @inheritdoc IRootRouter
        function executeSignedDepositMultiple(bytes memory, DepositMultipleParams memory, address, uint16)
            external
            payable
            override
            requiresExecutor
        {
            revert();
        }
    
        /*///////////////////////////////////////////////////////////////
                        TOKEN MANAGEMENT INTERNAL FUNCTIONS
        ////////////////////////////////////////////////////////////*/
    
        /**
         * @notice Internal function to add a global token to a specific chain. Must be called from a branch.
         *   @param _refundee Address of the excess gas receiver.
         *   @param _globalAddress global token to be added.
         *   @param _dstChainId chain to which the Global Token will be added.
         *   @param _gParams Gas parameters for remote execution.
         *
         */
        function _addGlobalToken(
            address _refundee,
            address _globalAddress,
            uint16 _dstChainId,
            GasParams[2] memory _gParams
        ) internal {
            if (_dstChainId == rootChainId) revert InvalidChainId();
    
            if (!IPort(rootPortAddress).isGlobalAddress(_globalAddress)) {
                revert UnrecognizedGlobalToken();
            }
    
            // Verify that it does not exist
            if (IPort(rootPortAddress).isGlobalToken(_globalAddress, _dstChainId)) {
                revert TokenAlreadyAdded();
            }
    
            // Encode CallData
            bytes memory params = abi.encode(
                _globalAddress,
                ERC20(_globalAddress).name(),
                ERC20(_globalAddress).symbol(),
                ERC20(_globalAddress).decimals(),
                _refundee,
                _gParams[1]
            );
    
            // Pack funcId into data
            bytes memory payload = abi.encodePacked(bytes1(0x01), params);
    
            //Add new global token to branch chain
            IBridgeAgent(bridgeAgentAddress).callOut{value: msg.value}(
                payable(_refundee), _refundee, _dstChainId, payload, _gParams[0]
            );
        }
    
        /**
         * @notice Function to add a new local to the global environment. Called from branch chain.
         *   @param _underlyingAddress the token's underlying/native chain address.
         *   @param _localAddress the token's address.
         *   @param _name the token's name.
         *   @param _symbol the token's symbol.
         *   @param _decimals the token's decimals.
         *   @param _srcChainId the token's origin chain Id.
         *
         */
        function _addLocalToken(
            address _underlyingAddress,
            address _localAddress,
            string memory _name,
            string memory _symbol,
            uint8 _decimals,
            uint16 _srcChainId
        ) internal {
            // Verify if the underlying address is already known by the branch or root chain
            if (IPort(rootPortAddress).isGlobalAddress(_underlyingAddress)) revert TokenAlreadyAdded();
            if (IPort(rootPortAddress).isLocalToken(_underlyingAddress, _srcChainId)) revert TokenAlreadyAdded();
            if (IPort(rootPortAddress).isUnderlyingToken(_underlyingAddress, _srcChainId)) revert TokenAlreadyAdded();
    
            //Create a new global token
            address newToken = address(IFactory(hTokenFactoryAddress).createToken(_name, _symbol, _decimals));
    
            // Update Registry
            IPort(rootPortAddress).setAddresses(
                newToken, (_srcChainId == rootChainId) ? newToken : _localAddress, _underlyingAddress, _srcChainId
            );
        }
    
        /**
         * @notice Internal function to set the local token on a specific chain for a global token.
         *   @param _globalAddress global token to be updated.
         *   @param _localAddress local token to be added.
         *   @param _dstChainId local token's chain.
         *
         */
        function _setLocalToken(address _globalAddress, address _localAddress, uint16 _dstChainId) internal {
            // Verify if the token already added
            if (IPort(rootPortAddress).isLocalToken(_localAddress, _dstChainId)) revert TokenAlreadyAdded();
    
            // Set the global token's new branch chain address
            IPort(rootPortAddress).setLocalAddress(_globalAddress, _localAddress, _dstChainId);
        }
    
        /*///////////////////////////////////////////////////////////////
                    BRIDGE AGENT MANAGEMENT INTERNAL FUNCTIONS
        ////////////////////////////////////////////////////////////*/
    
        /**
         * @dev Internal function sync a Root Bridge Agent with a newly created BRanch Bridge Agent.
         *   @param _newBranchBridgeAgent new branch bridge agent address
         *   @param _rootBridgeAgent new branch bridge agent address
         *   @param _srcChainId branch chain id.
         *
         */
        function _syncBranchBridgeAgent(address _newBranchBridgeAgent, address _rootBridgeAgent, uint256 _srcChainId)
            internal
        {
            IPort(rootPortAddress).syncBranchBridgeAgentWithRoot(_newBranchBridgeAgent, _rootBridgeAgent, _srcChainId);
        }
    
        /*///////////////////////////////////////////////////////////////
                                 MODIFIERS
        ///////////////////////////////////////////////////////////////*/
    
        /// @notice Modifier verifies the caller is the Bridge Agent Executor.
        modifier requiresExecutor() {
            if (msg.sender != bridgeAgentExecutorAddress) revert UnrecognizedBridgeAgentExecutor();
            _;
        }
    
        /*///////////////////////////////////////////////////////////////
                                    ERRORS
        ///////////////////////////////////////////////////////////////*/
    
        error InvalidChainId();
    
        error UnauthorizedChainId();
    
        error UnauthorizedCallerNotManager();
    
        error TokenAlreadyAdded();
    
        error UnrecognizedGlobalToken();
    
        error UnrecognizedBridgeAgentFactory();
    }

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L38-L531)

```solidity
File: src/RootBridgeAgent.sol

// @audit from 12 to 11 you need to change the structure elements order to: , mapping, mapping, mapping, mapping, mapping, uint256, address, address, address, address, address, uint32, uint16
32: contract RootBridgeAgent is IRootBridgeAgent, BridgeAgentConstants {
        using SafeTransferLib for address;
        using ExcessivelySafeCall for address;
        /*///////////////////////////////////////////////////////////////
                            ROOT BRIDGE AGENT STATE
        //////////////////////////////////////////////////////////////*/
    
        /// @notice Local Chain Id
        uint16 public immutable localChainId;
    
        /// @notice Bridge Agent Factory Address.
        address public immutable factoryAddress;
    
        /// @notice Local Core Root Router Address
        address public immutable localRouterAddress;
    
        /// @notice Local Port Address where funds deposited from this chain are stored.
        address public immutable localPortAddress;
    
        /// @notice Local Layer Zero Endpoint Address for cross-chain communication.
        address public immutable lzEndpointAddress;
    
        /// @notice Address of Root Bridge Agent Executor.
        address public immutable bridgeAgentExecutorAddress;
    
        /*///////////////////////////////////////////////////////////////
                            BRANCH BRIDGE AGENTS STATE
        //////////////////////////////////////////////////////////////*/
    
        /// @notice Chain -> Branch Bridge Agent Address. For N chains, each Root Bridge Agent Address has M =< N Branch Bridge Agent Address.
        mapping(uint256 chainId => address branchBridgeAgent) public getBranchBridgeAgent;
    
        /// @notice Message Path for each connected Branch Bridge Agent as bytes for Layzer Zero interaction = localAddress + destinationAddress abi.encodePacked()
        mapping(uint256 chainId => bytes branchBridgeAgentPath) public getBranchBridgeAgentPath;
    
        /// @notice If true, bridge agent manager has allowed for a new given branch bridge agent to be synced/added.
        mapping(uint256 chainId => bool allowed) public isBranchBridgeAgentAllowed;
    
        /*///////////////////////////////////////////////////////////////
                                SETTLEMENTS STATE
        //////////////////////////////////////////////////////////////*/
    
        /// @notice Deposit nonce used for identifying transaction.
        uint32 public settlementNonce;
    
        /// @notice Mapping from Settlement nonce to Settlement Struct.
        mapping(uint256 nonce => Settlement settlementInfo) public getSettlement;
    
        /*///////////////////////////////////////////////////////////////
                                EXECUTOR STATE
        //////////////////////////////////////////////////////////////*/
    
        /// @notice If true, bridge agent has already served a request with this nonce from  a given chain. Chain -> Nonce -> Bool
        mapping(uint256 chainId => mapping(uint256 nonce => uint256 state)) public executionState;
    
        /*///////////////////////////////////////////////////////////////
                                REENTRANCY STATE
        //////////////////////////////////////////////////////////////*/
    
        /// @notice Re-entrancy lock modifier state.
        uint256 internal _unlocked = 1;
    
        /*///////////////////////////////////////////////////////////////
                                CONSTRUCTOR
        //////////////////////////////////////////////////////////////*/
    
        /**
         * @notice Constructor for Bridge Agent.
         *     @param _localChainId Local Chain Id.
         *     @param _lzEndpointAddress Local Layerzero Endpoint Address.
         *     @param _localPortAddress Local Port Address.
         *     @param _localRouterAddress Local Port Address.
         */
        constructor(
            uint16 _localChainId,
            address _lzEndpointAddress,
            address _localPortAddress,
            address _localRouterAddress
        ) {
            require(_lzEndpointAddress != address(0), "Layerzero Enpoint Address cannot be zero address");
            require(_localPortAddress != address(0), "Port Address cannot be zero address");
            require(_localRouterAddress != address(0), "Router Address cannot be zero address");
    
            factoryAddress = msg.sender;
            localChainId = _localChainId;
            lzEndpointAddress = _lzEndpointAddress;
            localPortAddress = _localPortAddress;
            localRouterAddress = _localRouterAddress;
            bridgeAgentExecutorAddress = DeployRootBridgeAgentExecutor.deploy(address(this));
            settlementNonce = 1;
        }
    
        /*///////////////////////////////////////////////////////////////
                            FALLBACK FUNCTIONS
        //////////////////////////////////////////////////////////////*/
    
        receive() external payable {}
    
        /*///////////////////////////////////////////////////////////////
                            VIEW EXTERNAL FUNCTIONS
        //////////////////////////////////////////////////////////////*/
    
        /// @inheritdoc IRootBridgeAgent
        function getSettlementEntry(uint32 _settlementNonce) external view override returns (Settlement memory) {
            return getSettlement[_settlementNonce];
        }
    
        /// @inheritdoc IRootBridgeAgent
        function getFeeEstimate(
            uint256 _gasLimit,
            uint256 _remoteBranchExecutionGas,
            bytes calldata _payload,
            uint16 _dstChainId
        ) external view returns (uint256 _fee) {
            (_fee,) = ILayerZeroEndpoint(lzEndpointAddress).estimateFees(
                _dstChainId,
                address(this),
                _payload,
                false,
                abi.encodePacked(uint16(2), _gasLimit, _remoteBranchExecutionGas, getBranchBridgeAgent[_dstChainId])
            );
        }
    
        /*///////////////////////////////////////////////////////////////
                        ROOT ROUTER EXTERNAL FUNCTIONS
        //////////////////////////////////////////////////////////////*/
    
        /// @inheritdoc IRootBridgeAgent
        function callOut(
            address payable _refundee,
            address _recipient,
            uint16 _dstChainId,
            bytes calldata _params,
            GasParams calldata _gParams
        ) external payable override lock requiresRouter {
            //Encode Data for call.
            bytes memory payload = abi.encodePacked(bytes1(0x00), _recipient, settlementNonce++, _params);
    
            //Perform Call to clear hToken balance on destination branch chain.
            _performCall(_dstChainId, _refundee, payload, _gParams);
        }
    
        /// @inheritdoc IRootBridgeAgent
        function callOutAndBridge(
            address payable _refundee,
            address _recipient,
            uint16 _dstChainId,
            bytes calldata _params,
            SettlementInput calldata _sParams,
            GasParams calldata _gParams,
            bool _hasFallbackToggled
        ) external payable override lock requiresRouter {
            // Create Settlement and Perform call
            bytes memory payload = _createSettlement(
                settlementNonce,
                _refundee,
                _recipient,
                _dstChainId,
                _params,
                _sParams.globalAddress,
                _sParams.amount,
                _sParams.deposit,
                _hasFallbackToggled
            );
    
            //Perform Call.
            _performCall(_dstChainId, _refundee, payload, _gParams);
        }
    
        /// @inheritdoc IRootBridgeAgent
        function callOutAndBridgeMultiple(
            address payable _refundee,
            address _recipient,
            uint16 _dstChainId,
            bytes calldata _params,
            SettlementMultipleInput calldata _sParams,
            GasParams calldata _gParams,
            bool _hasFallbackToggled
        ) external payable override lock requiresRouter {
            // Create Settlement and Perform call
            bytes memory payload = _createSettlementMultiple(
                settlementNonce,
                _refundee,
                _recipient,
                _dstChainId,
                _sParams.globalAddresses,
                _sParams.amounts,
                _sParams.deposits,
                _params,
                _hasFallbackToggled
            );
    
            // Perform Call to destination Branch Chain.
            _performCall(_dstChainId, _refundee, payload, _gParams);
        }
    
        /*///////////////////////////////////////////////////////////////
                        SETTLEMENT EXTERNAL FUNCTIONS
        //////////////////////////////////////////////////////////////*/
    
        /// @inheritdoc IRootBridgeAgent
        function retrySettlement(
            uint32 _settlementNonce,
            address _recipient,
            bytes calldata _params,
            GasParams calldata _gParams,
            bool _hasFallbackToggled
        ) external payable override lock {
            // Get storage reference
            Settlement storage settlementReference = getSettlement[_settlementNonce];
    
            // Check if Settlement hasn't been redeemed.
            if (settlementReference.owner == address(0)) revert SettlementRetryUnavailable();
    
            // Check if caller is Settlement owner
            if (msg.sender != settlementReference.owner) {
                if (msg.sender != address(IPort(localPortAddress).getUserAccount(settlementReference.owner))) {
                    revert NotSettlementOwner();
                }
            }
    
            // Update Settlement Status
            settlementReference.status = STATUS_SUCCESS;
    
            // Perform Settlement Retry
            _performRetrySettlementCall(
                _hasFallbackToggled,
                settlementReference.hTokens,
                settlementReference.tokens,
                settlementReference.amounts,
                settlementReference.deposits,
                _params,
                _settlementNonce,
                payable(settlementReference.owner),
                _recipient,
                settlementReference.dstChainId,
                _gParams,
                msg.value
            );
        }
    
        /// @inheritdoc IRootBridgeAgent
        function retrieveSettlement(uint32 _settlementNonce, GasParams calldata _gParams) external payable lock {
            //Get settlement storage reference
            Settlement storage settlementReference = getSettlement[_settlementNonce];
    
            // Get Settlement owner.
            address settlementOwner = settlementReference.owner;
    
            // Check if Settlement is Retrieve.
            if (settlementOwner == address(0)) revert SettlementRetrieveUnavailable();
    
            // Check Settlement Owner
            if (msg.sender != settlementOwner) {
                if (msg.sender != address(IPort(localPortAddress).getUserAccount(settlementOwner))) {
                    revert NotSettlementOwner();
                }
            }
    
            //Encode Data for cross-chain call.
            bytes memory payload = abi.encodePacked(bytes1(0x03), settlementOwner, _settlementNonce);
    
            //Retrieve Deposit
            _performCall(settlementReference.dstChainId, payable(settlementOwner), payload, _gParams);
        }
    
        /// @inheritdoc IRootBridgeAgent
        function redeemSettlement(uint32 _settlementNonce) external override lock {
            // Get setttlement storage reference
            Settlement storage settlement = getSettlement[_settlementNonce];
    
            // Get deposit owner.
            address settlementOwner = settlement.owner;
    
            // Check if Settlement is redeemable.
            if (settlement.status == STATUS_SUCCESS) revert SettlementRedeemUnavailable();
            if (settlementOwner == address(0)) revert SettlementRedeemUnavailable();
    
            // Check if Settlement Owner is msg.sender or msg.sender is the virtual account of the settlement owner.
            if (msg.sender != settlementOwner) {
                if (msg.sender != address(IPort(localPortAddress).getUserAccount(settlementOwner))) {
                    revert NotSettlementOwner();
                }
            }
    
            // Clear Global hTokens To Recipient on Root Chain cancelling Settlement to Branch
            for (uint256 i = 0; i < settlement.hTokens.length;) {
                // Save to memory
                address _hToken = settlement.hTokens[i];
    
                // Check if asset
                if (_hToken != address(0)) {
                    // Save to memory
                    uint24 _dstChainId = settlement.dstChainId;
    
                    // Move hTokens from Branch to Root + Mint Sufficient hTokens to match new port deposit
                    IPort(localPortAddress).bridgeToRoot(
                        msg.sender,
                        IPort(localPortAddress).getGlobalTokenFromLocal(_hToken, _dstChainId),
                        settlement.amounts[i],
                        settlement.deposits[i],
                        _dstChainId
                    );
                }
    
                unchecked {
                    ++i;
                }
            }
    
            // Delete Settlement
            delete getSettlement[_settlementNonce];
        }
    
        /*///////////////////////////////////////////////////////////////
                        TOKEN MANAGEMENT EXTERNAL FUNCTIONS
        //////////////////////////////////////////////////////////////*/
    
        /// @inheritdoc IRootBridgeAgent
        function bridgeIn(address _recipient, DepositParams memory _dParams, uint256 _srcChainId)
            public
            override
            requiresAgentExecutor
        {
            // Deposit can't be greater than amount.
            if (_dParams.amount < _dParams.deposit) revert InvalidInputParams();
    
            // Cache local port address
            address _localPortAddress = localPortAddress;
    
            // Check local exists.
            if (_dParams.amount > 0) {
                if (!IPort(_localPortAddress).isLocalToken(_dParams.hToken, _srcChainId)) {
                    revert InvalidInputParams();
                }
            }
    
            // Check underlying exists.
            if (_dParams.deposit > 0) {
                if (IPort(_localPortAddress).getLocalTokenFromUnderlying(_dParams.token, _srcChainId) != _dParams.hToken) {
                    revert InvalidInputParams();
                }
            }
    
            // Move hTokens from Branch to Root + Mint Sufficient hTokens to match new port deposit
            IPort(_localPortAddress).bridgeToRoot(
                _recipient,
                IPort(_localPortAddress).getGlobalTokenFromLocal(_dParams.hToken, _srcChainId),
                _dParams.amount,
                _dParams.deposit,
                _srcChainId
            );
        }
    
        /// @inheritdoc IRootBridgeAgent
        function bridgeInMultiple(address _recipient, DepositMultipleParams calldata _dParams, uint256 _srcChainId)
            external
            override
            requiresAgentExecutor
        {
            // Cache length
            uint256 length = _dParams.hTokens.length;
    
            // Check MAX_LENGTH
            if (length > MAX_TOKENS_LENGTH) revert InvalidInputParams();
    
            // Bridge in assets
            for (uint256 i = 0; i < length;) {
                bridgeIn(
                    _recipient,
                    DepositParams({
                        hToken: _dParams.hTokens[i],
                        token: _dParams.tokens[i],
                        amount: _dParams.amounts[i],
                        deposit: _dParams.deposits[i],
                        depositNonce: 0
                    }),
                    _srcChainId
                );
    
                unchecked {
                    ++i;
                }
            }
        }
    
        /*///////////////////////////////////////////////////////////////
                        LAYER ZERO EXTERNAL FUNCTIONS
        //////////////////////////////////////////////////////////////*/
    
        /// @inheritdoc ILayerZeroReceiver
        function lzReceive(uint16 _srcChainId, bytes calldata _srcAddress, uint64, bytes calldata _payload) public {
            (bool success,) = address(this).excessivelySafeCall(
                gasleft(),
                150,
                abi.encodeWithSelector(this.lzReceiveNonBlocking.selector, msg.sender, _srcChainId, _srcAddress, _payload)
            );
    
            if (!success) if (msg.sender == getBranchBridgeAgent[localChainId]) revert ExecutionFailure();
        }
    
        /// @inheritdoc IRootBridgeAgent
        function lzReceiveNonBlocking(
            address _endpoint,
            uint16 _srcChainId,
            bytes calldata _srcAddress,
            bytes calldata _payload
        ) public override requiresEndpoint(_endpoint, _srcChainId, _srcAddress) {
            // Deposit Nonce
            uint32 nonce;
    
            // DEPOSIT FLAG: 0 (System request / response)
            if (_payload[0] == 0x00) {
                // Parse deposit nonce
                nonce = uint32(bytes4(_payload[PARAMS_START:PARAMS_TKN_START]));
    
                // Check if tx has already been executed
                if (executionState[_srcChainId][nonce] != STATUS_READY) {
                    revert AlreadyExecutedTransaction();
                }
    
                // Avoid stack too deep
                uint16 srcChainId = _srcChainId;
    
                // Try to execute remote request
                // Flag 0 - RootBridgeAgentExecutor(bridgeAgentExecutorAddress).executeSystemRequest(_localRouterAddress, _payload, _srcChainId)
                _execute(
                    nonce,
                    abi.encodeWithSelector(
                        RootBridgeAgentExecutor.executeSystemRequest.selector, localRouterAddress, _payload, srcChainId
                    ),
                    srcChainId
                );
    
                // DEPOSIT FLAG: 1 (Call without Deposit)
            } else if (_payload[0] == 0x01) {
                // Parse Deposit Nonce
                nonce = uint32(bytes4(_payload[PARAMS_START:PARAMS_TKN_START]));
    
                // Check if tx has already been executed
                if (executionState[_srcChainId][nonce] != STATUS_READY) {
                    revert AlreadyExecutedTransaction();
                }
    
                // Avoid stack too deep
                uint16 srcChainId = _srcChainId;
    
                // Try to execute remote request
                // Flag 1 - RootBridgeAgentExecutor(bridgeAgentExecutorAddress).executeNoDeposit(localRouterAddress, payload, _srcChainId)
                _execute(
                    nonce,
                    abi.encodeWithSelector(
                        RootBridgeAgentExecutor.executeNoDeposit.selector, localRouterAddress, _payload, srcChainId
                    ),
                    srcChainId
                );
    
                // DEPOSIT FLAG: 2 (Call with Deposit)
            } else if (_payload[0] == 0x02) {
                //Parse Deposit Nonce
                nonce = uint32(bytes4(_payload[PARAMS_START:PARAMS_TKN_START]));
    
                //Check if tx has already been executed
                if (executionState[_srcChainId][nonce] != STATUS_READY) {
                    revert AlreadyExecutedTransaction();
                }
    
                // Avoid stack too deep
                uint16 srcChainId = _srcChainId;
    
                // Try to execute remote request
                // Flag 2 - RootBridgeAgentExecutor(bridgeAgentExecutorAddress).executeWithDeposit(localRouterAddress, _payload, _srcChainId)
                _execute(
                    nonce,
                    abi.encodeWithSelector(
                        RootBridgeAgentExecutor.executeWithDeposit.selector, localRouterAddress, _payload, srcChainId
                    ),
                    srcChainId
                );
    
                // DEPOSIT FLAG: 3 (Call with multiple asset Deposit)
            } else if (_payload[0] == 0x03) {
                // Parse deposit nonce
                nonce = uint32(bytes4(_payload[2:6]));
    
                // Check if tx has already been executed
                if (executionState[_srcChainId][nonce] != STATUS_READY) {
                    revert AlreadyExecutedTransaction();
                }
    
                // Avoid stack too deep
                uint16 srcChainId = _srcChainId;
    
                // Try to execute remote request
                // Flag 3 - RootBridgeAgentExecutor(bridgeAgentExecutorAddress).executeWithDepositMultiple(localRouterAddress, _payload, _srcChainId)
                _execute(
                    nonce,
                    abi.encodeWithSelector(
                        RootBridgeAgentExecutor.executeWithDepositMultiple.selector,
                        localRouterAddress,
                        _payload,
                        srcChainId
                    ),
                    srcChainId
                );
    
                // DEPOSIT FLAG: 4 (Call without Deposit + msg.sender)
            } else if (_payload[0] == 0x04) {
                // Parse deposit nonce
                nonce = uint32(bytes4(_payload[PARAMS_START_SIGNED:PARAMS_TKN_START_SIGNED]));
    
                //Check if tx has already been executed
                if (executionState[_srcChainId][nonce] != STATUS_READY) {
                    revert AlreadyExecutedTransaction();
                }
    
                // Get User Virtual Account
                VirtualAccount userAccount = IPort(localPortAddress).fetchVirtualAccount(
                    address(uint160(bytes20(_payload[PARAMS_START:PARAMS_START_SIGNED])))
                );
    
                // Toggle Router Virtual Account use for tx execution
                IPort(localPortAddress).toggleVirtualAccountApproved(userAccount, localRouterAddress);
    
                // Avoid stack too deep
                uint16 srcChainId = _srcChainId;
    
                // Try to execute remote request
                // Flag 4 - RootBridgeAgentExecutor(bridgeAgentExecutorAddress).executeSignedNoDeposit(address(userAccount), localRouterAddress, data, _srcChainId
                _execute(
                    nonce,
                    abi.encodeWithSelector(
                        RootBridgeAgentExecutor.executeSignedNoDeposit.selector,
                        address(userAccount),
                        localRouterAddress,
                        _payload,
                        srcChainId
                    ),
                    srcChainId
                );
    
                // Toggle Router Virtual Account use for tx execution
                IPort(localPortAddress).toggleVirtualAccountApproved(userAccount, localRouterAddress);
    
                //DEPOSIT FLAG: 5 (Call with Deposit + msg.sender)
            } else if (_payload[0] & 0x7F == 0x05) {
                // Parse deposit nonce
                nonce = uint32(bytes4(_payload[PARAMS_START_SIGNED:PARAMS_TKN_START_SIGNED]));
    
                //Check if tx has already been executed
                if (executionState[_srcChainId][nonce] != STATUS_READY) {
                    revert AlreadyExecutedTransaction();
                }
    
                // Get User Virtual Account
                VirtualAccount userAccount = IPort(localPortAddress).fetchVirtualAccount(
                    address(uint160(bytes20(_payload[PARAMS_START:PARAMS_START_SIGNED])))
                );
    
                // Toggle Router Virtual Account use for tx execution
                IPort(localPortAddress).toggleVirtualAccountApproved(userAccount, localRouterAddress);
    
                // Avoid stack too deep
                uint16 srcChainId = _srcChainId;
    
                // Try to execute remote request
                // Flag 5 - RootBridgeAgentExecutor(bridgeAgentExecutorAddress).executeSignedWithDeposit(address(userAccount), localRouterAddress, data, _srcChainId)
                _execute(
                    _payload[0] == 0x85,
                    nonce,
                    address(uint160(bytes20(_payload[PARAMS_START:PARAMS_START_SIGNED]))),
                    abi.encodeWithSelector(
                        RootBridgeAgentExecutor.executeSignedWithDeposit.selector,
                        address(userAccount),
                        localRouterAddress,
                        _payload,
                        srcChainId
                    ),
                    srcChainId
                );
    
                // Toggle Router Virtual Account use for tx execution
                IPort(localPortAddress).toggleVirtualAccountApproved(userAccount, localRouterAddress);
    
                // DEPOSIT FLAG: 6 (Call with multiple asset Deposit + msg.sender)
            } else if (_payload[0] & 0x7F == 0x06) {
                // Parse deposit nonce
                nonce = uint32(bytes4(_payload[PARAMS_START_SIGNED + PARAMS_START:PARAMS_START_SIGNED + PARAMS_TKN_START]));
    
                // Check if tx has already been executed
                if (executionState[_srcChainId][nonce] != STATUS_READY) {
                    revert AlreadyExecutedTransaction();
                }
    
                // Get User Virtual Account
                VirtualAccount userAccount = IPort(localPortAddress).fetchVirtualAccount(
                    address(uint160(bytes20(_payload[PARAMS_START:PARAMS_START_SIGNED])))
                );
    
                // Toggle Router Virtual Account use for tx execution
                IPort(localPortAddress).toggleVirtualAccountApproved(userAccount, localRouterAddress);
    
                // Avoid stack too deep
                uint16 srcChainId = _srcChainId;
    
                // Try to execute remote request
                // Flag 6 - RootBridgeAgentExecutor(bridgeAgentExecutorAddress).executeSignedWithDepositMultiple(address(userAccount), localRouterAddress, data, _srcChainId)
                _execute(
                    _payload[0] == 0x86,
                    nonce,
                    address(uint160(bytes20(_payload[PARAMS_START:PARAMS_START_SIGNED]))),
                    abi.encodeWithSelector(
                        RootBridgeAgentExecutor.executeSignedWithDepositMultiple.selector,
                        address(userAccount),
                        localRouterAddress,
                        _payload,
                        srcChainId
                    ),
                    srcChainId
                );
    
                // Toggle Router Virtual Account use for tx execution
                IPort(localPortAddress).toggleVirtualAccountApproved(userAccount, localRouterAddress);
    
                /// DEPOSIT FLAG: 7 (retrySettlement)
            } else if (_payload[0] & 0x7F == 0x07) {
                // Prepare Variables for decoding
                address owner;
                bytes memory params;
                GasParams memory gParams;
    
                // Decode Input
                (nonce, owner, params, gParams) = abi.decode(_payload[PARAMS_START:], (uint32, address, bytes, GasParams));
    
                // Get storage reference
                Settlement storage settlementReference = getSettlement[nonce];
    
                // Check if Settlement hasn't been redeemed.
                if (settlementReference.owner == address(0)) revert SettlementRetryUnavailable();
    
                // Check settlement owner
                if (owner != settlementReference.owner) {
                    if (owner != address(IPort(localPortAddress).getUserAccount(settlementReference.owner))) {
                        revert NotSettlementOwner();
                    }
                }
    
                //Update Settlement Staus
                settlementReference.status = STATUS_SUCCESS;
    
                //Retry settlement call with new params and gas
                _performRetrySettlementCall(
                    _payload[0] == 0x87,
                    settlementReference.hTokens,
                    settlementReference.tokens,
                    settlementReference.amounts,
                    settlementReference.deposits,
                    params,
                    nonce,
                    payable(settlementReference.owner),
                    settlementReference.recipient,
                    settlementReference.dstChainId,
                    gParams,
                    address(this).balance
                );
    
                /// DEPOSIT FLAG: 8 (retrieveDeposit)
            } else if (_payload[0] == 0x08) {
                //Parse deposit nonce
                nonce = uint32(bytes4(_payload[PARAMS_START_SIGNED:PARAMS_TKN_START_SIGNED]));
    
                //Check if deposit is in retrieve mode
                if (executionState[_srcChainId][nonce] == STATUS_DONE) {
                    revert AlreadyExecutedTransaction();
                } else {
                    //Set settlement to retrieve mode, if not already set.
                    if (executionState[_srcChainId][nonce] == STATUS_READY) {
                        executionState[_srcChainId][nonce] = STATUS_RETRIEVE;
                    }
                    //Trigger fallback/Retry failed fallback
                    _performFallbackCall(
                        payable(address(uint160(bytes20(_payload[PARAMS_START:PARAMS_START_SIGNED])))), nonce, _srcChainId
                    );
                }
    
                //DEPOSIT FLAG: 9 (Fallback)
            } else if (_payload[0] == 0x09) {
                // Parse nonce
                nonce = uint32(bytes4(_payload[PARAMS_START:PARAMS_TKN_START]));
    
                // Reopen Settlement for redemption
                getSettlement[nonce].status = STATUS_FAILED;
    
                // Emit LogFallback
                emit LogFallback(nonce, _srcChainId);
    
                // return to prevent unnecessary emits/logic
                return;
    
                // Unrecognized Function Selector
            } else {
                revert UnknownFlag();
            }
    
            emit LogExecute(nonce, _srcChainId);
        }
    
        /*///////////////////////////////////////////////////////////////
                    DEPOSIT EXECUTION INTERNAL FUNCTIONS
        //////////////////////////////////////////////////////////////*/
    
        /**
         * @notice Internal function requests execution from Root Bridge Agent Executor Contract.
         *   @param _depositNonce Identifier for nonce being executed.
         *   @param _calldata Payload of message to be executed by the Root Bridge Agent Executor Contract.
         *   @param _srcChainId Chain ID of source chain where request originates from.
         */
        function _execute(uint256 _depositNonce, bytes memory _calldata, uint16 _srcChainId) private {
            //Update tx state as executed
            executionState[_srcChainId][_depositNonce] = STATUS_DONE;
    
            //Try to execute the remote request
            (bool success,) = bridgeAgentExecutorAddress.call{value: address(this).balance}(_calldata);
    
            // No fallback is requested revert allowing for retry.
            if (!success) revert ExecutionFailure();
        }
    
        /**
         * @notice Internal function requests execution from Root Bridge Agent Executor Contract.
         *   @param _hasFallbackToggled if true, fallback on execution failure is toggled on.
         *   @param _depositNonce Identifier for nonce being executed.
         *   @param _refundee address to refund gas to in case of fallback being triggered.
         *   @param _calldata Calldata to be executed by the Root Bridge Agent Executor Contract.
         *   @param _srcChainId Chain ID of source chain where request originates from.
         */
        function _execute(
            bool _hasFallbackToggled,
            uint32 _depositNonce,
            address _refundee,
            bytes memory _calldata,
            uint16 _srcChainId
        ) private {
            //Update tx state as executed
            executionState[_srcChainId][_depositNonce] = STATUS_DONE;
    
            //Try to execute the remote request
            (bool success,) = bridgeAgentExecutorAddress.call{value: address(this).balance}(_calldata);
    
            //Update tx state if execution failed
            if (!success) {
                //Read the fallback flag.
                if (_hasFallbackToggled) {
                    // Update tx state as retrieve only
                    executionState[_srcChainId][_depositNonce] = STATUS_RETRIEVE;
                    // Perform the fallback call
                    _performFallbackCall(payable(_refundee), _depositNonce, _srcChainId);
                } else {
                    // No fallback is requested revert allowing for retry.
                    revert ExecutionFailure();
                }
            }
        }
    
        /*///////////////////////////////////////////////////////////////
                        LAYER ZERO INTERNAL FUNCTIONS
        //////////////////////////////////////////////////////////////*/
    
        /**
         * @notice Internal function performs call to Layer Zero Endpoint Contract for cross-chain messaging.
         *   @param _refundee address to refund excess gas to.
         *   @param _dstChainId Layer Zero Chain ID of destination chain.
         *   @param _payload Payload of message to be sent to Layer Zero Endpoint Contract.
         *   @param _gParams Gas parameters for cross-chain message execution.
         */
    
        function _performCall(
            uint16 _dstChainId,
            address payable _refundee,
            bytes memory _payload,
            GasParams calldata _gParams
        ) internal {
            // Get destination Branch Bridge Agent
            address callee = getBranchBridgeAgent[_dstChainId];
    
            // Check if valid destination
            if (callee == address(0)) revert UnrecognizedBridgeAgent();
    
            // Check if call to remote chain
            if (_dstChainId != localChainId) {
                //Sends message to Layerzero Enpoint
                ILayerZeroEndpoint(lzEndpointAddress).send{value: msg.value}(
                    _dstChainId,
                    getBranchBridgeAgentPath[_dstChainId],
                    _payload,
                    _refundee,
                    address(0),
                    abi.encodePacked(uint16(2), _gParams.gasLimit, _gParams.remoteBranchExecutionGas, callee)
                );
    
                // Check if call to local chain
            } else {
                //Send Gas to Local Branch Bridge Agent
                callee.call{value: msg.value}("");
                //Execute locally
                IBranchBridgeAgent(callee).lzReceive(0, "", 0, _payload);
            }
        }
    
        /**
         * @notice Internal function performs call to Layer Zero Endpoint Contract for cross-chain messaging.
         *   @param _hasFallbackToggled if true, fallback is toggled on.
         *   @param _hTokens deposited global token address.
         *   @param _tokens deposited global token address.
         *   @param _amounts amounts of total hTokens + Tokens output.
         *   @param _deposits amount of underlying/native tokens to output.
         *   @param _params Payload of message to be sent to Layer Zero Endpoint Contract.
         *   @param _gParams Gas parameters for cross-chain message execution.
         *   @param _settlementNonce Identifier for token settlement.
         *   @param _refundee address of token owner and gas refundee.
         *   @param _recipient destination chain receiver address.
         *   @param _dstChainId Chain ID of destination chain.
         */
    
        function _performRetrySettlementCall(
            bool _hasFallbackToggled,
            address[] memory _hTokens,
            address[] memory _tokens,
            uint256[] memory _amounts,
            uint256[] memory _deposits,
            bytes memory _params,
            uint32 _settlementNonce,
            address payable _refundee,
            address _recipient,
            uint16 _dstChainId,
            GasParams memory _gParams,
            uint256 _value
        ) internal {
            // Check if payload is ready for message
            if (_hTokens.length == 0) revert SettlementRetryUnavailableUseCallout();
    
            // Get packed data
            bytes memory payload;
    
            // Check if it's a single asset settlement
            if (_hTokens.length == 1) {
                //Pack new Data
                payload = abi.encodePacked(
                    _hasFallbackToggled ? bytes1(0x81) : bytes1(0x01),
                    _recipient,
                    _settlementNonce,
                    _hTokens[0],
                    _tokens[0],
                    _amounts[0],
                    _deposits[0],
                    _params
                );
    
                // Check if it's mulitple asset settlement
            } else if (_hTokens.length > 1) {
                //Pack new Data
                payload = abi.encodePacked(
                    _hasFallbackToggled ? bytes1(0x82) : bytes1(0x02),
                    _recipient,
                    uint8(_hTokens.length),
                    _settlementNonce,
                    _hTokens,
                    _tokens,
                    _amounts,
                    _deposits,
                    _params
                );
            }
    
            // Get destination Branch Bridge Agent
            address callee = getBranchBridgeAgent[_dstChainId];
    
            // Check if valid destination
            if (callee == address(0)) revert UnrecognizedBridgeAgent();
    
            // Check if call to remote chain
            if (_dstChainId != localChainId) {
                //Sends message to Layerzero Enpoint
                ILayerZeroEndpoint(lzEndpointAddress).send{value: _value}(
                    _dstChainId,
                    getBranchBridgeAgentPath[_dstChainId],
                    payload,
                    payable(_refundee),
                    address(0),
                    abi.encodePacked(uint16(2), _gParams.gasLimit, _gParams.remoteBranchExecutionGas, callee)
                );
    
                // Check if call to local chain
            } else {
                //Send Gas to Local Branch Bridge Agent
                callee.call{value: _value}("");
                //Execute locally
                IBranchBridgeAgent(callee).lzReceive(0, "", 0, payload);
            }
        }
    
        /**
         * @notice Internal function performs call to Layerzero Enpoint Contract for cross-chain messaging.
         *   @param _depositNonce branch deposit nonce.
         *   @param _dstChainId Chain ID of destination chain.
         */
        function _performFallbackCall(address payable _refundee, uint32 _depositNonce, uint16 _dstChainId) internal {
            //Sends message to LayerZero messaging layer
            ILayerZeroEndpoint(lzEndpointAddress).send{value: address(this).balance}(
                _dstChainId,
                getBranchBridgeAgentPath[_dstChainId],
                abi.encodePacked(bytes1(0x04), _depositNonce),
                payable(_refundee),
                address(0),
                ""
            );
        }
    
        /*///////////////////////////////////////////////////////////////
                        SETTLEMENT INTERNAL FUNCTIONS
        //////////////////////////////////////////////////////////////*/
    
        /**
         * @notice Function to settle a single asset and perform a remote call to a branch chain.
         *     @param _settlementNonce Identifier for token settlement.
         *     @param _refundee address of token owner and gas refundee.
         *     @param _recipient destination chain token receiver address.
         *     @param _dstChainId branch chain to bridge to.
         *     @param _params params for branch bridge agent and router execution.
         *     @param _globalAddress global address of the token in root chain.
         *     @param _amount amount of hTokens to be bridged out.
         *     @param _deposit amount of underlying tokens to be cleared from branch port.
         *     @param _hasFallbackToggled if true, fallback is toggled on.
         */
        function _createSettlement(
            uint32 _settlementNonce,
            address payable _refundee,
            address _recipient,
            uint16 _dstChainId,
            bytes memory _params,
            address _globalAddress,
            uint256 _amount,
            uint256 _deposit,
            bool _hasFallbackToggled
        ) internal returns (bytes memory _payload) {
            // Update Settlement Nonce
            settlementNonce = _settlementNonce + 1;
    
            // Get Local Branch Token Address from Root Port
            address localAddress = IPort(localPortAddress).getLocalTokenFromGlobal(_globalAddress, _dstChainId);
    
            // Get Underlying Token Address from Root Port
            address underlyingAddress = IPort(localPortAddress).getUnderlyingTokenFromLocal(localAddress, _dstChainId);
    
            //Update State to reflect bridgeOut
            _updateStateOnBridgeOut(
                msg.sender, _globalAddress, localAddress, underlyingAddress, _amount, _deposit, _dstChainId
            );
    
            // Prepare data for call
            _payload = abi.encodePacked(
                _hasFallbackToggled ? bytes1(0x81) : bytes1(0x01),
                _recipient,
                _settlementNonce,
                localAddress,
                underlyingAddress,
                _amount,
                _deposit,
                _params
            );
    
            // Avoid stack too deep
            uint32 cachedSettlementNonce = _settlementNonce;
    
            // Get Auxiliary Dynamic Arrays
            address[] memory addressArray = new address[](1);
            uint256[] memory uintArray = new uint256[](1);
    
            // Get storage reference for new Settlement
            Settlement storage settlement = getSettlement[cachedSettlementNonce];
    
            // Update Setttlement
            settlement.owner = _refundee;
            settlement.recipient = _recipient;
    
            addressArray[0] = localAddress;
            settlement.hTokens = addressArray;
    
            addressArray[0] = underlyingAddress;
            settlement.tokens = addressArray;
    
            uintArray[0] = _amount;
            settlement.amounts = uintArray;
    
            uintArray[0] = _deposit;
            settlement.deposits = uintArray;
    
            settlement.dstChainId = _dstChainId;
            settlement.status = STATUS_SUCCESS;
        }
    
        /**
         * @notice Function to settle multiple assets and perform a remote call to a branch chain.
         *     @param _settlementNonce Identifier for token settlement.
         *     @param _refundee address of token owner and gas refundee.
         *     @param _recipient destination chain token receiver address.
         *     @param _dstChainId branch chain to bridge to.
         *     @param _globalAddresses addresses of the global tokens in root chain.
         *     @param _amounts amounts of hTokens to be bridged out.
         *     @param _deposits amounts of underlying tokens to be cleared from branch port.
         *     @param _params params for branch bridge agent and router execution.
         *     @param _hasFallbackToggled if true, fallback is toggled on.
         */
        function _createSettlementMultiple(
            uint32 _settlementNonce,
            address payable _refundee,
            address _recipient,
            uint16 _dstChainId,
            address[] memory _globalAddresses,
            uint256[] memory _amounts,
            uint256[] memory _deposits,
            bytes memory _params,
            bool _hasFallbackToggled
        ) internal returns (bytes memory _payload) {
            // Check if valid length
            if (_globalAddresses.length > MAX_TOKENS_LENGTH) revert InvalidInputParamsLength();
    
            // Check if valid length
            if (_globalAddresses.length != _amounts.length) revert InvalidInputParamsLength();
            if (_amounts.length != _deposits.length) revert InvalidInputParamsLength();
    
            //Update Settlement Nonce
            settlementNonce = _settlementNonce + 1;
    
            // Create Arrays
            address[] memory hTokens = new address[](_globalAddresses.length);
            address[] memory tokens = new address[](_globalAddresses.length);
    
            for (uint256 i = 0; i < hTokens.length;) {
                // Populate Addresses for Settlement
                hTokens[i] = IPort(localPortAddress).getLocalTokenFromGlobal(_globalAddresses[i], _dstChainId);
                tokens[i] = IPort(localPortAddress).getUnderlyingTokenFromLocal(hTokens[i], _dstChainId);
    
                // Avoid stack too deep
                uint16 destChainId = _dstChainId;
    
                // Update State to reflect bridgeOut
                _updateStateOnBridgeOut(
                    msg.sender, _globalAddresses[i], hTokens[i], tokens[i], _amounts[i], _deposits[i], destChainId
                );
    
                unchecked {
                    ++i;
                }
            }
    
            // Prepare data for call with settlement of multiple assets
            _payload = abi.encodePacked(
                _hasFallbackToggled ? bytes1(0x02) & 0x0F : bytes1(0x02),
                _recipient,
                uint8(hTokens.length),
                _settlementNonce,
                hTokens,
                tokens,
                _amounts,
                _deposits,
                _params
            );
    
            // Create and Save Settlement
            // Get storage reference
            Settlement storage settlement = getSettlement[_settlementNonce];
    
            // Update Setttlement
            settlement.owner = _refundee;
            settlement.recipient = _recipient;
            settlement.hTokens = hTokens;
            settlement.tokens = tokens;
            settlement.amounts = _amounts;
            settlement.deposits = _deposits;
            settlement.dstChainId = _dstChainId;
            settlement.status = STATUS_SUCCESS;
        }
    
        /*///////////////////////////////////////////////////////////////
                        TOKEN MANAGEMENT INTERNAL FUNCTIONS
        //////////////////////////////////////////////////////////////*/
    
        /**
         * @notice Updates the token balance state by moving assets from root omnichain environment to branch chain,
         *         when a user wants to bridge out tokens from the root bridge agent chain.
         *     @param _depositor address of the token depositor.
         *     @param _globalAddress address of the global token.
         *     @param _localAddress address of the local token.
         *     @param _underlyingAddress address of the underlying token.
         *     @param _amount amount of hTokens to be bridged out.
         *     @param _deposit amount of underlying tokens to be bridged out.
         *     @param _dstChainId chain to bridge to.
         */
        function _updateStateOnBridgeOut(
            address _depositor,
            address _globalAddress,
            address _localAddress,
            address _underlyingAddress,
            uint256 _amount,
            uint256 _deposit,
            uint16 _dstChainId
        ) internal {
            // Check if valid inputs
            if (_amount == 0 && _deposit == 0) revert InvalidInputParams();
            if (_amount < _deposit) revert InvalidInputParams();
    
            // Check if valid assets
            if (_localAddress == address(0)) revert UnrecognizedLocalAddress();
            if (_underlyingAddress == address(0)) if (_deposit > 0) revert UnrecognizedUnderlyingAddress();
    
            // Move output hTokens from Root to Branch
            if (_amount - _deposit > 0) {
                unchecked {
                    _globalAddress.safeTransferFrom(_depositor, localPortAddress, _amount - _deposit);
                }
            }
    
            // Clear Underlying Tokens from the destination Branch
            if (_deposit > 0) {
                // Verify there is enough balance to clear native tokens if needed
                if (IERC20hTokenRoot(_globalAddress).getTokenBalance(_dstChainId) < _deposit) {
                    revert InsufficientBalanceForSettlement();
                }
                IPort(localPortAddress).burn(_depositor, _globalAddress, _deposit, _dstChainId);
            }
        }
    
        /*///////////////////////////////////////////////////////////////
                                ADMIN FUNCTIONS
        //////////////////////////////////////////////////////////////*/
    
        /// @inheritdoc IRootBridgeAgent
        function approveBranchBridgeAgent(uint256 _branchChainId) external override requiresManager {
            if (getBranchBridgeAgent[_branchChainId] != address(0)) revert AlreadyAddedBridgeAgent();
            isBranchBridgeAgentAllowed[_branchChainId] = true;
        }
    
        /// @inheritdoc IRootBridgeAgent
        function syncBranchBridgeAgent(address _newBranchBridgeAgent, uint256 _branchChainId)
            external
            override
            requiresPort
        {
            getBranchBridgeAgent[_branchChainId] = _newBranchBridgeAgent;
            getBranchBridgeAgentPath[_branchChainId] = abi.encodePacked(_newBranchBridgeAgent, address(this));
        }
    
        /*///////////////////////////////////////////////////////////////
                                MODIFIERS
        //////////////////////////////////////////////////////////////*/
    
        /// @notice Modifier for a simple re-entrancy check.
        modifier lock() {
            require(_unlocked == 1);
            _unlocked = 2;
            _;
            _unlocked = 1;
        }
    
        /// @notice Internal function to verify msg sender is Bridge Agent's Router.
        modifier requiresRouter() {
            if (msg.sender != localRouterAddress) revert UnrecognizedRouter();
            _;
        }
    
        /// @notice Modifier verifies the caller is the Layerzero Enpoint or Local Branch Bridge Agent.
        modifier requiresEndpoint(address _endpoint, uint16 _srcChain, bytes calldata _srcAddress) virtual {
            if (msg.sender != address(this)) revert LayerZeroUnauthorizedEndpoint();
    
            if (_endpoint != getBranchBridgeAgent[localChainId]) {
                if (_endpoint != lzEndpointAddress) revert LayerZeroUnauthorizedEndpoint();
    
                if (_srcAddress.length != 40) revert LayerZeroUnauthorizedCaller();
    
                if (getBranchBridgeAgent[_srcChain] != address(uint160(bytes20(_srcAddress[PARAMS_ADDRESS_SIZE:])))) {
                    revert LayerZeroUnauthorizedCaller();
                }
            }
            _;
        }
    
        /// @notice Modifier that verifies msg sender is Bridge Agent Executor.
        modifier requiresAgentExecutor() {
            if (msg.sender != bridgeAgentExecutorAddress) revert UnrecognizedExecutor();
            _;
        }
    
        /// @notice Modifier that verifies msg sender is the Local Port.
        modifier requiresPort() {
            if (msg.sender != localPortAddress) revert UnrecognizedPort();
            _;
        }
    
        /// @notice Modifier that verifies msg sender is the Bridge Agent's Manager.
        modifier requiresManager() {
            if (msg.sender != IPort(localPortAddress).getBridgeAgentManager(address(this))) {
                revert UnrecognizedBridgeAgentManager();
            }
            _;
        }
    }

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L32-L1238)

```solidity
File: src/RootPort.sol

// @audit from 16 to 15 you need to change the structure elements order to: , uint256, mapping, mapping, mapping, mapping, mapping, mapping, mapping, mapping, mapping, mapping, mapping, address, address, address, bool, bool, array, array
16: contract RootPort is Ownable, IRootPort {
        using SafeTransferLib for address;
    
        /*///////////////////////////////////////////////////////////////
                                SETUP STATE
        //////////////////////////////////////////////////////////////*/
    
        /// @notice True if setup is still ongoing, false otherwise.
        bool internal _setup;
    
        /// @notice True if core setup is still ongoing, false otherwise.
        bool internal _setupCore;
    
        /*///////////////////////////////////////////////////////////////
                            ROOT PORT STATE
        //////////////////////////////////////////////////////////////*/
    
        /// @notice Local Chain Id
        uint256 public immutable localChainId;
    
        /// @notice The address of local branch port responsible for handling local transactions.
        address public localBranchPortAddress;
    
        /// @notice The address of the core router in charge of adding new tokens to the system.
        address public coreRootRouterAddress;
    
        /// @notice The address of the core router in charge of adding new tokens to the system.
        address public coreRootBridgeAgentAddress;
    
        /*///////////////////////////////////////////////////////////////
                            VIRTUAL ACCOUNT
        //////////////////////////////////////////////////////////////*/
    
        /// @notice Mapping from user address to Virtual Account.
        mapping(address user => VirtualAccount account) public getUserAccount;
    
        /// @notice Holds the mapping from Virtual account to router address => bool.
        /// @notice Stores whether a router is approved to spend a virtual account.
        mapping(VirtualAccount acount => mapping(address router => bool allowed)) public isRouterApproved;
    
        /*///////////////////////////////////////////////////////////////
                            BRIDGE AGENTS
        //////////////////////////////////////////////////////////////*/
    
        /// @notice Mapping from address to Bridge Agent.
        mapping(uint256 chainId => bool isActive) public isChainId;
    
        /// @notice Mapping from address to isBridgeAgent (bool).
        mapping(address bridgeAgent => bool isActive) public isBridgeAgent;
    
        /// @notice Bridge Agents deployed in root chain.
        address[] public bridgeAgents;
    
        /// @notice Mapping address Bridge Agent => address Bridge Agent Manager
        mapping(address bridgeAgent => address bridgeAgentManager) public getBridgeAgentManager;
    
        /*///////////////////////////////////////////////////////////////
                        BRIDGE AGENT FACTORIES
        //////////////////////////////////////////////////////////////*/
    
        /// @notice Mapping from Underlying Address to isUnderlying (bool).
        mapping(address bridgeAgentFactory => bool isActive) public isBridgeAgentFactory;
    
        /// @notice Bridge Agents deployed in root chain.
        address[] public bridgeAgentFactories;
    
        /*///////////////////////////////////////////////////////////////
                                hTOKENS
        //////////////////////////////////////////////////////////////*/
    
        /// @notice Mapping with all global hTokens deployed in the system.
        mapping(address token => bool isGlobalToken) public isGlobalAddress;
    
        /// @notice ChainId -> Local Address -> Global Address
        mapping(address chainId => mapping(uint256 localAddress => address globalAddress)) public getGlobalTokenFromLocal;
    
        /// @notice ChainId -> Global Address -> Local Address
        mapping(address chainId => mapping(uint256 globalAddress => address localAddress)) public getLocalTokenFromGlobal;
    
        /// @notice ChainId -> Underlying Address -> Local Address
        mapping(address chainId => mapping(uint256 underlyingAddress => address localAddress)) public
            getLocalTokenFromUnderlying;
    
        /// @notice Mapping from Local Address to Underlying Address.
        mapping(address chainId => mapping(uint256 localAddress => address underlyingAddress)) public
            getUnderlyingTokenFromLocal;
    
        /*///////////////////////////////////////////////////////////////
                                    CONSTRUCTOR
        //////////////////////////////////////////////////////////////*/
    
        /**
         * @notice Constructor for Root Port.
         * @param _localChainId layer zero chain id of the local chain.
         */
        constructor(uint256 _localChainId) {
            localChainId = _localChainId;
            isChainId[_localChainId] = true;
    
            _initializeOwner(msg.sender);
            _setup = true;
            _setupCore = true;
        }
    
        /*///////////////////////////////////////////////////////////////
                        INITIALIZATION FUNCTIONS
        //////////////////////////////////////////////////////////////*/
    
        /**
         *  @notice Function to initialize the Root Port.
         *   @param _bridgeAgentFactory The address of the Bridge Agent Factory.
         *   @param _coreRootRouter The address of the Core Root Router.
         */
        function initialize(address _bridgeAgentFactory, address _coreRootRouter) external onlyOwner {
            require(_bridgeAgentFactory != address(0), "Bridge Agent Factory cannot be 0 address.");
            require(_coreRootRouter != address(0), "Core Root Router cannot be 0 address.");
            require(_setup, "Setup ended.");
            _setup = false;
    
            isBridgeAgentFactory[_bridgeAgentFactory] = true;
            bridgeAgentFactories.push(_bridgeAgentFactory);
    
            coreRootRouterAddress = _coreRootRouter;
        }
    
        /**
         *  @notice Function to initialize the Root Chain Core Contracts in Port Storage.
         *   @param _coreRootBridgeAgent The address of the Core Root Bridge Agent.
         *   @param _coreLocalBranchBridgeAgent The address of the Core Arbitrum Branch Bridge Agent.
         *   @param _localBranchPortAddress The address of the Arbitrum Branch Port.
         */
        function initializeCore(
            address _coreRootBridgeAgent,
            address _coreLocalBranchBridgeAgent,
            address _localBranchPortAddress
        ) external onlyOwner {
            require(_coreRootBridgeAgent != address(0), "Core Root Bridge Agent cannot be 0 address.");
            require(_coreLocalBranchBridgeAgent != address(0), "Core Local Branch Bridge Agent cannot be 0 address.");
            require(_localBranchPortAddress != address(0), "Local Branch Port Address cannot be 0 address.");
            require(isBridgeAgent[_coreRootBridgeAgent], "Core Bridge Agent doesn't exist.");
            require(_setupCore, "Core Setup ended.");
            _setupCore = false;
    
            coreRootBridgeAgentAddress = _coreRootBridgeAgent;
            localBranchPortAddress = _localBranchPortAddress;
            IBridgeAgent(_coreRootBridgeAgent).syncBranchBridgeAgent(_coreLocalBranchBridgeAgent, localChainId);
            getBridgeAgentManager[_coreRootBridgeAgent] = owner();
        }
    
        /// @notice Function being overriden to prevent mistakenly renouncing ownership.
        function renounceOwnership() public payable override onlyOwner {
            revert("Cannot renounce ownership");
        }
    
        /*///////////////////////////////////////////////////////////////
                            EXTERNAL VIEW FUNCTIONS
        //////////////////////////////////////////////////////////////*/
    
        /// @inheritdoc IRootPort
        function getLocalToken(address _localAddress, uint256 _srcChainId, uint256 _dstChainId)
            external
            view
            override
            returns (address)
        {
            return _getLocalToken(_localAddress, _srcChainId, _dstChainId);
        }
    
        /**
         * @notice View Function returns Local Token's Local Address on another chain.
         * @param _localAddress The address of the token in the local chain.
         * @param _srcChainId The chainId of the chain where the token is deployed.
         * @param _dstChainId The chainId of the chain for which the token address is requested.
         */
        function _getLocalToken(address _localAddress, uint256 _srcChainId, uint256 _dstChainId)
            internal
            view
            returns (address)
        {
            address globalAddress = getGlobalTokenFromLocal[_localAddress][_srcChainId];
            return getLocalTokenFromGlobal[globalAddress][_dstChainId];
        }
    
        /// @inheritdoc IRootPort
        function getUnderlyingTokenFromGlobal(address _globalAddress, uint256 _srcChainId)
            external
            view
            override
            returns (address)
        {
            address localAddress = getLocalTokenFromGlobal[_globalAddress][_srcChainId];
            return getUnderlyingTokenFromLocal[localAddress][_srcChainId];
        }
    
        /// @inheritdoc IRootPort
        function isGlobalToken(address _globalAddress, uint256 _srcChainId) external view override returns (bool) {
            return getLocalTokenFromGlobal[_globalAddress][_srcChainId] != address(0);
        }
    
        /// @inheritdoc IRootPort
        function isLocalToken(address _localAddress, uint256 _srcChainId) external view override returns (bool) {
            return getGlobalTokenFromLocal[_localAddress][_srcChainId] != address(0);
        }
    
        /// @inheritdoc IRootPort
        function isLocalToken(address _localAddress, uint256 _srcChainId, uint256 _dstChainId)
            external
            view
            returns (bool)
        {
            return _getLocalToken(_localAddress, _srcChainId, _dstChainId) != address(0);
        }
    
        /// @inheritdoc IRootPort
        function isUnderlyingToken(address _underlyingToken, uint256 _srcChainId) external view override returns (bool) {
            return getLocalTokenFromUnderlying[_underlyingToken][_srcChainId] != address(0);
        }
    
        /*///////////////////////////////////////////////////////////////
                            hTOKEN MANAGEMENT FUNCTIONS
        //////////////////////////////////////////////////////////////*/
    
        /// @inheritdoc IRootPort
        function setAddresses(
            address _globalAddress,
            address _localAddress,
            address _underlyingAddress,
            uint256 _srcChainId
        ) external override requiresCoreRootRouter {
            if (_globalAddress == address(0)) revert InvalidGlobalAddress();
            if (_localAddress == address(0)) revert InvalidLocalAddress();
            if (_underlyingAddress == address(0)) revert InvalidUnderlyingAddress();
    
            isGlobalAddress[_globalAddress] = true;
            getGlobalTokenFromLocal[_localAddress][_srcChainId] = _globalAddress;
            getLocalTokenFromGlobal[_globalAddress][_srcChainId] = _localAddress;
            getLocalTokenFromUnderlying[_underlyingAddress][_srcChainId] = _localAddress;
            getUnderlyingTokenFromLocal[_localAddress][_srcChainId] = _underlyingAddress;
    
            emit LocalTokenAdded(_underlyingAddress, _localAddress, _globalAddress, _srcChainId);
        }
    
        /// @inheritdoc IRootPort
        function setLocalAddress(address _globalAddress, address _localAddress, uint256 _srcChainId)
            external
            override
            requiresCoreRootRouter
        {
            if (_localAddress == address(0)) revert InvalidLocalAddress();
    
            getGlobalTokenFromLocal[_localAddress][_srcChainId] = _globalAddress;
            getLocalTokenFromGlobal[_globalAddress][_srcChainId] = _localAddress;
    
            emit GlobalTokenAdded(_localAddress, _globalAddress, _srcChainId);
        }
    
        /*///////////////////////////////////////////////////////////////
                            hTOKEN ACCOUNTING FUNCTIONS
        //////////////////////////////////////////////////////////////*/
    
        /// @inheritdoc IRootPort
        function bridgeToRoot(address _recipient, address _hToken, uint256 _amount, uint256 _deposit, uint256 _srcChainId)
            external
            override
            requiresBridgeAgent
        {
            if (!isGlobalAddress[_hToken]) revert UnrecognizedToken();
    
            if (_amount - _deposit > 0) {
                unchecked {
                    _hToken.safeTransfer(_recipient, _amount - _deposit);
                }
            }
    
            if (_deposit > 0) if (!ERC20hTokenRoot(_hToken).mint(_recipient, _deposit, _srcChainId)) revert UnableToMint();
        }
    
        /// @inheritdoc IRootPort
        function bridgeToRootFromLocalBranch(address _from, address _hToken, uint256 _amount)
            external
            override
            requiresLocalBranchPort
        {
            if (!isGlobalAddress[_hToken]) revert UnrecognizedToken();
    
            _hToken.safeTransferFrom(_from, address(this), _amount);
        }
    
        function bridgeToLocalBranchFromRoot(address _to, address _hToken, uint256 _amount)
            external
            override
            requiresLocalBranchPort
        {
            if (!isGlobalAddress[_hToken]) revert UnrecognizedToken();
    
            _hToken.safeTransfer(_to, _amount);
        }
    
        /// @inheritdoc IRootPort
        function burn(address _from, address _hToken, uint256 _amount, uint256 _srcChainId)
            external
            override
            requiresBridgeAgent
        {
            if (!isGlobalAddress[_hToken]) revert UnrecognizedToken();
            ERC20hTokenRoot(_hToken).burn(_from, _amount, _srcChainId);
        }
    
        /// @inheritdoc IRootPort
        function burnFromLocalBranch(address _from, address _hToken, uint256 _amount)
            external
            override
            requiresLocalBranchPort
        {
            if (!isGlobalAddress[_hToken]) revert UnrecognizedToken();
    
            ERC20hTokenRoot(_hToken).burn(_from, _amount, localChainId);
        }
    
        /// @inheritdoc IRootPort
        function mintToLocalBranch(address _to, address _hToken, uint256 _amount)
            external
            override
            requiresLocalBranchPort
        {
            if (!isGlobalAddress[_hToken]) revert UnrecognizedToken();
            if (!ERC20hTokenRoot(_hToken).mint(_to, _amount, localChainId)) revert UnableToMint();
        }
    
        /*///////////////////////////////////////////////////////////////
                        VIRTUAL ACCOUNT MANAGEMENT FUNCTIONS
        //////////////////////////////////////////////////////////////*/
    
        /// @inheritdoc IRootPort
        function fetchVirtualAccount(address _user) external override returns (VirtualAccount account) {
            account = getUserAccount[_user];
            if (address(account) == address(0)) account = addVirtualAccount(_user);
        }
    
        /**
         * @notice Creates a new virtual account for a user.
         * @param _user address of the user to associate a virtual account with.
         */
        function addVirtualAccount(address _user) internal returns (VirtualAccount newAccount) {
            if (_user == address(0)) revert InvalidUserAddress();
    
            newAccount = new VirtualAccount{salt: keccak256(abi.encode(_user))}(_user, address(this));
            getUserAccount[_user] = newAccount;
    
            emit VirtualAccountCreated(_user, address(newAccount));
        }
    
        /// @inheritdoc IRootPort
        function toggleVirtualAccountApproved(VirtualAccount _userAccount, address _router)
            external
            override
            requiresBridgeAgent
        {
            isRouterApproved[_userAccount][_router] = !isRouterApproved[_userAccount][_router];
        }
    
        /*///////////////////////////////////////////////////////////////
                            BRIDGE AGENT ADDITION FUNCTIONS
        //////////////////////////////////////////////////////////////*/
    
        /// @inheritdoc IRootPort
        function addBridgeAgent(address _manager, address _bridgeAgent) external override requiresBridgeAgentFactory {
            if (isBridgeAgent[_bridgeAgent]) revert AlreadyAddedBridgeAgent();
    
            bridgeAgents.push(_bridgeAgent);
            getBridgeAgentManager[_bridgeAgent] = _manager;
            isBridgeAgent[_bridgeAgent] = true;
    
            emit BridgeAgentAdded(_bridgeAgent, _manager);
        }
    
        /// @inheritdoc IRootPort
        function syncBranchBridgeAgentWithRoot(
            address _newBranchBridgeAgent,
            address _rootBridgeAgent,
            uint256 _branchChainId
        ) external override requiresCoreRootRouter {
            if (IBridgeAgent(_rootBridgeAgent).getBranchBridgeAgent(_branchChainId) != address(0)) {
                revert AlreadyAddedBridgeAgent();
            }
            if (!IBridgeAgent(_rootBridgeAgent).isBranchBridgeAgentAllowed(_branchChainId)) {
                revert BridgeAgentNotAllowed();
            }
            IBridgeAgent(_rootBridgeAgent).syncBranchBridgeAgent(_newBranchBridgeAgent, _branchChainId);
    
            emit BridgeAgentSynced(_newBranchBridgeAgent, _rootBridgeAgent, _branchChainId);
        }
    
        /*///////////////////////////////////////////////////////////////
                                ADMIN FUNCTIONS
        //////////////////////////////////////////////////////////////*/
    
        /// @inheritdoc IRootPort
        function toggleBridgeAgent(address _bridgeAgent) external override onlyOwner {
            isBridgeAgent[_bridgeAgent] = !isBridgeAgent[_bridgeAgent];
    
            emit BridgeAgentToggled(_bridgeAgent);
        }
    
        /// @inheritdoc IRootPort
        function addBridgeAgentFactory(address _bridgeAgentFactory) external override onlyOwner {
            if (isBridgeAgentFactory[_bridgeAgentFactory]) revert AlreadyAddedBridgeAgentFactory();
    
            bridgeAgentFactories.push(_bridgeAgentFactory);
            isBridgeAgentFactory[_bridgeAgentFactory] = true;
    
            emit BridgeAgentFactoryAdded(_bridgeAgentFactory);
        }
    
        /// @inheritdoc IRootPort
        function toggleBridgeAgentFactory(address _bridgeAgentFactory) external override onlyOwner {
            isBridgeAgentFactory[_bridgeAgentFactory] = !isBridgeAgentFactory[_bridgeAgentFactory];
    
            emit BridgeAgentFactoryToggled(_bridgeAgentFactory);
        }
    
        /// @inheritdoc IRootPort
        function addNewChain(
            address _coreBranchBridgeAgentAddress,
            uint256 _chainId,
            string memory _wrappedGasTokenName,
            string memory _wrappedGasTokenSymbol,
            uint8 _wrappedGasTokenDecimals,
            address _newLocalBranchWrappedNativeTokenAddress,
            address _newUnderlyingBranchWrappedNativeTokenAddress
        ) external override onlyOwner {
            // Check if chain already added
            if (isChainId[_chainId]) revert AlreadyAddedChain();
    
            // Create new global token for new chain's wrapped native token
            address newGlobalToken = address(
                IERC20hTokenRootFactory(ICoreRootRouter(coreRootRouterAddress).hTokenFactoryAddress()).createToken(
                    _wrappedGasTokenName, _wrappedGasTokenSymbol, _wrappedGasTokenDecimals
                )
            );
    
            // Sync new branch bridge agent with root core bridge agent
            IBridgeAgent(ICoreRootRouter(coreRootRouterAddress).bridgeAgentAddress()).syncBranchBridgeAgent(
                _coreBranchBridgeAgentAddress, _chainId
            );
    
            // Update State
    
            // 1. Add new chain to chainId mapping
            isChainId[_chainId] = true;
            // 2. Add new chain to global address mapping
            isGlobalAddress[newGlobalToken] = true;
            // 3. Add new branch local token to global token address mapping
            getGlobalTokenFromLocal[_newLocalBranchWrappedNativeTokenAddress][_chainId] = newGlobalToken;
            // 4. Add new global token to branch local token address mapping
            getLocalTokenFromGlobal[newGlobalToken][_chainId] = _newLocalBranchWrappedNativeTokenAddress;
            // 5. Add new branch underlying token to branch local token address mapping
            getLocalTokenFromUnderlying[_newUnderlyingBranchWrappedNativeTokenAddress][_chainId] =
                _newLocalBranchWrappedNativeTokenAddress;
            // 6. Add new branch local token to branch underlying token address mapping
            getUnderlyingTokenFromLocal[_newLocalBranchWrappedNativeTokenAddress][_chainId] =
                _newUnderlyingBranchWrappedNativeTokenAddress;
    
            emit NewChainAdded(_chainId);
        }
    
        /// @inheritdoc IRootPort
        function addEcosystemToken(address _ecoTokenGlobalAddress) external override onlyOwner {
            // Check if token already added
            if (isGlobalAddress[_ecoTokenGlobalAddress]) revert AlreadyAddedEcosystemToken();
    
            // Check if token is already a underlying token in current chain
            if (getUnderlyingTokenFromLocal[_ecoTokenGlobalAddress][localChainId] != address(0)) {
                revert AlreadyAddedEcosystemToken();
            }
    
            // Check if token is already a local branch token in current chain
            if (getLocalTokenFromUnderlying[_ecoTokenGlobalAddress][localChainId] != address(0)) {
                revert AlreadyAddedEcosystemToken();
            }
    
            // Update State
            // 1. Add new global token to global address mapping
            isGlobalAddress[_ecoTokenGlobalAddress] = true;
            // 2. Add new branch local token address to global token mapping
            getGlobalTokenFromLocal[_ecoTokenGlobalAddress][localChainId] = _ecoTokenGlobalAddress;
            // 3. Add new global token to branch local token address mapping
            getLocalTokenFromGlobal[_ecoTokenGlobalAddress][localChainId] = _ecoTokenGlobalAddress;
    
            emit EcosystemTokenAdded(_ecoTokenGlobalAddress);
        }
    
        /// @inheritdoc IRootPort
        function setCoreRootRouter(address _coreRootRouter, address _coreRootBridgeAgent) external override onlyOwner {
            if (_coreRootRouter == address(0)) revert InvalidCoreRootRouter();
            if (_coreRootBridgeAgent == address(0)) revert InvalidCoreRootBridgeAgent();
    
            coreRootRouterAddress = _coreRootRouter;
            coreRootBridgeAgentAddress = _coreRootBridgeAgent;
            getBridgeAgentManager[_coreRootBridgeAgent] = owner();
    
            emit CoreRootSet(_coreRootRouter, _coreRootBridgeAgent);
        }
    
        /// @inheritdoc IRootPort
        function setCoreBranchRouter(
            address _refundee,
            address _coreBranchRouter,
            address _coreBranchBridgeAgent,
            uint16 _dstChainId,
            GasParams calldata _gParams
        ) external payable override onlyOwner {
            if (_coreBranchRouter == address(0)) revert InvalidCoreBranchRouter();
            if (_coreBranchBridgeAgent == address(0)) revert InvalidCoreBrancBridgeAgent();
    
            ICoreRootRouter(coreRootRouterAddress).setCoreBranch{value: msg.value}(
                _refundee, _coreBranchRouter, _coreBranchBridgeAgent, _dstChainId, _gParams
            );
    
            emit CoreBranchSet(_coreBranchRouter, _coreBranchBridgeAgent, _dstChainId);
        }
    
        /// @inheritdoc IRootPort
        function syncNewCoreBranchRouter(address _coreBranchRouter, address _coreBranchBridgeAgent, uint16 _dstChainId)
            external
            override
            onlyOwner
        {
            if (_coreBranchRouter == address(0)) revert InvalidCoreBranchRouter();
            if (_coreBranchBridgeAgent == address(0)) revert InvalidCoreBrancBridgeAgent();
    
            IBridgeAgent(coreRootBridgeAgentAddress).syncBranchBridgeAgent(_coreBranchBridgeAgent, _dstChainId);
    
            emit CoreBranchSynced(_coreBranchRouter, _coreBranchBridgeAgent, _dstChainId);
        }
    
        /*///////////////////////////////////////////////////////////////
                                    MODIFIERS
        //////////////////////////////////////////////////////////////*/
    
        /// @notice Modifier that verifies msg sender is an active Bridge Agent Factory.
        modifier requiresBridgeAgentFactory() {
            if (!isBridgeAgentFactory[msg.sender]) revert UnrecognizedBridgeAgentFactory();
            _;
        }
    
        /// @notice Modifier that verifies msg sender is an active Bridge Agent.
        modifier requiresBridgeAgent() {
            if (!isBridgeAgent[msg.sender]) revert UnrecognizedBridgeAgent();
            _;
        }
    
        /// @notice Modifier that verifies msg sender is the Root Chain's Core Router.
        modifier requiresCoreRootRouter() {
            if (msg.sender != coreRootRouterAddress) revert UnrecognizedCoreRootRouter();
            _;
        }
    
        /// @notice Modifier that verifies msg sender is the Root Chain's Local Branch Port.
        modifier requiresLocalBranchPort() {
            if (msg.sender != localBranchPortAddress) revert UnrecognizedLocalBranchPort();
            _;
        }
    }

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L16-L579)

</details>

### <a name="GAS-8-1693313026"></a>[GAS-8] The use of a logical AND in place of double if is slightly less gas efficient in instances where there isn't a corresponding else statement for the given if statement
Using a double if statement instead of logical AND (&&) can provide similar short-circuiting behavior whereas double if is slightly more efficient.

*Instances (1)*:
<details><summary> see instances </summary>

```solidity
File: src/RootBridgeAgent.sol

1141:         if (_amount == 0 && _deposit == 0) revert InvalidInputParams();
              if (_amount < _deposit) revert InvalidInputParams();

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1141-L1142)

</details>

### <a name="GAS-9-1693313275"></a>[GAS-9] Use assembly to validate `msg.sender`
We can use assembly to efficiently validate msg.sender with the least amount of opcodes necessary. For more details check the following report [Here](https://code4rena.com/reports/2023-05-juicebox#g-06-use-assembly-to-validate-msgsender)

*Instances (41)*:
<details><summary> see instances </summary>

```solidity
File: src/ArbitrumBranchBridgeAgent.sol

126:         if (msg.sender != address(this)) revert LayerZeroUnauthorizedEndpoint();

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchBridgeAgent.sol#L126-L126)

```solidity
File: src/BaseBranchRouter.sol

207:         if (msg.sender != bridgeAgentExecutorAddress) revert UnrecognizedBridgeAgentExecutor();

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L207-L207)

```solidity
File: src/BranchBridgeAgent.sol

354:         if (deposit.owner != msg.sender) revert NotDepositOwner();

424:         if (getDeposit[_depositNonce].owner != msg.sender) revert NotDepositOwner();

441:         if (deposit.owner != msg.sender) revert NotDepositOwner();

938:         if (msg.sender != address(this)) revert LayerZeroUnauthorizedEndpoint();

948:         if (msg.sender != localRouterAddress) revert UnrecognizedRouter();

954:         if (msg.sender != bridgeAgentExecutorAddress) revert UnrecognizedBridgeAgentExecutor();

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L954-L954)

```solidity
File: src/BranchPort.sol

487:         if (block.timestamp - lastManaged[msg.sender][_token] >= 1 days) {

487:         if (block.timestamp - lastManaged[msg.sender][_token] >= 1 days) {

542:         if (msg.sender != coreBranchRouterAddress) revert UnrecognizedCore();

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L542-L542)

```solidity
File: src/CoreRootRouter.sol

112:         if (msg.sender != IPort(rootPortAddress).getBridgeAgentManager(_rootBridgeAgent)) {

278:         require(msg.sender == rootPortAddress, "Only root port can call");

512:         if (msg.sender != bridgeAgentExecutorAddress) revert UnrecognizedBridgeAgentExecutor();

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L512-L512)

```solidity
File: src/MulticallRootRouter.sol

605:         if (msg.sender != bridgeAgentExecutorAddress) revert UnrecognizedBridgeAgentExecutor();

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L605-L605)

```solidity
File: src/RootBridgeAgent.sol

247:         if (msg.sender != settlementReference.owner) {

248:             if (msg.sender != address(IPort(localPortAddress).getUserAccount(settlementReference.owner))) {

248:             if (msg.sender != address(IPort(localPortAddress).getUserAccount(settlementReference.owner))) {

285:         if (msg.sender != settlementOwner) {

286:             if (msg.sender != address(IPort(localPortAddress).getUserAccount(settlementOwner))) {

286:             if (msg.sender != address(IPort(localPortAddress).getUserAccount(settlementOwner))) {

311:         if (msg.sender != settlementOwner) {

312:             if (msg.sender != address(IPort(localPortAddress).getUserAccount(settlementOwner))) {

312:             if (msg.sender != address(IPort(localPortAddress).getUserAccount(settlementOwner))) {

430:         if (!success) if (msg.sender == getBranchBridgeAgent[localChainId]) revert ExecutionFailure();

430:         if (!success) if (msg.sender == getBranchBridgeAgent[localChainId]) revert ExecutionFailure();

1199:         if (msg.sender != localRouterAddress) revert UnrecognizedRouter();

1205:         if (msg.sender != address(this)) revert LayerZeroUnauthorizedEndpoint();

1221:         if (msg.sender != bridgeAgentExecutorAddress) revert UnrecognizedExecutor();

1227:         if (msg.sender != localPortAddress) revert UnrecognizedPort();

1233:         if (msg.sender != IPort(localPortAddress).getBridgeAgentManager(address(this))) {

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1233-L1233)

```solidity
File: src/RootPort.sol

570:         if (msg.sender != coreRootRouterAddress) revert UnrecognizedCoreRootRouter();

576:         if (msg.sender != localBranchPortAddress) revert UnrecognizedLocalBranchPort();

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L576-L576)

```solidity
File: src/VirtualAccount.sol

162:             if (msg.sender != userAddress) {

162:             if (msg.sender != userAddress) {

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L162-L162)

```solidity
File: src/factories/ArbitrumBranchBridgeAgentFactory.sol

85:             msg.sender == localCoreBranchRouterAddress, "Only the Core Branch Router can create a new Bridge Agent."

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ArbitrumBranchBridgeAgentFactory.sol#L85-L85)

```solidity
File: src/factories/BranchBridgeAgentFactory.sol

121:             msg.sender == localCoreBranchRouterAddress, "Only the Core Branch Router can create a new Bridge Agent."

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/BranchBridgeAgentFactory.sol#L121-L121)

```solidity
File: src/factories/ERC20hTokenBranchFactory.sol

113:         if (msg.sender != localCoreRouterAddress) revert UnrecognizedCoreRouter();

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenBranchFactory.sol#L113-L113)

```solidity
File: src/factories/ERC20hTokenRootFactory.sol

97:         if (msg.sender != coreRootRouterAddress) {

98:             if (msg.sender != rootPortAddress) {

98:             if (msg.sender != rootPortAddress) {

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenRootFactory.sol#L98-L98)

</details>

### <a name="GAS-10-1694794906"></a>[GAS-10] Using mappings instead of arrays to avoid length checks save gas
Just by using a mapping, we get a gas saving of 2102 gas. When you read the value of an index of an array, solidity adds bytecode that checks that you are reading from a valid index (i.e an index strictly less than the length of the array) else it reverts with a panic error (Panic(0x32) to be precise). This prevents from reading unallocated or worse, allocated storage/memory locations.
Due to the way mappings are(simply a key => value pair), no check like that exists and we are able to read from the a storage slot directly.It’s important to note that when using mappings in this manner, your code should ensure that you are not reading an out of bound index of your canonical array.

*Instances (8)*:
<details><summary> see instances </summary>

```solidity
File: src/BranchPort.sol

35:     address[] public bridgeAgents;

45:     address[] public bridgeAgentFactories;

55:     address[] public strategyTokens;

71:     address[] public portStrategies;

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L71-L71)

```solidity
File: src/RootPort.sol

67:     address[] public bridgeAgents;

80:     address[] public bridgeAgentFactories;

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L80-L80)

```solidity
File: src/factories/ERC20hTokenBranchFactory.sol

23:     ERC20hTokenBranch[] public hTokens;

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenBranchFactory.sol#L23-L23)

```solidity
File: src/factories/ERC20hTokenRootFactory.sol

23:     ERC20hTokenRoot[] public hTokens;

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenRootFactory.sol#L23-L23)

</details>

### <a name="GAS-11-1693313317"></a>[GAS-11] Can make the variable outside the loop to save gas
Creating variables inside the loop consum more gas compared to declaring them outside and just reaffecting values to them inside the loop.

*Instances (9)*:
<details><summary> see instances </summary>

```solidity
File: src/BranchBridgeAgent.sol

//@audit variable `currentIterationOffset` is created inside a loop.
515:             uint256 currentIterationOffset = PARAMS_START + i;

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L515)

```solidity
File: src/RootBridgeAgent.sol

//@audit variable `_hToken` is created inside a loop.
320:             address _hToken = settlement.hTokens[i];

//@audit variable `_dstChainId` is created inside a loop.
325:                 uint24 _dstChainId = settlement.dstChainId;

//@audit variable `destChainId` is created inside a loop.
1076:             uint16 destChainId = _dstChainId;

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1076)

```solidity
File: src/RootBridgeAgentExecutor.sol

//@audit variable `currentIterationOffset` is created inside a loop.
283:             uint256 currentIterationOffset = PARAMS_START + i;

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L283)

```solidity
File: src/VirtualAccount.sol

//@audit variable `success` is created inside a loop.
71:             bool success;

//@audit variable `_call` is created inside a loop.
72:             Call calldata _call = calls[i];

//@audit variable `val` is created inside a loop.
92:             uint256 val = _call.value;

//@audit variable `success` is created inside a loop.
99:             bool success;

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L99)

</details>

