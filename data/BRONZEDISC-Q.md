## QA
---

### Layout Order [1]

- The best-practices for layout within a contract is the following order: state variables, events, modifiers, constructor and functions.

https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenRootFactory.sol

```solidity
// place this modifier before the constructor
96:    modifier requiresCoreRouterOrPort() {
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenBranchFactory.sol

```solidity
// place this modifier before the constructor
112:    modifier requiresCoreRouter() {
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol

```solidity
// place these modifiers before the constructor
557:    modifier requiresBridgeAgentFactory() {
563:    modifier requiresBridgeAgent() {
569:    modifier requiresCoreRootRouter() {
575:    modifier requiresLocalBranchPort() {
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol

```solidity
// place these modifiers before the constructor
206:    modifier requiresAgentExecutor() {
212:    modifier lock() {
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol

```solidity
// place these modifiers before the constructor
922:    modifier lock() {
930:    modifier requiresEndpoint(address _endpoint, bytes calldata _srcAddress) {
947:    modifier requiresRouter() {
953:    modifier requiresAgentExecutor() {
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol

```solidity
// place this modifier before the constructor
160:    modifier requiresApprovedCaller() {
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol

```solidity
// place these modifiers before the constructor
1190:    modifier lock() {
1198:    modifier requiresRouter() {
1204:    modifier requiresEndpoint(address _endpoint, uint16 _srcChain, bytes calldata _srcAddress) virtual {
1220:    modifier requiresAgentExecutor() {
1226:    modifier requiresPort() {
1232:    modifier requiresManager() {
```

```solidity
// place this modifier before the constructor
511:    modifier requiresExecutor() {

// place these errors before the constructor
520:    error InvalidChainId();
522:    error UnauthorizedChainId();
524:    error UnauthorizedCallerNotManager();
526:    error TokenAlreadyAdded();
528:    error UnrecognizedGlobalToken();
530:    error UnrecognizedBridgeAgentFactory();
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol

```solidity
// place these modifiers before the constructor
590:    modifier lock() {
598:    modifier requiresExecutor() {
```

---

### Function Visibility [2]

- Order of Functions: Ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. Functions should be grouped according to their visibility and ordered: constructor, receive function (if exists), fallback function (if exists), external, public, internal, private. Within a grouping, place the view and pure functions last.

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol

```solidity
// place these internal functions for last since there are no private ones.
190:    function _getLocalToken(address _localAddress, uint256 _srcChainId, uint256 _dstChainId)
359:    function addVirtualAccount(address _user) internal returns (VirtualAccount newAccount) {

// place this public function right after all the external ones.
166:    function renounceOwnership() public payable override onlyOwner {
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol

```solidity
// place these private functions for last.
712:    function _execute(uint256 _settlementNonce, bytes memory _calldata) private {
730:    function _execute(bool _hasFallbackToggled, uint32 _settlementNonce, address _refundee, bytes memory _calldata)
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol

```solidity
// place this public function right after external ones.
85:    function payableCall(PayableCall[] calldata calls) public payable returns (bytes[] memory returnData) {
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol

```solidity
// place  this public function after all the external ones
351:    function bridgeIn(address _recipient, DepositParams memory _dParams, uint256 _srcChainId)

// place these private functions for last.
749:    function _execute(uint256 _depositNonce, bytes memory _calldata, uint16 _srcChainId) private {
768:    function _execute(
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumCoreBranchRouter.sol

```solidity
// place this internal after all the external functions.
85:    function _receiveAddBridgeAgent(
```

---

### natSpec missing [3]

Some functions are missing @params or @returns. Specification Format.” These are written with a triple slash (///) or a double asterisk block(/** ... */) directly above function declarations or statements to generate documentation in JSON format for developers and end-users. It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). These comments contain different types of tags:
- @title: A title that should describe the contract/interface @author: The name of the author (for contract, interface) 
- @notice: Explain to an end user what this does (for contract, interface, function, public state variable, event) 
- @dev: Explain to a developer any extra details (for contract, interface, function, state variable, event) 
- @param: Documents a parameter (just like in doxygen) and must be followed by parameter name (for function, event)
- @return: Documents the return variables of a contract’s function (function, public state variable)
- @inheritdoc: Copies all missing tags from the base function and must be followed by the contract name (for function, public state variable)
- @custom…: Custom tag, semantics is application-defined (for everywhere)

https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/IRootBridgeAgentFactory.sol

```solidity
16:    function createBridgeAgent(address newRootRouterAddress) external returns (address newBridgeAgent);
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/IBranchRouter.sol

```solidity
// @return missing
27:    function localPortAddress() external view returns (address);
30:    function localBridgeAgentAddress() external view returns (address);
33:    function bridgeAgentExecutorAddress() external view returns (address);
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/IMulticall2.sol

```solidity
9:  interface IMulticall2 {
10:    struct Call {
15:    struct Result {
20:    function aggregate(Call[] memory calls) external returns (uint256 blockNumber, bytes[] memory returnData);
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/IVirtualAccount.sol

```solidity
// @params missing
7:  struct Call {
13:  struct PayableCall {
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/ILayerZeroEndpoint.sol

```solidity
// @return missing
43:    function getInboundNonce(uint16 _srcChainId, bytes calldata _srcAddress) external view returns (uint64);
47:    function getOutboundNonce(uint16 _dstChainId, address _srcAddress) external view returns (uint64);
55:    function estimateFees(
64:    function getChainId() external view returns (uint16);
75:    function hasStoredPayload(uint16 _srcChainId, bytes calldata _srcAddress) external view returns (bool);
79:    function getSendLibraryAddress(address _userApplication) external view returns (address);
83:    function getReceiveLibraryAddress(address _userApplication) external view returns (address);
98:    function getConfig(uint16 _version, uint16 _chainId, address _userApplication, uint256 _configType)
105:    function getSendVersion(address _userApplication) external view returns (uint16);
109:    function getReceiveVersion(address _userApplication) external view returns (uint16);
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/IRootPort.sol

```solidity
11:    function bridgeAgentAddress() external view returns (address);
12:    function hTokenFactoryAddress() external view returns (address);
13:    function setCoreBranch(
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/BridgeAgentStructs.sol

```solidity
13:  struct Deposit {
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ArbitrumBranchBridgeAgentFactory.sol

```solidity
// @param missing
79:    function createBridgeAgent(
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/BranchBridgeAgentFactory.sol

```solidity
// @param missing
115:    function createBridgeAgent(
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/token/ERC20hTokenBranch.sol

```solidity
13:    constructor(
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol

```solidity
304:    function bridgeToLocalBranchFromRoot(address _to, address _hToken, uint256 _amount)

// @return missing
359:    function addVirtualAccount(address _user) internal returns (VirtualAccount newAccount) {
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol

```solidity
24:    function deploy(

// @params missing
930:    modifier requiresEndpoint(address _endpoint, bytes calldata _srcAddress) {
936:    function _requiresEndpoint(address _endpoint, bytes calldata _srcAddress) internal view virtual {
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouterLibZip.sol

```solidity
29:    constructor(uint256 _localChainId, address _localPortAddress, address _multicallAddress)
37:    function _decode(bytes calldata data) internal pure override returns (bytes memory) {
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol

```solidity
147:    function isContract(address addr) internal view returns (bool) {
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol

```solidity
14:    function deploy(address _rootBridgeAgent) external returns (address) {
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol

```solidity
83:    function initialize(address _bridgeAgentAddress, address _hTokenFactory) external onlyOwner {
520:    error InvalidChainId();
522:    error UnauthorizedChainId();
524:    error UnauthorizedCallerNotManager();
526:    error TokenAlreadyAdded();
528:    error UnrecognizedGlobalToken();
530:    error UnrecognizedBridgeAgentFactory();
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol

```solidity
581:    function _decode(bytes calldata data) internal pure virtual returns (bytes memory) {
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchBridgeAgent.sol

```solidity
10:  library DeployArbitrumBranchBridgeAgent {
11:    function deploy(uint16 _localChainId, address _daoAddress, address _localRouterAddress, address _localPortAddress)
```

---

### State variable and function names [4]

- Variables should be named according to their specifications
- private and internal `variables` should preppend with `underline`
- private and internal `functions` should preppend with `underline`
- constant state variables should be UPPER_CASE

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol

```solidity
//  private and internal `variables` should preppend with `underline`
101:    uint256 internal _unlocked = 1;
```