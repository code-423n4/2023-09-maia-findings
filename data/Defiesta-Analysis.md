Instead of using error strings, to reduce deployment and runtime cost, you should use Custom Errors.

ArbitrumBranchPort.sol
```
    constructor(uint16 _localChainId, address _rootPortAddress, address _owner) BranchPort(_owner) {
        require(_rootPortAddress != address(0), "Root Port Address cannot be 0");

        localChainId = _localChainId;
        rootPortAddress = _rootPortAddress;
    }
```

BaseBranchRouter.sol
```
    function initialize(address _localBridgeAgentAddress) external onlyOwner {
        require(_localBridgeAgentAddress != address(0), "Bridge Agent address cannot be 0");
        localBridgeAgentAddress = _localBridgeAgentAddress;
        localPortAddress = IBridgeAgent(_localBridgeAgentAddress).localPortAddress();
        bridgeAgentExecutorAddress = IBridgeAgent(_localBridgeAgentAddress).bridgeAgentExecutorAddress();
        
        renounceOwnership();
    }
```

BranchBridgeAgent.sol
```
constructor(
        uint16 _rootChainId,
        uint16 _localChainId,
        address _rootBridgeAgentAddress,
        address _lzEndpointAddress,
        address _localRouterAddress,
        address _localPortAddress
    ) {
        require(_rootBridgeAgentAddress != address(0), "Root Bridge Agent Address cannot be the zero address.");
        require(
            _lzEndpointAddress != address(0) || _rootChainId == _localChainId,
            "Layerzero Endpoint Address cannot be the zero address."
        );
        require(_localRouterAddress != address(0), "Local Router Address cannot be the zero address.");
        require(_localPortAddress != address(0), "Local Port Address cannot be the zero address.");

        localChainId = _localChainId;
        rootChainId = _rootChainId;
        rootBridgeAgentAddress = _rootBridgeAgentAddress;
        lzEndpointAddress = _lzEndpointAddress;
        localRouterAddress = _localRouterAddress;
        localPortAddress = _localPortAddress;
        bridgeAgentExecutorAddress = DeployBranchBridgeAgentExecutor.deploy();
        depositNonce = 1;

        rootBridgeAgentPath = abi.encodePacked(_rootBridgeAgentAddress, address(this));
    }
```

BranchPort.sol
```
    constructor(address _owner) {
        require(_owner != address(0), "Owner is zero address");
        _initializeOwner(_owner);
    }
```

```
    function initialize(address _coreBranchRouter, address _bridgeAgentFactory) external virtual onlyOwner {
        require(coreBranchRouterAddress == address(0), "Contract already initialized");
        require(!isBridgeAgentFactory[_bridgeAgentFactory], "Contract already initialized");

        require(_coreBranchRouter != address(0), "CoreBranchRouter is zero address");
        require(_bridgeAgentFactory != address(0), "BridgeAgentFactory is zero address");

        coreBranchRouterAddress = _coreBranchRouter;
        isBridgeAgentFactory[_bridgeAgentFactory] = true;
        bridgeAgentFactories.push(_bridgeAgentFactory);
    }
```

```
function setCoreRouter(address _newCoreRouter) external override requiresCoreRouter {
        require(coreBranchRouterAddress != address(0), "CoreRouter address is zero");
        require(_newCoreRouter != address(0), "New CoreRouter address is zero");
        coreBranchRouterAddress = _newCoreRouter;
    }
```

CoreRootRouter.sol
```
function initialize(address _bridgeAgentAddress, address _hTokenFactory) external onlyOwner {
        require(_setup, "Contract is already initialized");
        _setup = false;
        bridgeAgentAddress = payable(_bridgeAgentAddress);
        bridgeAgentExecutorAddress = IBridgeAgent(_bridgeAgentAddress).bridgeAgentExecutorAddress();
        hTokenFactoryAddress = _hTokenFactory;
    }
```
```
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
```

MulticalRootRouter.sol
```
    constructor(uint256 _localChainId, address _localPortAddress, address _multicallAddress) {
        require(_localPortAddress != address(0), "Local Port Address cannot be 0");
        require(_multicallAddress != address(0), "Multicall Address cannot be 0");

        localChainId = _localChainId;
        localPortAddress = _localPortAddress;
        multicallAddress = _multicallAddress;
        _initializeOwner(msg.sender);
    }
```

RootBridgeAgent.sol
```
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
```

RootPort.sol
```
function initialize(address _bridgeAgentFactory, address _coreRootRouter) external onlyOwner {
        require(_bridgeAgentFactory != address(0), "Bridge Agent Factory cannot be 0 address.");
        require(_coreRootRouter != address(0), "Core Root Router cannot be 0 address.");
        require(_setup, "Setup ended.");
        _setup = false;

        isBridgeAgentFactory[_bridgeAgentFactory] = true;
        bridgeAgentFactories.push(_bridgeAgentFactory);

        coreRootRouterAddress = _coreRootRouter;
    }
```
```
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
```

token/ERC20hTokenRoot.sol
```
constructor(
        uint16 _localChainId,
        address _factoryAddress,
        address _rootPortAddress,
        string memory _name,
        string memory _symbol,
        uint8 _decimals
    ) ERC20(string(string.concat(_name)), string(string.concat(_symbol)), _decimals) {
        require(_rootPortAddress != address(0), "Root Port Address cannot be 0");
        require(_factoryAddress != address(0), "Factory Address cannot be 0");

        localChainId = _localChainId;
        factoryAddress = _factoryAddress;
        _initializeOwner(_rootPortAddress);
    }
```

factories/ArbitrumBranchBridgeAgentFactory.sol
```
function initialize(address _coreRootBridgeAgent) external override onlyOwner {
        require(_coreRootBridgeAgent != address(0), "Core Root Bridge Agent Address cannot be 0");

        address newCoreBridgeAgent = address(
            DeployArbitrumBranchBridgeAgent.deploy(
                rootChainId, _coreRootBridgeAgent, localCoreBranchRouterAddress, localPortAddress
            )
        );

        IPort(localPortAddress).addBridgeAgent(newCoreBridgeAgent);

        renounceOwnership();
    }
```

factories/BranchBridgeAgentFactory.sol
```
    constructor(
        uint16 _localChainId,
        uint16 _rootChainId,
        address _rootBridgeAgentFactoryAddress,
        address _lzEndpointAddress,
        address _localCoreBranchRouterAddress,
        address _localPortAddress,
        address _owner
    ) {
        require(_rootBridgeAgentFactoryAddress != address(0), "Root Bridge Agent Factory Address cannot be 0");
        require(
            _lzEndpointAddress != address(0) || _rootChainId == _localChainId,
            "Layerzero Endpoint Address cannot be the zero address."
        );
        require(_localCoreBranchRouterAddress != address(0), "Core Branch Router Address cannot be 0");
        require(_localPortAddress != address(0), "Port Address cannot be 0");
        require(_owner != address(0), "Owner cannot be 0");

        localChainId = _localChainId;
        rootChainId = _rootChainId;
        rootBridgeAgentFactoryAddress = _rootBridgeAgentFactoryAddress;
        lzEndpointAddress = _lzEndpointAddress;
        localCoreBranchRouterAddress = _localCoreBranchRouterAddress;
        localPortAddress = _localPortAddress;
        _initializeOwner(_owner);
    }
```

factories/ERC20hTokenBranchFactory.sol
```
    constructor(uint16 _localChainId, address _localPortAddress, string memory _chainName, string memory _chainSymbol) {
        require(_localPortAddress != address(0), "Port address cannot be 0");
        chainName = string.concat(_chainName, " Ulysses");
        chainSymbol = string.concat(_chainSymbol, "-u");
        localChainId = _localChainId;
        localPortAddress = _localPortAddress;
        _initializeOwner(msg.sender);
    }

   function initialize(address _wrappedNativeTokenAddress, address _coreRouter) external onlyOwner {
        require(_coreRouter != address(0), "CoreRouter address cannot be 0");

        ERC20hTokenBranch newToken = new ERC20hTokenBranch(
            chainName,
            chainSymbol,
            ERC20(_wrappedNativeTokenAddress).name(),
            ERC20(_wrappedNativeTokenAddress).symbol(),
            ERC20(_wrappedNativeTokenAddress).decimals(),
            localPortAddress
        );

        hTokens.push(newToken);

        localCoreRouterAddress = _coreRouter;

        renounceOwnership();
    }
```

factories/ERC20hTokenRootFactory.sol
```
    constructor(uint16 _localChainId, address _rootPortAddress) {
        require(_rootPortAddress != address(0), "Root Port Address cannot be 0");
        localChainId = _localChainId;
        rootPortAddress = _rootPortAddress;
        _initializeOwner(msg.sender);
    }

    function initialize(address _coreRouter) external onlyOwner {
        require(_coreRouter != address(0), "CoreRouter address cannot be 0");
        coreRootRouterAddress = _coreRouter;
        renounceOwnership();
    }
```

factories/RootBridgeAgentFactory.sol
```
    constructor(uint16 _rootChainId, address _lzEndpointAddress, address _rootPortAddress) {
        require(_rootPortAddress != address(0), "Root Port Address cannot be 0");

        rootChainId = _rootChainId;
        lzEndpointAddress = _lzEndpointAddress;
        rootPortAddress = _rootPortAddress;
    }
```



### Time spent:
3 hours