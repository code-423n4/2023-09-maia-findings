# Gas Optimization

# Summary

| Number | Gas Optimization                                                                      | Context |
| :----: | :------------------------------------------------------------------------------------ | :-----: |
| [G-01] | Shorten the array rather than copying to a new one                                    |   15    |
| [G-02] | It is sometimes cheaper to cache calldata                                             |    7    |
| [G-03] | Split revert statements                                                               |    1    |
| [G-04] | NOT USING THE NAMED RETURN VARIABLES WHEN A FUNCTION RETURNS, WASTES DEPLOYMENT GAS   |   15    |
| [G-05] | Do-While loops are cheaper than for loops                                             |   15    |
| [G-06] | Empty blocks should be removed or emit something                                      |    4    |
| [G-07] | Use ++i instead of i++ to increment                                                   |    4    |
| [G-08] | Use assembly to perform efficient back-to-back calls                                  |   39    |
| [G-09] | Sort Solidity operations using short-circuit mode                                     |    4    |
| [G-10] | Missing Zero address checks in the constructor                                        |    3    |
| [G-11] | Use hardcode address instead address(this)                                            |   32    |
| [G-12] | Use assembly to validate msg.sender                                                   |   30    |
| [G-13] | Avoid contract calls by making the architecture monolithic                            |    8    |
| [G-14] | use internal functions instead of modifiers                                           |   28    |
| [G-15] | Using > 0 costs more gas than != 0                                                    |   17    |
| [G-16] | USE BITMAPS TO SAVE GAS                                                               |    9    |
| [G-17] | Use bytes.concat() instead of abi.encodePacked(), since this is preferred since 0.8.4 |   27    |
| [G-18] | Pre-increment and pre-decrement are cheaper than +1 ,-1                               |    4    |

## [G-01] Shorten the array rather than copying to a new one

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

## [G-02] It is sometimes cheaper to cache calldata

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

## [G-03] Split revert statements

Similar to splitting require statements, you will usually save some gas by not having a boolean operator in the if statement.

```solidity
File: src/BranchPort.sol

363        if (_minimumReservesRatio >= DIVISIONER || _minimumReservesRatio < MIN_RESERVE_RATIO) {
364            revert InvalidMinimumReservesRatio();
365        }
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L363C1-L365C10

## [G-04] NOT USING THE NAMED RETURN VARIABLES WHEN A FUNCTION RETURNS, WASTES DEPLOYMENT GAS

The solidity compiler outputs more efficient code when the variable is declared in the return statement. There seem to be very few exceptions to this in practice, so if you see an anonymous return, you should test it with a named return instead to determine which case is most efficient.

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

## [G-05] Do-While loops are cheaper than for loops

If you want to push optimization at the expense of creating slightly unconventional code, Solidity do-while loops are more gas efficient than for loops, even if you add an if-condition check for the case where the loop doesn’t execute at all.

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

## [G-06] Empty blocks should be removed or emit something

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting.
If the contract is meant to be extended, the contract should be abstract and the function signatures be added without any default implementation.

If the block is an empty if-statement block to avoid doing subsequent checks in the else-if/else conditions, the else-if/else conditions should be
nested under the negation of the if-statement, because they involve different classes of checks, which may lead to the introduction of errors when
the code is later modified (if( x ) {}else if(y){ . . . }else{ . . . } => if ( !x) {if(y) { . . . }else{ . . .}}) .
Empty receive()/fallback() payable functions that are not used, can be removed to save deployment gas.

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

## [G-07] Use ++i instead of i++ to increment

The reason behind this is in way ++i and i++ are evaluated by the compiler.

i++ returns i(its old value) before incrementing i to a new value. This means that 2 values are stored on the stack for usage whether you wish to use it or not. ++i on the other hand, evaluates the ++ operation on i (i.e it increments i) then returns i (its incremented value) which means that only one item needs to be stored on the stack.

```solidity
File: src/RootBridgeAgent.sol

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

## [G-08] Use assembly to perform efficient back-to-back calls

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
File: src/MulticallRootRouter.sol

239            IVirtualAccount(userAccount).call(calls);
248            IVirtualAccount(userAccount).call(calls);
251            IVirtualAccount(userAccount).withdrawERC20(outputParams.outputToken, outputParams.amountOut);
255                IVirtualAccount(userAccount).userAddress(),
275            IVirtualAccount(userAccount).call(calls);

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L239

## [G-09] Sort Solidity operations using short-circuit mode

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

## [G-10] Missing Zero address checks in the constructor

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

## [G-11] Use hardcode address instead address(this)

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

## [G-12] Use assembly to validate msg.sender

Using assembly to validate msg.sender means that you can manually inspect and manipulate the value of msg.sender using inline assembly code to enforce custom validation rules or conditions.

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

## [G-13] Avoid contract calls by making the architecture monolithic

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

## [G-14] use internal functions instead of modifiers

```solidity
File: src/MulticallRootRouter.sol


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

## [G-15] Using > 0 costs more gas than != 0

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

## [G-16] USE BITMAPS TO SAVE GAS

```solidity
File: src/RootPort.sol

54    mapping(VirtualAccount acount => mapping(address router => bool allowed)) public isRouterApproved;

61    mapping(uint256 chainId => bool isActive) public isChainId;

64    mapping(address bridgeAgent => bool isActive) public isBridgeAgent;

77    mapping(address bridgeAgentFactory => bool isActive) public isBridgeAgentFactory;

87    mapping(address token => bool isGlobalToken) public isGlobalAddress;

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L54

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


```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L32

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

## [G-18] Pre-increment and pre-decrement are cheaper than +1 ,-1

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