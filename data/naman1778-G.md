## [G-01] abi.encode() is less efficient than abi.encodePacked()

Changing abi.encode function to abi.encodePacked can save gas since the abi.encode function pads extra null bytes at the end of the call data, which is unnecessary. Also, in general, abi.encodePacked is more gas-efficient (see [Solidity-Encode-Gas-Comparison](https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison)).

    File: src/ArbitrumCoreBranchRouter.sol	
    
    53: bytes memory params = abi.encode(

    112: bytes memory data = abi.encode(newBridgeAgent, _rootBridgeAgent);

    diff --git a/src/ArbitrumCoreBranchRouter.sol b/src/ArbitrumCoreBranchRouter.sol
    index 2abff5f..e3840dd 100644
    --- a/src/ArbitrumCoreBranchRouter.sol
    +++ b/src/ArbitrumCoreBranchRouter.sol
    @@ -50,7 +50,7 @@ contract ArbitrumCoreBranchRouter is CoreBranchRouter {
         ///@inheritdoc CoreBranchRouter
         function addLocalToken(address _underlyingAddress, GasParams calldata) external payable override {
             //Encode Data, no need to create local token since we are already in the global environment
    -        bytes memory params = abi.encode(
    +        bytes memory params = abi.encodePacked(
                 _underlyingAddress,
                 address(0),
                 string.concat("Arbitrum Ulysses ", ERC20(_underlyingAddress).name()),
    @@ -109,7 +109,7 @@ contract ArbitrumCoreBranchRouter is CoreBranchRouter {
             }

             // Encode Params
    -        bytes memory data = abi.encode(newBridgeAgent, _rootBridgeAgent);
    +        bytes memory data = abi.encodePacked(newBridgeAgent, _rootBridgeAgent);

             // Pack FuncId and Params to create payload
             bytes memory payload = abi.encodePacked(bytes1(0x04), data);

https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumCoreBranchRouter.sol

    File: src/CoreRootRouter.sol	

    126: bytes memory params = abi.encode(

    168: bytes memory params = abi.encode(_branchBridgeAgentFactory);

    193: bytes memory params = abi.encode(_branchBridgeAgent);

    220: bytes memory params = abi.encode(_underlyingToken, _minimumReservesRatio);

    251: bytes memory params = abi.encode(_portStrategy, _underlyingToken, _dailyManagementLimit, _isUpdateDailyLimit);

    281: bytes memory params = abi.encode(_coreBranchRouter, _coreBranchBridgeAgent);

    424: bytes memory params = abi.encode(

    diff --git a/src/CoreRootRouter.sol b/src/CoreRootRouter.sol
    index 3104a2a..97539fe 100644
    --- a/src/CoreRootRouter.sol
    +++ b/src/CoreRootRouter.sol
    @@ -123,7 +123,7 @@ contract CoreRootRouter is IRootRouter, Ownable {
             if (!IBridgeAgent(_rootBridgeAgent).isBranchBridgeAgentAllowed(_dstChainId)) revert UnauthorizedChainId();

             // Encode CallData
    -        bytes memory params = abi.encode(
    +        bytes memory params = abi.encodePacked(
                 _newBranchRouter,
                 _branchBridgeAgentFactory,
                 _rootBridgeAgent,
    @@ -165,7 +165,7 @@ contract CoreRootRouter is IRootRouter, Ownable {
             }

             // Encode CallData
    -        bytes memory params = abi.encode(_branchBridgeAgentFactory);
    +        bytes memory params = abi.encodePacked(_branchBridgeAgentFactory);

             // Pack funcId into data
             bytes memory payload = abi.encodePacked(bytes1(0x03), params);
    @@ -190,7 +190,7 @@ contract CoreRootRouter is IRootRouter, Ownable {
             GasParams calldata _gParams
         ) external payable onlyOwner {
             //Encode CallData
    -        bytes memory params = abi.encode(_branchBridgeAgent);
    +        bytes memory params = abi.encodePacked(_branchBridgeAgent);

             // Pack funcId into data
             bytes memory payload = abi.encodePacked(bytes1(0x04), params);
    @@ -217,7 +217,7 @@ contract CoreRootRouter is IRootRouter, Ownable {
             GasParams calldata _gParams
         ) external payable onlyOwner {
             // Encode CallData
    -        bytes memory params = abi.encode(_underlyingToken, _minimumReservesRatio);
    +        bytes memory params = abi.encodePacked(_underlyingToken, _minimumReservesRatio);

             // Pack funcId into data
             bytes memory payload = abi.encodePacked(bytes1(0x05), params);
    @@ -248,7 +248,7 @@ contract CoreRootRouter is IRootRouter, Ownable {
             GasParams calldata _gParams
         ) external payable onlyOwner {
             // Encode CallData
    -        bytes memory params = abi.encode(_portStrategy, _underlyingToken, _dailyManagementLimit, _isUpdateDailyLimit);
    +        bytes memory params = abi.encodePacked(_portStrategy, _underlyingToken, _dailyManagementLimit, _isUpdateDailyLimit);

             // Pack funcId into data
             bytes memory payload = abi.encodePacked(bytes1(0x06), params);
    @@ -278,7 +278,7 @@ contract CoreRootRouter is IRootRouter, Ownable {
             require(msg.sender == rootPortAddress, "Only root port can call");

             // Encode CallData
    -        bytes memory params = abi.encode(_coreBranchRouter, _coreBranchBridgeAgent);
    +        bytes memory params = abi.encodePacked(_coreBranchRouter, _coreBranchBridgeAgent);

             // Pack funcId into data
             bytes memory payload = abi.encodePacked(bytes1(0x07), params);
    @@ -421,7 +421,7 @@ contract CoreRootRouter is IRootRouter, Ownable {
             }

             // Encode CallData
    -        bytes memory params = abi.encode(
    +        bytes memory params = abi.encodePacked(
                 _globalAddress,
                 ERC20(_globalAddress).name(),
                 ERC20(_globalAddress).symbol(),

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol

    File: src/CoreBranchRouter.sol	

    48: bytes memory params = abi.encode(msg.sender, _globalAddress, _dstChainId, [_gParams[1], _gParams[2]]);

    72: bytes memory params = abi.encode(_underlyingAddress, newToken, newToken.name(), newToken.symbol(), decimals);

    178: bytes memory params = abi.encode(_globalAddress, newToken);

    226: bytes memory params = abi.encode(newBridgeAgent, _rootBridgeAgent);

    diff --git a/src/CoreBranchRouter.sol b/src/CoreBranchRouter.sol
    index e0503b6..f8dd199 100644
    --- a/src/CoreBranchRouter.sol
    +++ b/src/CoreBranchRouter.sol
    @@ -45,7 +45,7 @@ contract CoreBranchRouter is ICoreBranchRouter, BaseBranchRouter {
             payable
         {
             // Encode Call Data
    -        bytes memory params = abi.encode(msg.sender, _globalAddress, _dstChainId, [_gParams[1], _gParams[2]]);
    +        bytes memory params = abi.encodePacked(msg.sender, _globalAddress, _dstChainId, [_gParams[1], _gParams[2]]);
    
             // Pack FuncId
             bytes memory payload = abi.encodePacked(bytes1(0x01), params);
    @@ -69,7 +69,7 @@ contract CoreBranchRouter is ICoreBranchRouter, BaseBranchRouter {
             );
    
             //Encode Data
    -        bytes memory params = abi.encode(_underlyingAddress, newToken, newToken.name(), newToken.symbol(), decimals);
    +        bytes memory params = abi.encodePacked(_underlyingAddress, newToken, newToken.name(), newToken.symbol(), decimals);
    
             // Pack FuncId
             bytes memory payload = abi.encodePacked(bytes1(0x02), params);
    @@ -175,7 +175,7 @@ contract CoreBranchRouter is ICoreBranchRouter, BaseBranchRouter {
             ERC20hToken newToken = ITokenFactory(hTokenFactoryAddress).createToken(_name, _symbol, _decimals, false);
    
             // Encode Data
    -        bytes memory params = abi.encode(_globalAddress, newToken);
    +        bytes memory params = abi.encodePacked(_globalAddress, newToken);
    
             // Pack FuncId
             bytes memory payload = abi.encodePacked(bytes1(0x03), params);
    @@ -223,7 +223,7 @@ contract CoreBranchRouter is ICoreBranchRouter, BaseBranchRouter {
             }
    
             // Encode Data
    -        bytes memory params = abi.encode(newBridgeAgent, _rootBridgeAgent);
    +        bytes memory params = abi.encodePacked(newBridgeAgent, _rootBridgeAgent);
    
             // Pack FuncId
             bytes memory payload = abi.encodePacked(bytes1(0x04), params);

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol

    File: src/BranchBridgeAgent.sol	

    471: bytes memory params = abi.encode(_settlementNonce, msg.sender, _params, _gParams[1]);
    
    diff --git a/src/BranchBridgeAgent.sol b/src/BranchBridgeAgent.sol
    index b076d2d..7cbfbcb 100644
    --- a/src/BranchBridgeAgent.sol
    +++ b/src/BranchBridgeAgent.sol
    @@ -468,7 +468,7 @@ contract BranchBridgeAgent is IBranchBridgeAgent, BridgeAgentConstants {
             bool _hasFallbackToggled
         ) external payable virtual override lock {
             // Encode Retry Settlement Params
    -        bytes memory params = abi.encode(_settlementNonce, msg.sender, _params, _gParams[1]);
    +        bytes memory params = abi.encodePacked(_settlementNonce, msg.sender, _params, _gParams[1]);

             // Prepare payload for cross-chain call.
             bytes memory payload = abi.encodePacked(_hasFallbackToggled ? bytes1(0x87) : bytes1(0x07), params);

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol

    File: src/RootPort.sol	

    362: newAccount = new VirtualAccount{salt: keccak256(abi.encode(_user))}(_user, address(this));

    diff --git a/src/RootPort.sol b/src/RootPort.sol
    index c379b6e..8862198 100644
    --- a/src/RootPort.sol
    +++ b/src/RootPort.sol
    @@ -359,7 +359,7 @@ contract RootPort is Ownable, IRootPort {
         function addVirtualAccount(address _user) internal returns (VirtualAccount newAccount) {
             if (_user == address(0)) revert InvalidUserAddress();

    -        newAccount = new VirtualAccount{salt: keccak256(abi.encode(_user))}(_user, address(this));
    +        newAccount = new VirtualAccount{salt: keccak256(abi.encodePacked(_user))}(_user, address(this));
             getUserAccount[_user] = newAccount;

             emit VirtualAccountCreated(_user, address(newAccount));

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized();
            c1.optimized();
        }
    }
    contract Contract0 {
        string a = "Code4rena";
        function not_optimized() public returns(bytes32){
            return keccak256(abi.encode(a));
        }
    }
    contract Contract1 {
        string a = "Code4rena";
        function optimized() public returns(bytes32){
            return keccak256(abi.encodePacked(a));
        }
    }

#### Gas Report

| Contract0 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 101871                                    | 683             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| not_optimized                             | 2661            | 2661 | 2661   | 2661 | 1       |


| Contract1 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 99465                                     | 671             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| optimized                                 | 2608            | 2608 | 2608   | 2608 | 1       |

## [G-02] Use assembly to write address storage values

When writing value for variables whose type is address, make use of assembly code instead of solidity code.

    File: src/ArbitrumBranchPort.sol	

    42: rootPortAddress = _rootPortAddress;

    diff --git a/src/ArbitrumBranchPort.sol b/src/ArbitrumBranchPort.sol
    index 5777422..5661698 100644
    --- a/src/ArbitrumBranchPort.sol
    +++ b/src/ArbitrumBranchPort.sol
    @@ -39,7 +39,9 @@ contract ArbitrumBranchPort is BranchPort, IArbitrumBranchPort {
             require(_rootPortAddress != address(0), "Root Port Address cannot be 0");

             localChainId = _localChainId;
    -        rootPortAddress = _rootPortAddress;
    +        assembly{
    +            sstore(rootPortAddress.slot, _rootPortAddress)
    +        }
         }

         /*///////////////////////////////////////////////////////////////

https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol

    File: src/MulticallRootRouter.sol	

    97: localPortAddress = _localPortAddress;

    98: multicallAddress = _multicallAddress;

    112: bridgeAgentAddress = payable(_bridgeAgentAddress);

    113: bridgeAgentExecutorAddress = IBridgeAgent(_bridgeAgentAddress).bridgeAgentExecutorAddress();

    diff --git a/src/MulticallRootRouter.sol b/src/MulticallRootRouter.sol
    index 0621f81..aa8a15c 100644
    --- a/src/MulticallRootRouter.sol
    +++ b/src/MulticallRootRouter.sol
    @@ -94,8 +94,12 @@ contract MulticallRootRouter is IRootRouter, Ownable {
             require(_multicallAddress != address(0), "Multicall Address cannot be 0");

             localChainId = _localChainId;
    -        localPortAddress = _localPortAddress;
    -        multicallAddress = _multicallAddress;
    +        assembly{
    +            sstore(localPortAddress.slot,_localPortAddress)
    +        }
    +        assembly{
    +            sstore(multicallAddress.slot,_multicallAddress)
    +        }
             _initializeOwner(msg.sender);
         }

    @@ -108,9 +112,12 @@ contract MulticallRootRouter is IRootRouter, Ownable {
          */
         function initialize(address _bridgeAgentAddress) external onlyOwner {
             require(_bridgeAgentAddress != address(0), "Bridge Agent Address cannot be 0");
    -
    -        bridgeAgentAddress = payable(_bridgeAgentAddress);
    -        bridgeAgentExecutorAddress = IBridgeAgent(_bridgeAgentAddress).bridgeAgentExecutorAddress();
    +        assembly{
    +            sstore(bridgeAgentAddress.slot,payable(_bridgeAgentAddress))
    +        }
    +        assembly{
    +            sstore(bridgeAgentExecutorAddress.slot,IBridgeAgent(_bridgeAgentAddress).bridgeAgentExecutorAddress())
    +        }
             renounceOwnership();
         }


https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol

    File: src/CoreRootRouter.sol	

    73: rootPortAddress = _rootPortAddress;

    86: bridgeAgentAddress = payable(_bridgeAgentAddress);

    87: bridgeAgentExecutorAddress = IBridgeAgent(_bridgeAgentAddress).bridgeAgentExecutorAddress();

    88: hTokenFactoryAddress = _hTokenFactory;

    diff --git a/src/CoreRootRouter.sol b/src/CoreRootRouter.sol
    index 3104a2a..c2536c8 100644
    --- a/src/CoreRootRouter.sol
    +++ b/src/CoreRootRouter.sol
    @@ -69,7 +69,9 @@ contract CoreRootRouter is IRootRouter, Ownable {
          * @param _rootPortAddress address of the Root Port.
          */
         constructor(uint256 _rootChainId, address _rootPortAddress) {
    -        rootChainId = _rootChainId;
    +        assembly{
    +            sstore(rootChainId.slot,_rootChainId)
    +        }
             rootPortAddress = _rootPortAddress;

             _initializeOwner(msg.sender);
    @@ -83,9 +85,15 @@ contract CoreRootRouter is IRootRouter, Ownable {
         function initialize(address _bridgeAgentAddress, address _hTokenFactory) external onlyOwner {
             require(_setup, "Contract is already initialized");
             _setup = false;
    -        bridgeAgentAddress = payable(_bridgeAgentAddress);
    -        bridgeAgentExecutorAddress = IBridgeAgent(_bridgeAgentAddress).bridgeAgentExecutorAddress();
    -        hTokenFactoryAddress = _hTokenFactory;
    +        assembly{
    +            sstore(bridgeAgentAddress.slot,payable(_bridgeAgentAddress))
    +        }
    +        assembly{
    +            sstore(bridgeAgentExecutorAddress.slot,IBridgeAgent(_bridgeAgentAddress).bridgeAgentExecutorAddress())
    +        }
    +        assembly{
    +            sstore(hTokenFactoryAddress.slot,_hTokenFactory)
    +        }
         }

         /*///////////////////////////////////////////////////////////////

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol

    File: src/CoreBranchRouter.sol	

    31: hTokenFactoryAddress = _hTokenFactoryAddress;

    diff --git a/src/CoreBranchRouter.sol b/src/CoreBranchRouter.sol
    index e0503b6..279d9c6 100644
    --- a/src/CoreBranchRouter.sol
    +++ b/src/CoreBranchRouter.sol
    @@ -28,7 +28,9 @@ contract CoreBranchRouter is ICoreBranchRouter, BaseBranchRouter {
          * @param _hTokenFactoryAddress Branch hToken Factory Address.
          */
         constructor(address _hTokenFactoryAddress) BaseBranchRouter() {
    -        hTokenFactoryAddress = _hTokenFactoryAddress;
    +        assembly{
    +            sstore(hTokenFactoryAddress.slot,_hTokenFactoryAddress)
    +        }
         }

         /*///////////////////////////////////////////////////////////////

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol

    File: src/RootBridgeAgent.sol	

    115: factoryAddress = msg.sender;

    117: lzEndpointAddress = _lzEndpointAddress;

    118: localPortAddress = _localPortAddress;

    119: localRouterAddress = _localRouterAddress;

    120: bridgeAgentExecutorAddress = DeployRootBridgeAgentExecutor.deploy(address(this));

    diff --git a/src/RootBridgeAgent.sol b/src/RootBridgeAgent.sol
    index a6ad0ef..dc8ca2f 100644
    --- a/src/RootBridgeAgent.sol
    +++ b/src/RootBridgeAgent.sol
    @@ -112,13 +112,23 @@ contract RootBridgeAgent is IRootBridgeAgent, BridgeAgentConstants {
             require(_localPortAddress != address(0), "Port Address cannot be zero address");
             require(_localRouterAddress != address(0), "Router Address cannot be zero address");

    -        factoryAddress = msg.sender;
    +        assembly{
    +            sstore(factoryAddress.slot,msg.sender)
    +        }
             localChainId = _localChainId;
    -        lzEndpointAddress = _lzEndpointAddress;
    -        localPortAddress = _localPortAddress;
    -        localRouterAddress = _localRouterAddress;
    -        bridgeAgentExecutorAddress = DeployRootBridgeAgentExecutor.deploy(address(this));
    -        settlementNonce = 1;
    +        assembly{
    +            sstore(lzEndpointAddress.slot,_lzEndpointAddress)
    +        }
    +        assembly{
    +            sstore(localPortAddress.slot,_localPortAddress)
    +        }
    +        assembly{
    +            sstore(localRouterAddress.slot,_localRouterAddress)
    +        }
    +        assembly{
    +            sstore(bridgeAgentExecutorAddress.slot,DeployRootBridgeAgentExecutor.deploy(address(this)))
    +        }
    +       settlementNonce = 1;
         }

         /*///////////////////////////////////////////////////////////////

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol

    File: src/VirtualAccount.sol	

    36: userAddress = _userAddress;

    37: localPortAddress = _localPortAddress;

    diff --git a/src/VirtualAccount.sol b/src/VirtualAccount.sol
    index f6a9134..c05d3ed 100644
    --- a/src/VirtualAccount.sol
    +++ b/src/VirtualAccount.sol
    @@ -33,8 +33,12 @@ contract VirtualAccount is IVirtualAccount, ERC1155Receiver {
          * @param _localPortAddress Address of the root port contract.
          */
         constructor(address _userAddress, address _localPortAddress) {
    -        userAddress = _userAddress;
    -        localPortAddress = _localPortAddress;
    +        assembly{
    +            sstore(userAddress.slot,_userAddress)
    +        }
    +        assembly{
    +            sstore(localPortAddress.slot,_localPortAddress)
    +        }
         }

         /*//////////////////////////////////////////////////////////////

https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol

    File: src/BranchPort.sol	

    129: coreBranchRouterAddress = _coreBranchRouter;

    diff --git a/src/BranchPort.sol b/src/BranchPort.sol
    index ba60acc..94524d0 100644
    --- a/src/BranchPort.sol
    +++ b/src/BranchPort.sol
    @@ -126,7 +126,9 @@ contract BranchPort is Ownable, IBranchPort {
             require(_coreBranchRouter != address(0), "CoreBranchRouter is zero address");
             require(_bridgeAgentFactory != address(0), "BridgeAgentFactory is zero address");

    -        coreBranchRouterAddress = _coreBranchRouter;
    +        assembly{
    +            sstore(coreBranchRouterAddress.slot,_coreBranchRouter)
    +        }
             isBridgeAgentFactory[_bridgeAgentFactory] = true;
             bridgeAgentFactories.push(_bridgeAgentFactory);
         }

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol

    File: src/BranchBridgeAgent.sol	

    135: rootBridgeAgentAddress = _rootBridgeAgentAddress;

    136: lzEndpointAddress = _lzEndpointAddress;

    137: localRouterAddress = _localRouterAddress;

    138: localPortAddress = _localPortAddress;
    
    139: bridgeAgentExecutorAddress = DeployBranchBridgeAgentExecutor.deploy();

    diff --git a/src/BranchBridgeAgent.sol b/src/BranchBridgeAgent.sol
    index b076d2d..cf1db9f 100644
    --- a/src/BranchBridgeAgent.sol
    +++ b/src/BranchBridgeAgent.sol
    @@ -132,11 +132,21 @@ contract BranchBridgeAgent is IBranchBridgeAgent, BridgeAgentConstants {

             localChainId = _localChainId;
             rootChainId = _rootChainId;
    -        rootBridgeAgentAddress = _rootBridgeAgentAddress;
    -        lzEndpointAddress = _lzEndpointAddress;
    -        localRouterAddress = _localRouterAddress;
    -        localPortAddress = _localPortAddress;
    -        bridgeAgentExecutorAddress = DeployBranchBridgeAgentExecutor.deploy();
    +        assembly{
    +            sstore(rootBridgeAgentAddress.slot,_rootBridgeAgentAddress)
    +        }
    +        assembly{
    +            sstore(lzEndpointAddress.slot,_lzEndpointAddress)
    +        }
    +        assembly{
    +            sstore(localRouterAddress.slot,_localRouterAddress)
    +        }
    +        assembly{
    +            sstore(localPortAddress.slot,_localPortAddress)
    +        }
    +        assembly{
    +            sstore(bridgeAgentExecutorAddress.slot,DeployBranchBridgeAgentExecutor.deploy())
    +        }
             depositNonce = 1;

             rootBridgeAgentPath = abi.encodePacked(_rootBridgeAgentAddress, address(this));

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol

    File: src/BaseBranchRouter.sol	

    62: localBridgeAgentAddress = _localBridgeAgentAddress;

    63: localPortAddress = IBridgeAgent(_localBridgeAgentAddress).localPortAddress();

    64: bridgeAgentExecutorAddress = IBridgeAgent(_localBridgeAgentAddress).bridgeAgentExecutorAddress();

    diff --git a/src/BaseBranchRouter.sol b/src/BaseBranchRouter.sol
    index 62bc287..74840d2 100644
    --- a/src/BaseBranchRouter.sol
    +++ b/src/BaseBranchRouter.sol
    @@ -59,9 +59,15 @@ contract BaseBranchRouter is IBranchRouter, Ownable {
          */
         function initialize(address _localBridgeAgentAddress) external onlyOwner {
             require(_localBridgeAgentAddress != address(0), "Bridge Agent address cannot be 0");
    -        localBridgeAgentAddress = _localBridgeAgentAddress;
    -        localPortAddress = IBridgeAgent(_localBridgeAgentAddress).localPortAddress();
    -        bridgeAgentExecutorAddress = IBridgeAgent(_localBridgeAgentAddress).bridgeAgentExecutorAddress();
    +        assembly{
    +            sstore(localBridgeAgentAddress.slot,_localBridgeAgentAddress)
    +        }
    +        assembly{
    +            sstore(localPortAddress.slot,IBridgeAgent(_localBridgeAgentAddress).localPortAddress())
    +        }
    +        assembly{
    +            sstore(bridgeAgentExecutorAddress.slot,IBridgeAgent(_localBridgeAgentAddress).bridgeAgentExecutorAddress())
    +        }

             renounceOwnership();
         }

https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol

    File: src/RootPort.sol	

    138: coreRootRouterAddress = _coreRootRouter;

    159: coreRootBridgeAgentAddress = _coreRootBridgeAgent;

    160: localBranchPortAddress = _localBranchPortAddress;

    513: coreRootRouterAddress = _coreRootRouter;

    514: coreRootBridgeAgentAddress = _coreRootBridgeAgent;

    diff --git a/src/RootPort.sol b/src/RootPort.sol
    index c379b6e..7e65cb1 100644
    --- a/src/RootPort.sol
    +++ b/src/RootPort.sol
    @@ -135,7 +135,9 @@ contract RootPort is Ownable, IRootPort {
             isBridgeAgentFactory[_bridgeAgentFactory] = true;
             bridgeAgentFactories.push(_bridgeAgentFactory);

    -        coreRootRouterAddress = _coreRootRouter;
    +        assembly{
    +            sstore(coreRootRouterAddress.slot,_coreRootRouter)
    +        }
         }

         /**
    @@ -156,8 +158,12 @@ contract RootPort is Ownable, IRootPort {
             require(_setupCore, "Core Setup ended.");
             _setupCore = false;

    -        coreRootBridgeAgentAddress = _coreRootBridgeAgent;
    -        localBranchPortAddress = _localBranchPortAddress;
    +        assembly{
    +            sstore(coreRootBridgeAgentAddress.slot,_coreRootBridgeAgent)
    +        }
    +        assembly{
    +            sstore(localBranchPortAddress.slot,_localBranchPortAddress)
    +        }
             IBridgeAgent(_coreRootBridgeAgent).syncBranchBridgeAgent(_coreLocalBranchBridgeAgent, localChainId);
             getBridgeAgentManager[_coreRootBridgeAgent] = owner();
         }
    @@ -510,8 +516,12 @@ contract RootPort is Ownable, IRootPort {
             if (_coreRootRouter == address(0)) revert InvalidCoreRootRouter();
             if (_coreRootBridgeAgent == address(0)) revert InvalidCoreRootBridgeAgent();

    -        coreRootRouterAddress = _coreRootRouter;
    -        coreRootBridgeAgentAddress = _coreRootBridgeAgent;
    +        assembly{
    +            sstore(coreRootRouterAddress.slot,_coreRootRouter)
    +        }
    +        assembly{
    +            sstore(coreRootBridgeAgentAddress.slot,_coreRootBridgeAgent)
    +        }
             getBridgeAgentManager[_coreRootBridgeAgent] = owner();

             emit CoreRootSet(_coreRootRouter, _coreRootBridgeAgent);

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol

    File: src/token/ERC20hTokenRoot.sol	

    43: factoryAddress = _factoryAddress;

    diff --git a/src/token/ERC20hTokenRoot.sol b/src/token/ERC20hTokenRoot.sol
    index 5125639..cdcadf3 100644
    --- a/src/token/ERC20hTokenRoot.sol
    +++ b/src/token/ERC20hTokenRoot.sol
    @@ -40,7 +40,9 @@ contract ERC20hTokenRoot is ERC20, Ownable, IERC20hTokenRoot {
             require(_factoryAddress != address(0), "Factory Address cannot be 0");

             localChainId = _localChainId;
    -        factoryAddress = _factoryAddress;
    +        assembly{
    +            sstore(factoryAddress.slot,_factoryAddress)
    +        }
             _initializeOwner(_rootPortAddress);
     }

https://github.com/code-423n4/2023-09-maia/blob/main/src/token/ERC20hTokenRoot.sol

    File: src/factories/BranchBridgeAgentFactory.sol	

    72: rootBridgeAgentFactoryAddress = _rootBridgeAgentFactoryAddress;

    73: lzEndpointAddress = _lzEndpointAddress;

    74: localCoreBranchRouterAddress = _localCoreBranchRouterAddress;

    75: localPortAddress = _localPortAddress;

    diff --git a/src/factories/BranchBridgeAgentFactory.sol b/src/factories/BranchBridgeAgentFactory.sol
    index c789a27..f6a34e2 100644
    --- a/src/factories/BranchBridgeAgentFactory.sol
    +++ b/src/factories/BranchBridgeAgentFactory.sol
    @@ -69,10 +69,18 @@ contract BranchBridgeAgentFactory is Ownable, IBranchBridgeAgentFactory {

             localChainId = _localChainId;
             rootChainId = _rootChainId;
    -        rootBridgeAgentFactoryAddress = _rootBridgeAgentFactoryAddress;
    -        lzEndpointAddress = _lzEndpointAddress;
    -        localCoreBranchRouterAddress = _localCoreBranchRouterAddress;
    -        localPortAddress = _localPortAddress;
    +        assembly{
    +            sstore(rootBridgeAgentFactoryAddress.slot,_rootBridgeAgentFactoryAddress)
    +        }
    +        assembly{
    +            sstore(lzEndpointAddress.slot,_lzEndpointAddress)
    +        }
    +        assembly{
    +            sstore(localCoreBranchRouterAddress.slot,_localCoreBranchRouterAddress)
    +        }
    +        assembly{
    +            sstore(localPortAddress.slot,_localPortAddress)
    +        }
             _initializeOwner(_owner);
         }

https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/BranchBridgeAgentFactory.sol

    File: src/factories/RootBridgeAgentFactory.sol	

    35: lzEndpointAddress = _lzEndpointAddress;

    36: rootPortAddress = _rootPortAddress;

    diff --git a/src/factories/RootBridgeAgentFactory.sol b/src/factories/RootBridgeAgentFactory.sol
    index 03a81fa..4549953 100644
    --- a/src/factories/RootBridgeAgentFactory.sol
    +++ b/src/factories/RootBridgeAgentFactory.sol
    @@ -32,8 +32,12 @@ contract RootBridgeAgentFactory is IRootBridgeAgentFactory {
             require(_rootPortAddress != address(0), "Root Port Address cannot be 0");

             rootChainId = _rootChainId;
    -        lzEndpointAddress = _lzEndpointAddress;
    -        rootPortAddress = _rootPortAddress;
    +        assembly{
    +            sstore(lzEndpointAddress.slot,_lzEndpointAddress)
    +        }
    +        assembly{
    +            sstore(rootPortAddress.slot,_rootPortAddress)
    +        }
         }

         /*///////////////////////////////////////////////////////////////

https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/RootBridgeAgentFactory.sol

    File: src/factories/ERC20hTokenBranchFactory.sol	

    47: localPortAddress = _localPortAddress;

    74: localCoreRouterAddress = _coreRouter;

    diff --git a/src/factories/ERC20hTokenBranchFactory.sol b/src/factories/ERC20hTokenBranchFactory.sol
    index abcccb7..bd55e1a 100644
    --- a/src/factories/ERC20hTokenBranchFactory.sol
    +++ b/src/factories/ERC20hTokenBranchFactory.sol
    @@ -44,7 +44,9 @@ contract ERC20hTokenBranchFactory is Ownable, IERC20hTokenBranchFactory {
             chainName = string.concat(_chainName, " Ulysses");
             chainSymbol = string.concat(_chainSymbol, "-u");
             localChainId = _localChainId;
    -        localPortAddress = _localPortAddress;
    +        assembly{
    +            sstore(localPortAddress.slot,_localPortAddress)
    +        }
             _initializeOwner(msg.sender);
         }

    @@ -71,7 +73,9 @@ contract ERC20hTokenBranchFactory is Ownable, IERC20hTokenBranchFactory {

             hTokens.push(newToken);

    -        localCoreRouterAddress = _coreRouter;
    +        assembly{
    +            sstore(localCoreRouterAddress.slot,_coreRouter)
    +        }

             renounceOwnership();
         }

https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenBranchFactory.sol

    File: src/factories/ERC20hTokenRootFactory.sol	

    37: rootPortAddress = _rootPortAddress;

    51: coreRootRouterAddress = _coreRouter;

    diff --git a/src/factories/ERC20hTokenRootFactory.sol b/src/factories/ERC20hTokenRootFactory.sol
    index b3c6c50..a0b8c2b 100644
    --- a/src/factories/ERC20hTokenRootFactory.sol
    +++ b/src/factories/ERC20hTokenRootFactory.sol
    @@ -34,7 +34,9 @@ contract ERC20hTokenRootFactory is Ownable, IERC20hTokenRootFactory {
         constructor(uint16 _localChainId, address _rootPortAddress) {
             require(_rootPortAddress != address(0), "Root Port Address cannot be 0");
             localChainId = _localChainId;
    -        rootPortAddress = _rootPortAddress;
    +        assembly{
    +            sstore(rootPortAddress.slot,_rootPortAddress)
    +        }
             _initializeOwner(msg.sender);
         }

    @@ -48,7 +50,9 @@ contract ERC20hTokenRootFactory is Ownable, IERC20hTokenRootFactory {
          */
         function initialize(address _coreRouter) external onlyOwner {
             require(_coreRouter != address(0), "CoreRouter address cannot be 0");
    -        coreRootRouterAddress = _coreRouter;
    +        assembly{
    +            sstore(coreRootRouterAddress.slot,_coreRouter)
    +        }
             renounceOwnership();
         }

https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenRootFactory.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
    
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
    
        function testGas() public {
            c0.setOwnerAssembly(0xFD2dabe9DFcc4d88a12A9D0D40D834E81217Cccf);
            c1.setOwner(0xFD2dabe9DFcc4d88a12A9D0D40D834E81217Cccf);
        }
    }
    
    contract Contract0 {
    
        address owner;
        function setOwnerAssembly(address _owner) public {
            assembly{
                sstore(owner.slot,_owner)
            }
        }
    
    }
    
    contract Contract1 {
        address owner;
        function setOwner(address _owner) public {
            owner = _owner;
        }
    
    }

#### Gas Report

|  Contract0 contract                       |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 35287                                     | 207             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| setOwnerAssembly                          | 22324           | 22324 | 22324  | 22324 | 1       |


|  Contract1 contract                       |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 48499                                     | 273             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| setOwner                                  | 22363           | 22363 | 22363  | 22363 | 1       |

## [G-03] Stack variable used as a cheaper cache for a state variable is only used once

If the variable is only accessed once, it’s cheaper to use the state variable directly that one time, and save the 3 gas the extra stack assignment would spend.

    File: src/ArbitrumCoreBranchRouter.sol	
    
    62:  bytes memory payload = abi.encodePacked(bytes1(0x02), params);

    115: bytes memory payload = abi.encodePacked(bytes1(0x04), data);

https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumCoreBranchRouter.sol

    File: src/MulticallRootRouter.sol	

    325: Call[] memory calls = abi.decode(_decode(encodedData[1:]), (Call[]));

    413: Call[] memory calls = abi.decode(_decode(encodedData[1:]), (Call[]));

https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol

    File: src/CoreRootRouter.sol	

    136: bytes memory payload = abi.encodePacked(bytes1(0x02), params);

    171: bytes memory payload = abi.encodePacked(bytes1(0x03), params);

    196: bytes memory payload = abi.encodePacked(bytes1(0x04), params);

    223: bytes memory payload = abi.encodePacked(bytes1(0x05), params);

    254: bytes memory payload = abi.encodePacked(bytes1(0x06), params);

    284: bytes memory payload = abi.encodePacked(bytes1(0x07), params);

    434: bytes memory payload = abi.encodePacked(bytes1(0x01), params);

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol

    File: src/BranchBridgeAgentExecutor.sol	 

    131: uint256 length = _payload.length;{

    274: uint32 nonce = uint32(bytes4(_dParams[PARAMS_START:5]));

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgentExecutor.sol

    File: src/CoreBranchRouter.sol	

    51: bytes memory payload = abi.encodePacked(bytes1(0x01), params);

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol

    File: src/RootBridgeAgent.sol	

    292: bytes memory payload = abi.encodePacked(bytes1(0x03), settlementOwner, _settlementNonce);

    1004: uint32 cachedSettlementNonce = _settlementNonce;

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol

    File: src/BranchBridgeAgent.sol	

    188: bytes memory payload = abi.encodePacked(bytes1(0x00), depositNonce++, _params);
    
    202: bytes memory payload = abi.encodePacked(bytes1(0x01), depositNonce++, _params);

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol

    File: src/RootPort.sol	

    195: address globalAddress = getGlobalTokenFromLocal[_localAddress][_srcChainId];

    206: address localAddress = getLocalTokenFromGlobal[_globalAddress][_srcChainId];

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized();
            c1.optimized();
        }
    }

    contract Contract0 {
        uint256 num = 5;
        uint256 sum;
        function not_optimized() public returns(bool){
            uint256 num1 = num;
            sum = num1 + 2;
        }
    }

    contract Contract1 {
        uint256 num = 5;
        uint256 sum;
        function optimized() public returns(bool){
            sum = num + 2;
        }
    }

#### Gas Report

| Contract0 contract                        |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 63799                                     | 244             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| not_optimized                             | 24462           | 24462 | 24462  | 24462 | 1       |


| Contract1 contract                        |                 |       |        |       |         |
|-------------------------------------------|-----------------|-------|--------|-------|---------|
| Deployment Cost                           | Deployment Size |       |        |       |         |
| 63599                                     | 243             |       |        |       |         |
| Function Name                             | min             | avg   | median | max   | # calls |
| optimized                                 | 24460           | 24460 | 24460  | 24460 | 1       |

## [G-04] *<x> -= <y>* costs more gas than *<x> = <x> - <y>* for state variables (+= too)

Using the Addition operator instead of plus-equals is gas efficient. Subtraction act the same way.
    
    File: src/VirtualAccount.sol	

    96: valAccumulator += val;

https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol

    File: src/BranchPort.sol	

    157: getPortStrategyTokenDebt[msg.sender][_token] += _amount;

    169: getPortStrategyTokenDebt[msg.sender][_token] -= _amount;
    
    172: getStrategyTokenDebt[_token] -= _amount;

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol

    File: src/token/ERC20hTokenRoot.sol	

    58: getTokenBalance[chainId] += amount;

    70: getTokenBalance[chainId] -= amount;

https://github.com/code-423n4/2023-09-maia/blob/main/src/token/ERC20hTokenRoot.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;

        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }

        function testGas() public {
            c0.add();
            c1.add1();
        }
    }

    contract Contract0 {

        uint8 num1 = 1;

        function add() public{
            num1 += 1;
        }

    }

    contract Contract1 {

        uint8 num1 = 1;

        function add1() public{
            num1 = num1 + 1;
        }

    }

#### Gas Report

| Contract0 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 67017                                     | 268             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| add                                       | 5405            | 5405 | 5405   | 5405 | 1       |

| Contract1 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 70623                                     | 286             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| add1                                      | 5363            | 5363 | 5363   | 5363 | 1       |

## [G-05] Instead of counting down in *for* statements, count up

Counting down is more gas efficient than counting up because neither we are making zero variable to non-zero variable and also we will get gas refund in the last transaction when making non-zero to zero variable.

    File: src/MulticallRootRouter.sol	

    278: for (uint256 i = 0; i < outputParams.outputTokens.length;) {     
    281:     unchecked {
    282:         ++i;
    283:     }

    367: for (uint256 i = 0; i < outputParams.outputTokens.length;) {
    370:     unchecked {
    371:         ++i;
    372:     }

    455: for (uint256 i = 0; i < outputParams.outputTokens.length;) {
    458:     unchecked {
    459:         ++i;
    460:     }

    557: for (uint256 i = 0; i < outputTokens.length;) {

https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol

    File: src/RootBridgeAgentExecutor.sol	

    281: for (uint256 i = 0; i < uint256(uint8(numOfAssets));) {
    332:     unchecked {
    333:         ++i;
    334:     }

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol

    File: src/RootBridgeAgent.sol	

    318: for (uint256 i = 0; i < settlement.hTokens.length;) {
    337:     unchecked {
    338:         ++i;
    339:     }

    399: for (uint256 i = 0; i < length;) {
    412:     unchecked {
    413:         ++i;
    414:     }

    1070: for (uint256 i = 0; i < hTokens.length;) {
    1083:     unchecked {
    1083:         ++i;
    1083:     }

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol

    File: src/VirtualAccount.sol	

    70: for (uint256 i = 0; i < length;) {
    78:     unchecked {
    79:         ++i;
    80:     }

    90: for (uint256 i = 0; i < length;) {
    105:     unchecked {
    106:         ++i;
    107:     }

https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol

    File: src/BranchPort.sol	

    257: for (uint256 i = 0; i < length;) {
    270:     unchecked {
    271:         ++i;
    272:     }

    305: for (uint256 i = 0; i < length;) {
    308:     unchecked {
    309:         i++;
    310:     }

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol

    File: src/BranchBridgeAgent.sol	

    447: for (uint256 i = 0; i < deposit.tokens.length;) {
    450:     unchecked {
    451:         ++i;
    452:     }
    
    513: for (uint256 i = 0; i < numOfAssets;) {
    563:    unchecked {
    564:        ++i;
    565:    }

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol

    File: src/BaseBranchRouter.sol	

    192: for (uint256 i = 0; i < _hTokens.length;) {
    195:     unchecked {
    196:         ++i;
    197:     }

https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol

#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.AddNum();
            c1.AddNum();
        }
    }


    contract Contract0 {
        uint256 num = 3;
        function AddNum() public {
            uint256 _num = num;
            for(uint i=0;i<=9;i++){
                _num = _num +1;
            }
            num = _num;
        }
    }


    contract Contract1 {
        uint256 num = 3;
        function AddNum() public {
            uint256 _num = num;
            for(uint i=9;i>=0;i--){
                _num = _num +1;
            }
            num = _num;
        }
    }

#### Gas Report

| Contract0 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 77011                                     | 311             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| AddNum                                    | 7040            | 7040 | 7040   | 7040 | 1       |

| Contract1 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 73811                                     | 295             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| AddNum                                    | 3819            | 3819 | 3819   | 3819 | 1       |

