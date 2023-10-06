# gas optimization

Note: [G-03],[G-04],[G-05] issue missed from bots

# Summary 

|      | issue | insteance |
|------|-------|-----------|
|[G-01]|Refactor external/internal function to avoid unnecessary External Calls|68|
|[G-02]|Can Make The Variable Outside The Loop To Save Gas|5|
|[G-03]|Avoid contract existence checks by using low-level calls|130|
|[G-04]|Avoid updating storage when the value hasn't changed|5|
|[G-05]|Using calldata instead of memory for read-only arguments in external functions saves gas|16|
|[G-06]|Access mappings directly rather than using accessor functions|2|
|[G-07]|A modifier used only once and not being inherited should be inlined to save gas|6|
|[G-08]|Cache external calls outside of loop to avoid re-calling function on each iteration|7|
|[G-09]|Use assembly in place of abi.decode to extract calldata values more efficiently|29|
|[G-10]|Use assembly to perform efficient back-to-back calls|12|
|[G-11]|Use do while loops instead of for loops|15|
|[G-12]|OR in if-condition can be rewritten to two single if conditions|1|
|[G-13]|Replace state variable reads and writes within loops with local variable reads and writes.|2|
|[G-14]|>=/<= costs less gas than >/<|17|
|[G-15]|Use Assembly To Check For address(0)|35|
|[G-16]|Use ERC721A instead ERC721|1|
|[G-17]|Using Fixed Bytes Is Cheaper Than Using String|2|
|[G-18]|Empty blocks should be removed or emit something|1|
|[G-19]|Use Modifiers Instead of Functions To Save Gas|5|
|[G-20]|State variables can be packed to use fewer storage slots|1|
|[G-21]|abi.encode() is less efficient than abi.encodepacked()|4|
|[G-22]|When possible, use assembly instead of unchecked{++i}|15|



## [G-01] Refactor external/internal function to avoid unnecessary External Calls
The functions below perform external calls that are previously performed in the functions that invoke them. We can refactor the external/internal functions so we could pass the cached external calls into them and avoid the extra external calls that would otherwise take place in the internal functions.

There are 68 instances of this issue:
```solidity
File:  factories/ERC20hTokenBranchFactory.sol
66          ERC20(_wrappedNativeTokenAddress).name(),
            ERC20(_wrappedNativeTokenAddress).symbol(),
            ERC20(_wrappedNativeTokenAddress).decimals(),
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenBranchFactory.sol#L66-L69

the wrappedToken variable is introduced to cache the result of the ERC20(_wrappedNativeTokenAddress) external call.
`Fix code`
```diff
    function initialize(address _wrappedNativeTokenAddress, address _coreRouter) external onlyOwner {
        require(_coreRouter != address(0), "CoreRouter address cannot be 0");
+       ERC20 wrappedToken = ERC20(_wrappedNativeTokenAddress); // Cache the external call result
        ERC20hTokenBranch newToken = new ERC20hTokenBranch(
            chainName,
            chainSymbol,
-           ERC20(_wrappedNativeTokenAddress).name(),
+           wrappedToken.name(),
-           ERC20(_wrappedNativeTokenAddress).symbol(),
+           wrappedToken.symbol(),
-           ERC20(_wrappedNativeTokenAddress).decimals(),
+           wrappedToken.decimals(),
            localPortAddress
        );

        hTokens.push(newToken);

        localCoreRouterAddress = _coreRouter;

        renounceOwnership();
    }
```

```solidity
File:  ArbitrumBranchPort.sol
83   if (!IRootPort(_rootPortAddress).isGlobalToken(_globalAddress, localChainId)) revert UnknownGlobalToken();

87   IRootPort(_rootPortAddress).getUnderlyingTokenFromLocal(_globalAddress, localChainId);

92   IRootPort(_rootPortAddress).burnFromLocalBranch(_depositor, _globalAddress, _amount);
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol#L83

the IRootPort instance is cached in the rootPort variable after the external call to IRootPort(_rootPortAddress).
`fix code`
```diff
    function withdrawFromPort(address _depositor, address _recipient, address _globalAddress, uint256 _amount)
        external
        override
        lock
        requiresBridgeAgent
    {
        // Save root port address to memory
        address _rootPortAddress = rootPortAddress;

        // Cache the IRootPort instance
+       IRootPort rootPort = IRootPort(_rootPortAddress);

        // Check if the global token exists
-       if (!IRootPort(_rootPortAddress).isGlobalToken(_globalAddress, localChainId)) revert UnknownGlobalToken();
+       if (!rootPort.isGlobalToken(_globalAddress, localChainId)) revert UnknownGlobalToken();

        // Get the underlying token address from the root port
-       address _underlyingAddress =
            IRootPort(_rootPortAddress).getUnderlyingTokenFromLocal(_globalAddress, localChainId);
+       address _underlyingAddress = rootPort.getUnderlyingTokenFromLocal(_globalAddress, localChainId);    

        // Check if the underlying token exists
        if (_underlyingAddress == address(0)) revert UnknownUnderlyingToken();

-       IRootPort(_rootPortAddress).burnFromLocalBranch(_depositor, _globalAddress, _amount);
+       rootPort.burnFromLocalBranch(_depositor, _globalAddress, _amount);

        _underlyingAddress.safeTransfer(_recipient, _amount);
    }
```

```solidity
File:  CoreBranchRouter.sol
64   uint8 decimals = ERC20(_underlyingAddress).decimals();

68   ERC20(_underlyingAddress).name(), ERC20(_underlyingAddress).symbol(), decimals, true
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L64


the ERC20 instance is cached in the underlyingToken variable after the external call to ERC20(_underlyingAddress). 
`Fix code`
```diff
    function addLocalToken(address _underlyingAddress, GasParams calldata _gParams) external payable virtual {
+       // Cache the ERC20 instance
+       ERC20 underlyingToken = ERC20(_underlyingAddress);

        //Get Token Info
        uint8 decimals = ERC20(_underlyingAddress).decimals();

        //Create Token
-       ERC20hToken newToken = ITokenFactory(hTokenFactoryAddress).createToken(
-           ERC20(_underlyingAddress).name(), ERC20(_underlyingAddress).symbol(), decimals, true
-       );
+       ERC20hToken newToken = ITokenFactory(hTokenFactoryAddress).createToken(
+       underlyingToken.name(), underlyingToken.symbol(), decimals, true
+       );

        //Encode Data
        bytes memory params = abi.encode(_underlyingAddress, newToken, newToken.name(), newToken.symbol(), decimals);

        // Pack FuncId
        bytes memory payload = abi.encodePacked(bytes1(0x02), params);

        //Send Cross-Chain request (System Response/Request)
        IBridgeAgent(localBridgeAgentAddress).callOutSystem{value: msg.value}(payable(msg.sender), payload, _gParams);
    }
```

```solidity
File:  CoreBranchRouter.sol
249   if (IPort(_localPortAddress).isBridgeAgentFactory(_newBridgeAgentFactoryAddress)) {

251   IPort(_localPortAddress).toggleBridgeAgentFactory(_newBridgeAgentFactoryAddress);

254   IPort(_localPortAddress).addBridgeAgentFactory(_newBridgeAgentFactoryAddress);
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L249
the IPort instance is cached in the port variable after the external call to IPort(localPortAddress).
`Fix code`
```diff
    function _toggleBranchBridgeAgentFactory(address _newBridgeAgentFactoryAddress) internal {
-       // Save Port Address to memory
-       address _localPortAddress = localPortAddress;
+       // Cache the IPort instance
+       IPort port = IPort(localPortAddress);

        // Check if BridgeAgentFactory is active
-       if (IPort(_localPortAddress).isBridgeAgentFactory(_newBridgeAgentFactoryAddress)) {
+       if (port.isBridgeAgentFactory(_newBridgeAgentFactoryAddress)) {
            // If so, disable it.
-           IPort(_localPortAddress).toggleBridgeAgentFactory(_newBridgeAgentFactoryAddress);
+           port.toggleBridgeAgentFactory(_newBridgeAgentFactoryAddress);
        } else {
            // If not, add it.
-           IPort(_localPortAddress).addBridgeAgentFactory(_newBridgeAgentFactoryAddress);
+           port.addBridgeAgentFactory(_newBridgeAgentFactoryAddress);
        }
    }
```

```solidity
File:  CoreBranchRouter.sol
289  if (IPort(_localPortAddress).isStrategyToken(_underlyingToken)) {

291  IPort(_localPortAddress).toggleStrategyToken(_underlyingToken);

294  IPort(_localPortAddress).addStrategyToken(_underlyingToken, _minimumReservesRatio);
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L289

the IPort instance is cached in the port variable after the external call to IPort(localPortAddress).
`Fix code`
```diff
    function _manageStrategyToken(address _underlyingToken, uint256 _minimumReservesRatio) internal {
-       // Save Port Address to memory
-       address _localPortAddress = localPortAddress;
+       // Cache the IPort instance
+       IPort port = IPort(localPortAddress);

        // Check if the token is an active Strategy Token
-       if (IPort(_localPortAddress).isStrategyToken(_underlyingToken)) {
+       if (port.isStrategyToken(_underlyingToken)) {
           // If so, toggle it off.
-           IPort(_localPortAddress).toggleStrategyToken(_underlyingToken);
+           port.toggleStrategyToken(_underlyingToken);
        } else {
            // If not, add it.
-           IPort(_localPortAddress).addStrategyToken(_underlyingToken, _minimumReservesRatio);
+           port.addStrategyToken(_underlyingToken, _minimumReservesRatio);
        }
    }

```

```solidity
File:  CoreBranchRouter.sol
317  if (!IPort(_localPortAddress).isPortStrategy(_portStrategy, _underlyingToken)) {

319  IPort(_localPortAddress).addPortStrategy(_portStrategy, _underlyingToken, _dailyManagementLimit);

322  IPort(_localPortAddress).updatePortStrategy(_portStrategy, _underlyingToken, _dailyManagementLimit);

325  IPort(_localPortAddress).togglePortStrategy(_portStrategy, _underlyingToken);
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L317

the IPort instance is cached in the port variable after the external call to IPort(localPortAddress).
`Fix code`
```diff
    function _managePortStrategy(
        address _portStrategy,
        address _underlyingToken,
        uint256 _dailyManagementLimit,
        bool _isUpdateDailyLimit
    ) internal {
-       // Save Port Address to memory
-       address _localPortAddress = localPortAddress;
+       // Cache the IPort instance
+       IPort port = IPort(localPortAddress)

        // Check if Port Strategy is active
-       if (!IPort(_localPortAddress).isPortStrategy(_portStrategy, _underlyingToken)) {
+       if (!port.isPortStrategy(_portStrategy, _underlyingToken)) {   
            // If Port Strategy is not active, add new Port Strategy.
-           IPort(_localPortAddress).addPortStrategy(_portStrategy, _underlyingToken, _dailyManagementLimit);
+           port.addPortStrategy(_portStrategy, _underlyingToken, _dailyManagementLimit);
        } else if (_isUpdateDailyLimit) {
            // Or update daily limit.
-           IPort(_localPortAddress).updatePortStrategy(_portStrategy, _underlyingToken, _dailyManagementLimit);
+           port.updatePortStrategy(_portStrategy, _underlyingToken, _dailyManagementLimit);
        } else {
            // Or Toggle Port Strategy.
-           IPort(_localPortAddress).togglePortStrategy(_portStrategy, _underlyingToken);
+           port.togglePortStrategy(_portStrategy, _underlyingToken);
        }
    }
```

```solidity
File:  CoreRootRouter.sol
// @audit in addBranchToBridgeAgent function cache IBridgeAgent(_rootBridgeAgent)
120  if (IBridgeAgent(_rootBridgeAgent).getBranchBridgeAgent(_dstChainId) != address(0)) revert InvalidChainId();

123  if (!IBridgeAgent(_rootBridgeAgent).isBranchBridgeAgentAllowed(_dstChainId)) revert UnauthorizedChainId();

130  IBridgeAgent(_rootBridgeAgent).factoryAddress(),

// @audit in _addGlobalToken function cache ERC20(_globalAddress)
426         ERC20(_globalAddress).name(),
427         ERC20(_globalAddress).symbol(),
428         ERC20(_globalAddress).decimals(),

// @audit in _addLocalToken function cache IPort(rootPortAddress)
461        if (IPort(rootPortAddress).isGlobalAddress(_underlyingAddress)) revert TokenAlreadyAdded();
462        if (IPort(rootPortAddress).isLocalToken(_underlyingAddress, _srcChainId)) revert TokenAlreadyAdded();
463        if (IPort(rootPortAddress).isUnderlyingToken(_underlyingAddress, _srcChainId)) revert TokenAlreadyAdded();
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L120

```solidity
File:  MulticallRootRouter.sol
// @audit in executeSigned function cache IVirtualAccount(userAccount)
239  IVirtualAccount(userAccount).call(calls);

248  IVirtualAccount(userAccount).call(calls);

251  IVirtualAccount(userAccount).withdrawERC20(outputParams.outputToken, outputParams.amountOut);

255  IVirtualAccount(userAccount).userAddress(),

275  IVirtualAccount(userAccount).call(calls);

279  IVirtualAccount(userAccount).withdrawERC20(outputParams.outputTokens[i], outputParams.amountsOut[i]);

288  IVirtualAccount(userAccount).userAddress(),


// @audit in executeSignedDepositSingle function cache IVirtualAccount(userAccount)
328  IVirtualAccount(userAccount).call(calls);

337  IVirtualAccount(userAccount).call(calls);

340  IVirtualAccount(userAccount).withdrawERC20(outputParams.outputToken, outputParams.amountOut);

344  IVirtualAccount(userAccount).userAddress(),

364  IVirtualAccount(userAccount).call(calls);

368  IVirtualAccount(userAccount).withdrawERC20(outputParams.outputTokens[i], outputParams.amountsOut[i]);

377  IVirtualAccount(userAccount).userAddress(),



// @audit in executeSignedDepositMultiple function cache IVirtualAccount(userAccount)
416  IVirtualAccount(userAccount).call(calls);

425  IVirtualAccount(userAccount).call(calls);

428  IVirtualAccount(userAccount).withdrawERC20(outputParams.outputToken, outputParams.amountOut);

432  IVirtualAccount(userAccount).userAddress(),

422  IVirtualAccount(userAccount).call(calls);

456  IVirtualAccount(userAccount).withdrawERC20(outputParams.outputTokens[i], outputParams.amountsOut[i]);

465  IVirtualAccount(userAccount).userAddress(),
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L239

```solidity
File:  RootBridgeAgent.sol
// @audit in redeemSettlement function cache IPort(localPortAddress)
312  if (msg.sender != address(IPort(localPortAddress).getUserAccount(settlementOwner))) {

328  IPort(localPortAddress).bridgeToRoot(

330  IPort(localPortAddress).getGlobalTokenFromLocal(_hToken, _dstChainId),        


// @audit in bridgeIn function cache IPort(_localPortAddress)
364  if (!IPort(_localPortAddress).isLocalToken(_dParams.hToken, _srcChainId)) {

371  if (IPort(_localPortAddress).getLocalTokenFromUnderlying(_dParams.token, _srcChainId) != _dParams.hToken) {

377  IPort(_localPortAddress).bridgeToRoot(

379  IPort(_localPortAddress).getGlobalTokenFromLocal(_dParams.hToken, _srcChainId), 

// @audit in lzReceiveNonBlocking function cache IPort(localPortAddress)
549   VirtualAccount userAccount = IPort(localPortAddress).fetchVirtualAccount(

574   IPort(localPortAddress).toggleVirtualAccountApproved(userAccount, localRouterAddress);

587   VirtualAccount userAccount = IPort(localPortAddress).fetchVirtualAccount(

592   IPort(localPortAddress).toggleVirtualAccountApproved(userAccount, localRouterAddress);

614   IPort(localPortAddress).toggleVirtualAccountApproved(userAccount, localRouterAddress);

627   VirtualAccount userAccount = IPort(localPortAddress).fetchVirtualAccount(

632   IPort(localPortAddress).toggleVirtualAccountApproved(userAccount, localRouterAddress);

654   IPort(localPortAddress).toggleVirtualAccountApproved(userAccount, localRouterAddress);

674   if (owner != address(IPort(localPortAddress).getUserAccount(settlementReference.owner))) {
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L312

```solidity
File:  RootPort.sol
// @audit in syncBranchBridgeAgentWithRoot function cache IBridgeAgent(_rootBridgeAgent)
398  if (IBridgeAgent(_rootBridgeAgent).getBranchBridgeAgent(_branchChainId) != address(0)) {

401  if (!IBridgeAgent(_rootBridgeAgent).isBranchBridgeAgentAllowed(_branchChainId)) {

404  IBridgeAgent(_rootBridgeAgent).syncBranchBridgeAgent(_newBranchBridgeAgent, _branchChainId);        
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L398


## [G-02] Can Make The Variable Outside The Loop To Save Gas 
When you declare a variable inside a loop, Solidity creates a new instance of the variable for each iteration of the loop. This can lead to unnecessary gas costs, especially if the loop is executed frequently or iterates over a large number of elements.

By declaring the variable outside the loop, you can avoid the creation of multiple instances of the variable and reduce the gas cost of your contract. Here's an example:

```
contract MyContract {
    function sum(uint256[] memory values) public pure returns (uint256) {
        uint256 total = 0;
        
        for (uint256 i = 0; i < values.length; i++) {
            total += values[i];
        }
        
        return total;
    }
}
```
There are 5 instances of this issue:
```solidity
File:  BranchBridgeAgent.sol
515  uint256 currentIterationOffset = PARAMS_START + i;
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L515

```solidity
File:  RootBridgeAgent.sol
320  address _hToken = settlement.hTokens[i];

1076 uint16 destChainId = _dstChainId;
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L320

```solidity
File:  RootBridgeAgentExecutor.sol
283   uint256 currentIterationOffset = PARAMS_START + i;
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L283

```solidity
File:  VirtualAccount.sol
92  uint256 val = _call.value;
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L92


## [G‑03] Avoid contract existence checks by using low-level calls
Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low-level calls never check for contract existence

There are 130 instances of this issue:

```solidity
File:  factories/ArbitrumBranchBridgeAgentFactory.sol
65   IPort(localPortAddress).addBridgeAgent(newCoreBridgeAgent);

98   IPort(localPortAddress).addBridgeAgent(newBridgeAgent);
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ArbitrumBranchBridgeAgentFactory.sol#L65

```solidity
File:  factories/BranchBridgeAgentFactory.sol
101   IPort(localPortAddress).addBridgeAgent(newCoreBridgeAgent);

139   IPort(localPortAddress).addBridgeAgent(newBridgeAgent);
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/BranchBridgeAgentFactory.sol#L101

```solidity
File:  factories/RootBridgeAgentFactory.sol
58  IRootPort(rootPortAddress).addBridgeAgent(msg.sender, newBridgeAgent);
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/RootBridgeAgentFactory.sol#L58

```solidity
File: ArbitrumBranchBridgeAgent.sol
70  IArbPort(localPortAddress).depositToPort(msg.sender, msg.sender, underlyingAddress, amount);

80  IArbPort(localPortAddress).withdrawFromPort(msg.sender, msg.sender, localAddress, amount);

105 IRootBridgeAgent(_rootBridgeAgentAddress).lzReceive(rootChainId, "", 0, _calldata);

114 IRootBridgeAgent(rootBridgeAgentAddress).lzReceive(
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchBridgeAgent.sol#L70

```solidity
File:  ArbitrumBranchPort.sol
69   IRootPort(_rootPortAddress).mintToLocalBranch(_recipient, _globalToken, _deposit);

83   if (!IRootPort(_rootPortAddress).isGlobalToken(_globalAddress, localChainId)) revert UnknownGlobalToken();

92   IRootPort(_rootPortAddress).burnFromLocalBranch(_depositor, _globalAddress, _amount);

108  IRootPort(rootPortAddress).bridgeToLocalBranchFromRoot(_recipient, _localAddress, _amount);

134  IRootPort(rootPortAddress).bridgeToRootFromLocalBranch(_depositor, _localAddress, _amount - _deposit);
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol#L69

```solidity
File:  ArbitrumCoreBranchRouter.sol
65   IBridgeAgent(localBridgeAgentAddress).callOutSystem(payable(msg.sender), payload, GasParams(0, 0));

97   if (!IPort(_localPortAddress).isBridgeAgentFactory(_branchBridgeAgentFactory)) {

107  if (!IPort(_localPortAddress).isBridgeAgent(newBridgeAgent)) {

118  IBridgeAgent(localBridgeAgentAddress).callOutSystem(payable(_refundee), payload, _gParams);
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumCoreBranchRouter.sol#L65

```solidity
File:  BaseBranchRouter.sol
75   return IBridgeAgent(localBridgeAgentAddress).getDepositEntry(_depositNonce);

84   IBridgeAgent(localBridgeAgentAddress).callOut{value: msg.value}(payable(msg.sender), _params, _gParams);

98   IBridgeAgent(localBridgeAgentAddress).callOutAndBridge{value: msg.value}(

113  IBridgeAgent(localBridgeAgentAddress).callOutAndBridgeMultiple{value: msg.value}(

168  ERC20(_hToken).approve(_localPortAddress, _amount - _deposit);

175  ERC20(_token).approve(_localPortAddress, _deposit);
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L75

```solidity
File:  BranchBridgeAgent.sol
568  IPort(localPortAddress).bridgeInMultiple(_recipient, _hTokens, _tokens, _amounts, _deposits);

770  ILayerZeroEndpoint(lzEndpointAddress).send{value: msg.value}(

787  ILayerZeroEndpoint(lzEndpointAddress).send{value: address(this).balance}(

824  IPort(localPortAddress).bridgeOut(msg.sender, _hToken, _token, _amount, _deposit);

878  IPort(localPortAddress).bridgeOutMultiple(msg.sender, _hTokens, _tokens, _amounts, _deposits);

908  IPort(localPortAddress).bridgeIn(_recipient, _hToken, _amount - _deposit);

913  IPort(localPortAddress).withdraw(_recipient, _token, _deposit);
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L568


```solidity
File:  BranchBridgeAgentExecutor.sol
55    IRouter(_router).executeNoSettlement{value: msg.value}(_payload[PARAMS_TKN_START_SIGNED:]);

82    BranchBridgeAgent(payable(msg.sender)).clearToken(

89    IRouter(_router).executeSettlement{value: msg.value}(_payload[PARAMS_SETTLEMENT_OFFSET:], sParams);

121   IRouter(_router).executeSettlementMultiple{value: msg.value}(
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgentExecutor.sol#L55

```solidity
File:  BranchPort.sol
210   IPortStrategy(_strategy).withdraw(address(this), _token, amountToWithdraw);

503   ERC20hTokenBranch(_localAddress).mint(_recipient, _amount);

527   ERC20hTokenBranch(_localAddress).burn(_hTokenAmount);
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L210

```solidity
File:  CoreBranchRouter.sol
54   IBridgeAgent(localBridgeAgentAddress).callOut{value: msg.value}(payable(msg.sender), payload, _gParams[0]);

78   IBridgeAgent(localBridgeAgentAddress).callOutSystem{value: msg.value}(payable(msg.sender), payload, _gParams);

143  IPort(localPortAddress).setCoreBranchRouter(coreBranchRouter, coreBranchBridgeAgent);

184   IBridgeAgent(localBridgeAgentAddress).callOutSystem{value: msg.value}(payable(_refundee), payload, _gParams);

211  if (!IPort(_localPortAddress).isBridgeAgentFactory(_branchBridgeAgentFactory)) {

221  if (!IPort(_localPortAddress).isBridgeAgent(newBridgeAgent)) {    

232  IBridgeAgent(localBridgeAgentAddress).callOutSystem{value: msg.value}(payable(_refundee), payload, _gParams); 

249  if (IPort(_localPortAddress).isBridgeAgentFactory(_newBridgeAgentFactoryAddress)) {

251  IPort(_localPortAddress).toggleBridgeAgentFactory(_newBridgeAgentFactoryAddress);

254  IPort(_localPortAddress).addBridgeAgentFactory(_newBridgeAgentFactoryAddress);

268  if (!IPort(_localPortAddress).isBridgeAgent(_branchBridgeAgent)) revert UnrecognizedBridgeAgent();

271  IPort(_localPortAddress).toggleBridgeAgent(_branchBridgeAgent);

289  if (IPort(_localPortAddress).isStrategyToken(_underlyingToken)) {

291  IPort(_localPortAddress).toggleStrategyToken(_underlyingToken);

294  IPort(_localPortAddress).addStrategyToken(_underlyingToken, _minimumReservesRatio);

317  if (!IPort(_localPortAddress).isPortStrategy(_portStrategy, _underlyingToken)) {

319  IPort(_localPortAddress).addPortStrategy(_portStrategy, _underlyingToken, _dailyManagementLimit);

322  IPort(_localPortAddress).updatePortStrategy(_portStrategy, _underlyingToken, _dailyManagementLimit);

325  IPort(_localPortAddress).togglePortStrategy(_portStrategy, _underlyingToken);
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L54

```solidity
File:  CoreRootRouter.sol
117  if (!IPort(rootPortAddress).isChainId(_dstChainId)) revert InvalidChainId();

123  if (!IBridgeAgent(_rootBridgeAgent).isBranchBridgeAgentAllowed(_dstChainId)) revert UnauthorizedChainId();

139  IBridgeAgent(bridgeAgentAddress).callOut{value: msg.value}(

163  if (!IPort(rootPortAddress).isBridgeAgentFactory(_rootBridgeAgentFactory)) {

174  IBridgeAgent(bridgeAgentAddress).callOut{value: msg.value}(

199  IBridgeAgent(bridgeAgentAddress).callOut{value: msg.value}(

226  IBridgeAgent(bridgeAgentAddress).callOut{value: msg.value}(

257  IBridgeAgent(bridgeAgentAddress).callOut{value: msg.value}(

287  IBridgeAgent(bridgeAgentAddress).callOut{value: msg.value}(

414  if (!IPort(rootPortAddress).isGlobalAddress(_globalAddress)) {

419  if (IPort(rootPortAddress).isGlobalToken(_globalAddress, _dstChainId)) {

437  IBridgeAgent(bridgeAgentAddress).callOut{value: msg.value}(

461  if (IPort(rootPortAddress).isGlobalAddress(_underlyingAddress)) revert TokenAlreadyAdded();

462  if (IPort(rootPortAddress).isLocalToken(_underlyingAddress, _srcChainId)) revert TokenAlreadyAdded();

463  if (IPort(rootPortAddress).isUnderlyingToken(_underlyingAddress, _srcChainId)) revert TokenAlreadyAdded();           
469  IPort(rootPortAddress).setAddresses(

483  if (IPort(rootPortAddress).isLocalToken(_localAddress, _dstChainId)) revert TokenAlreadyAdded();

486   IPort(rootPortAddress).setLocalAddress(_globalAddress, _localAddress, _dstChainId);

503  IPort(rootPortAddress).syncBranchBridgeAgentWithRoot(_newBranchBridgeAgent, _rootBridgeAgent, _srcChainId);
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L117

```solidity
File:  MulticallRootRouter.sol
239   IVirtualAccount(userAccount).call(calls);

248   IVirtualAccount(userAccount).call(calls);

251   IVirtualAccount(userAccount).withdrawERC20(outputParams.outputToken, outputParams.amountOut);

255   IVirtualAccount(userAccount).userAddress(),

275   IVirtualAccount(userAccount).call(calls);

279   IVirtualAccount(userAccount).withdrawERC20(outputParams.outputTokens[i], outputParams.amountsOut[i]);

288   IVirtualAccount(userAccount).userAddress(),

328   IVirtualAccount(userAccount).call(calls);

337   IVirtualAccount(userAccount).call(calls);

340   IVirtualAccount(userAccount).withdrawERC20(outputParams.outputToken, outputParams.amountOut);

344   IVirtualAccount(userAccount).userAddress(),

364   IVirtualAccount(userAccount).call(calls);

377   IVirtualAccount(userAccount).userAddress(),

416   IVirtualAccount(userAccount).call(calls);

425   IVirtualAccount(userAccount).call(calls);

428   IVirtualAccount(userAccount).withdrawERC20(outputParams.outputToken, outputParams.amountOut);

432   IVirtualAccount(userAccount).userAddress(),

452   IVirtualAccount(userAccount).call(calls);

456   IVirtualAccount(userAccount).withdrawERC20(outputParams.outputTokens[i], outputParams.amountsOut[i]);

465   IVirtualAccount(userAccount).userAddress(),

524   IBridgeAgent(_bridgeAgentAddress).callOutAndBridge{value: msg.value}(

566   IBridgeAgent(_bridgeAgentAddress).callOutAndBridgeMultiple{value: msg.value}(        
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L239

```solidity
File:  RootBridgeAgent.sol
364   if (!IPort(_localPortAddress).isLocalToken(_dParams.hToken, _srcChainId)) {

377   IPort(_localPortAddress).bridgeToRoot(

554   IPort(localPortAddress).toggleVirtualAccountApproved(userAccount, localRouterAddress);

574   IPort(localPortAddress).toggleVirtualAccountApproved(userAccount, localRouterAddress);

592   IPort(localPortAddress).toggleVirtualAccountApproved(userAccount, localRouterAddress);

614   IPort(localPortAddress).toggleVirtualAccountApproved(userAccount, localRouterAddress);

632   IPort(localPortAddress).toggleVirtualAccountApproved(userAccount, localRouterAddress);

654   IPort(localPortAddress).toggleVirtualAccountApproved(userAccount, localRouterAddress);

837   IBranchBridgeAgent(callee).lzReceive(0, "", 0, _payload);

915   ILayerZeroEndpoint(lzEndpointAddress).send{value: _value}(

929   IBranchBridgeAgent(callee).lzReceive(0, "", 0, payload);

940   ILayerZeroEndpoint(lzEndpointAddress).send{value: address(this).balance}(
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L364

```solidity
File:  RootBridgeAgentExecutor.sol
56   IRouter(_router).executeResponse(_payload[PARAMS_TKN_START:], _srcChainId);

72   IRouter(_router).execute{value: msg.value}(_payload[PARAMS_TKN_START:], _srcChainId);

102  IRouter(_router).executeDepositSingle{value: msg.value}(

136  IRouter(_router).executeDepositMultiple{value: msg.value}(

156  IRouter(_router).executeSigned{value: msg.value}(_payload[PARAMS_TKN_START_SIGNED:], _account, _srcChainId);

187  IRouter(_router).executeSignedDepositSingle{value: msg.value}(

225  IRouter(_router).executeSignedDepositMultiple{value: msg.value}(

245  RootBridgeAgent(payable(msg.sender)).bridgeIn(_recipient, _dParams, _srcChainId);

347  RootBridgeAgent(payable(msg.sender)).bridgeInMultiple(_recipient, dParams, _srcChainId);
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L56

```solidity
File:   RootPort.sol
161   IBridgeAgent(_coreRootBridgeAgent).syncBranchBridgeAgent(_coreLocalBranchBridgeAgent, localChainId);

290   if (_deposit > 0) if (!ERC20hTokenRoot(_hToken).mint(_recipient, _deposit, _srcChainId)) revert UnableToMint();

321   ERC20hTokenRoot(_hToken).burn(_from, _amount, _srcChainId);

332   ERC20hTokenRoot(_hToken).burn(_from, _amount, localChainId);

342   if (!ERC20hTokenRoot(_hToken).mint(_to, _amount, localChainId)) revert UnableToMint();

401   if (!IBridgeAgent(_rootBridgeAgent).isBranchBridgeAgentAllowed(_branchChainId)) {

404   IBridgeAgent(_rootBridgeAgent).syncBranchBridgeAgent(_newBranchBridgeAgent, _branchChainId);

452   IERC20hTokenRootFactory(ICoreRootRouter(coreRootRouterAddress).hTokenFactoryAddress()).createToken(

458   IBridgeAgent(ICoreRootRouter(coreRootRouterAddress).bridgeAgentAddress()).syncBranchBridgeAgent(

531   ICoreRootRouter(coreRootRouterAddress).setCoreBranch{value: msg.value}(

547   IBridgeAgent(coreRootBridgeAgentAddress).syncBranchBridgeAgent(_coreBranchBridgeAgent, _dstChainId);
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L161

```solidity
File:  VirtualAccount.sol
62   ERC721(_token).transferFrom(address(this), msg.sender, _tokenId);

161  if (!IRootPort(localPortAddress).isRouterApproved(this, msg.sender)) {
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L62


## [G‑04] Avoid updating storage when the value hasn't changed
If the old value is equal to the new value, not re-storing the value will avoid a Gsreset (2900 gas), potentially at the expense of a Gcoldsload (2100 gas) or a Gwarmaccess (100 gas)

There are 5 instances of this issue:

```solidity
File:  BaseBranchRouter.sol
62   localBridgeAgentAddress = _localBridgeAgentAddress;
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L62

```solidity
File:  MulticallRootRouter.sol
112  bridgeAgentAddress = payable(_bridgeAgentAddress);
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L112

```solidity
File: factories/ERC20hTokenBranchFactory.sol
74  localCoreRouterAddress = _coreRouter;
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenBranchFactory.sol#L74

```solidity
File:  factories/ERC20hTokenRootFactory.sol
51  coreRootRouterAddress = _coreRouter;
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenRootFactory.sol#L51

```solidity
File:  CoreRootRouter.sol
86   bridgeAgentAddress = payable(_bridgeAgentAddress);
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L86


## [G‑05] Using calldata instead of memory for read-only arguments in external functions saves gas
When a function with a memory array is called externally, the abi.decode() step has to copy read each index of the calldata to memory. Each copy costs at least 60 gas (i.e. 60 * <mem_array>.length). Using calldata directly, obviates the need for copies of words of the struct/array not being read. Note that even if an interface defines a function as having memory arguments, it's still valid for implementation contracts to use calldata arguments instead.

If the array is passed to an internal function which passes the array to another internal function where the array is modified and therefore memory is used in the external call, it's still more gass-efficient to use calldata when the external function uses modifiers, since the modifiers may prevent the internal functions from being called. Structs have the same overhead as an array of length one

Note that I've also flagged instances where the function is public but can be marked as external since it's not called by the contract, and cases where a constructor is involved

There are 16 instances of this issue:
```solidity
File:  BranchPort.sol
248   address[] memory _localAddresses,
249   address[] memory _underlyingAddresses,

290   address[] memory _localAddresses,
291   address[] memory _underlyingAddresses,
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L248-L249

```solidity
File:  interfaces/IBranchPort.sol
105   address[] memory _localAddresses,
106   address[] memory _underlyingAddresses,

137   address[] memory _localAddresses,
138   address[] memory _underlyingAddresses,
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/IBranchPort.sol#L105-L106

```solidity
File:  interfaces/IRootBridgeAgent.sol
172  bytes memory _params,
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/IRootBridgeAgent.sol#L172

```solidity
File:  interfaces/IRootRouter.sol
25  function executeResponse(bytes memory params, uint16 srcChainId) external payable;

33  function execute(bytes memory params, uint16 srcChainId) external payable;

42  function executeDepositSingle(bytes memory params, DepositParams memory dParams, uint16 srcChainId)
        external
        payable;

53   function executeDepositMultiple(bytes memory params, DepositMultipleParams memory dParams, uint16 srcChainId)
        external
        payable;    

63  function executeSigned(bytes memory params, address userAccount, uint16 srcChainId) external payable;

72  function executeSignedDepositSingle(
        bytes memory params,
        DepositParams memory dParams,
        address userAccount,
        uint16 srcChainId
    ) external payable;

86   function executeSignedDepositMultiple(
        bytes memory params,
        DepositMultipleParams memory dParams,
        address userAccount,
        uint16 srcChainId
    ) external payable;    
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/IRootRouter.sol#L25



## [G-06] Access mappings directly rather than using accessor functions
Saves having to do two JUMP instructions, along with stack setup
There are 2 instances of this issue:
```solidity
File:  BranchBridgeAgent.sol
157  return getDeposit[_depositNonce];
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L157

```solidity
File:  RootBridgeAgent.sol
136   return getSettlement[_settlementNonce];
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L136


## [G-07] A modifier used only once and not being inherited should be inlined to save gas
When you use a modifier in Solidity, Solidity generates code to check the conditions of the modifier and execute the modified function if the conditions are met. This generated code can consume gas, especially if the modifier is used frequently or if the modified function is called multiple times.

By inlining a modifier that is used only once and not being inherited, you can eliminate the overhead of the generated code and reduce the gas cost of your contract.

There are 6 instances of this issue:
```solidity
File:  BranchBridgeAgent.sol
// @audit only used in BranchBridgeAgent.lzReceiveNonBlocking function line:590
930  modifier requiresEndpoint(address _endpoint, bytes calldata _srcAddress) {
        _requiresEndpoint(_endpoint, _srcAddress);
        _;
    }

// @audit only used in BranchBridgeAgent.callOutSystem function line:180    
947 modifier requiresRouter() {
        if (msg.sender != localRouterAddress) revert UnrecognizedRouter();
        _;
    }    
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L930

```solidity
File:  BranchPort.sol
// @audit only used in BranchPort.addBridgeAgent function line:319

553  modifier requiresBridgeAgentFactory() {
        if (!isBridgeAgentFactory[msg.sender]) revert UnrecognizedBridgeAgentFactory();
        _;
    }

// @audit only used in BranchPort.manage function line:144
559  modifier requiresPortStrategy(address _token) {
        if (!isStrategyToken[_token]) revert UnrecognizedStrategyToken();
        if (!isPortStrategy[msg.sender][_token]) revert UnrecognizedPortStrategy();
        _;
    }    
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L553

```solidity
File:  RootBridgeAgent.sol
// @audit only used in RootBridgeAgent.syncBranchBridgeAgent function line:1179
1226   modifier requiresPort() {
        if (msg.sender != localPortAddress) revert UnrecognizedPort();
        _;
    }

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1226

```solidity
File:  RootPort.sol
// @audit only used in RootPort.addBridgeAgent function line:382
557  modifier requiresBridgeAgentFactory() {
        if (!isBridgeAgentFactory[msg.sender]) revert UnrecognizedBridgeAgentFactory();
        _;
    }
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L557

## [G-08] Cache external calls outside of loop to avoid re-calling function on each iteration
Performing STATICCALLs that do not depend on variables incremented in loops should always try to be avoided within the loop. In the following instances, we are able to cache the external calls outside of the loop to save a STATICCALL (100 gas) per loop iteration.

There are 7 instances of this issue:
```solidity
File:  MulticallRootRouter.sol
279  IVirtualAccount(userAccount).withdrawERC20(outputParams.outputTokens[i], outputParams.amountsOut[i]);

368  IVirtualAccount(userAccount).withdrawERC20(outputParams.outputTokens[i], outputParams.amountsOut[i]);

456  IVirtualAccount(userAccount).withdrawERC20(outputParams.outputTokens[i], outputParams.amountsOut[i]);
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L279

```solidity
File:  RootBridgeAgent.sol
328                  IPort(localPortAddress).bridgeToRoot(
                    msg.sender,
                    IPort(localPortAddress).getGlobalTokenFromLocal(_hToken, _dstChainId),


1072            hTokens[i] = IPort(localPortAddress).getLocalTokenFromGlobal(_globalAddresses[i], _dstChainId);
1073            tokens[i] = IPort(localPortAddress).getUnderlyingTokenFromLocal(hTokens[i], _dstChainId);                    
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L328-L330


## [G-09] Use assembly in place of abi.decode to extract calldata values more efficiently
Instead of using abi.decode, we can use assembly to decode our desired calldata values directly. This will allow us to avoid decoding calldata values that we will not use.
https://code4rena.com/reports/2023-08-goodentry#gas-optimizations:~:text=%5BG%2D02%5D%20Use%20assembly%20in%20place%20of%20abi.decode%20to%20extract%20calldata%20values%20more%20efficiently

There are 29 instances of this issue:
```solidity
File:   ArbitrumCoreBranchRouter.sol
134  ) = abi.decode(_data[1:], (address, address, address, address, address, GasParams));

147   (address bridgeAgentFactoryAddress) = abi.decode(_data[1:], (address));

153   (address branchBridgeAgent) = abi.decode(_data[1:], (address));

158    (address underlyingToken, uint256 minimumReservesRatio) = abi.decode(_data[1:], (address, uint256));

164   abi.decode(_data[1:], (address, address, uint256, bool));
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumCoreBranchRouter.sol#L134

```solidity
File:  CoreBranchRouter.sol
96    ) = abi.decode(_params[1:], (address, string, string, uint8, address, GasParams));

108     ) = abi.decode(_params[1:], (address, address, address, address, address, GasParams));

116   (address bridgeAgentFactoryAddress) = abi.decode(_params[1:], (address));

122   (address branchBridgeAgent) = abi.decode(_params[1:], (address));

128     (address underlyingToken, uint256 minimumReservesRatio) = abi.decode(_params[1:], (address, uint256));

135    abi.decode(_params[1:], (address, address, uint256, bool));

141     (address coreBranchRouter, address coreBranchBridgeAgent) = abi.decode(_params[1:], (address, address));
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L96

```solidity
File:  CoreRootRouter.sol
309   = abi.decode(_encodedData[1:], (address, address, string, string, uint8));

315   (address globalAddress, address localAddress) = abi.decode(_encodedData[1:], (address, address));

321  (address newBranchBridgeAgent, address rootBridgeAgent) = abi.decode(_encodedData[1:], (address, address));

339   abi.decode(_encodedData[1:], (address, address, uint16, GasParams[2]));
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L309

```solidity
File:  MulticallRootRouter.sol
144  (IMulticall.Call[] memory callData) = abi.decode(_decode(encodedData[1:]), (IMulticall.Call[]));

157   ) = abi.decode(_decode(encodedData[1:]), (IMulticall.Call[], OutputParams, uint16, GasParams));

181    ) = abi.decode(_decode(encodedData[1:]), (IMulticall.Call[], OutputMultipleParams, uint16, GasParams));

236  Call[] memory calls = abi.decode(_decode(encodedData[1:]), (Call[]));

245   abi.decode(_decode(encodedData[1:]), (Call[], OutputParams, uint16, GasParams));

272    ) = abi.decode(_decode(encodedData[1:]), (Call[], OutputMultipleParams, uint16, GasParams));

325   Call[] memory calls = abi.decode(_decode(encodedData[1:]), (Call[]));

334    abi.decode(_decode(encodedData[1:]), (Call[], OutputParams, uint16, GasParams));

361   ) = abi.decode(_decode(encodedData[1:]), (Call[], OutputMultipleParams, uint16, GasParams));

413    Call[] memory calls = abi.decode(_decode(encodedData[1:]), (Call[]));

422    abi.decode(_decode(encodedData[1:]), (Call[], OutputParams, uint16, GasParams));

449    ) = abi.decode(_decode(encodedData[1:]), (Call[], OutputMultipleParams, uint16, GasParams));
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L144

```solidity
File:  RootBridgeAgent.sol
664   (nonce, owner, params, gParams) = abi.decode(_payload[PARAMS_START:], (uint32, address, bytes, GasParams));
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L664


## [G-10] Use assembly to perform efficient back-to-back calls
If a similar external call is performed back-to-back, we can use assembly to reuse any function signatures and function parameters that stay the same. In addition, we can also reuse the same memory space for each function call (scratch space + free memory pointer + zero slot), which can potentially allow us to avoid memory expansion costs.
Note: In order to do this optimization safely we will cache the free memory pointer value and restore it once we are done with our function calls. We will also set the zero slot back to 0 if neccessary.
[Reffrence](https://code4rena.com/reports/2023-05-ajna#g-10-use-assembly-to-perform-efficient-back-to-back-calls)

There are 12 instances of this issue:
```solidity
File:  factories/ERC20hTokenBranchFactory.sol
66    ERC20(_wrappedNativeTokenAddress).name(),
67    ERC20(_wrappedNativeTokenAddress).symbol(),
68    ERC20(_wrappedNativeTokenAddress).decimals(),
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenBranchFactory.sol#L66-L68

```solidity
File:  BaseBranchRouter.sol
63    localPortAddress = IBridgeAgent(_localBridgeAgentAddress).localPortAddress();
64    bridgeAgentExecutorAddress = IBridgeAgent(_localBridgeAgentAddress).bridgeAgentExecutorAddress();
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L63

```solidity
File:  BranchPort.sol
175     uint256 currBalance = ERC20(_token).balanceOf(address(this));

        // Withdraw tokens from startegy
178     IPortStrategy(msg.sender).withdraw(address(this), _token, _amount);

        // Check if _token balance has increased by _amount
181    require(ERC20(_token).balanceOf(address(this)) - currBalance == _amount, "Port Strategy Withdraw Failed");
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L175-L181

```solidity
File:  CoreBranchRouter.sol
317          if (!IPort(_localPortAddress).isPortStrategy(_portStrategy, _underlyingToken)) {
            // If Port Strategy is not active, add new Port Strategy.
            IPort(_localPortAddress).addPortStrategy(_portStrategy, _underlyingToken, _dailyManagementLimit);
        } else if (_isUpdateDailyLimit) {
            // Or update daily limit.
            IPort(_localPortAddress).updatePortStrategy(_portStrategy, _underlyingToken, _dailyManagementLimit);
        } else {
            // Or Toggle Port Strategy.
            IPort(_localPortAddress).togglePortStrategy(_portStrategy, _underlyingToken);
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L317-L325

```solidity
File:  CoreRootRouter.sol
461     if (IPort(rootPortAddress).isGlobalAddress(_underlyingAddress)) revert TokenAlreadyAdded();
462     if (IPort(rootPortAddress).isLocalToken(_underlyingAddress, _srcChainId)) revert TokenAlreadyAdded();
463     if (IPort(rootPortAddress).isUnderlyingToken(_underlyingAddress, _srcChainId)) revert TokenAlreadyAdded();

483   if (IPort(rootPortAddress).isLocalToken(_localAddress, _dstChainId)) revert TokenAlreadyAdded();

486   IPort(rootPortAddress).setLocalAddress(_globalAddress, _localAddress, _dstChainId);
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L461-L463

```solidity
File:  MulticallRootRouter.sol
248         IVirtualAccount(userAccount).call(calls);

            // Withdraw assets from Virtual Account
            IVirtualAccount(userAccount).withdrawERC20(outputParams.outputToken, outputParams.amountOut);


425         IVirtualAccount(userAccount).call(calls);

            // Withdraw assets from Virtual Account
            IVirtualAccount(userAccount).withdrawERC20(outputParams.outputToken, outputParams.amountOut);
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L248-L251

```solidity
File:  RootBridgeAgent.sol
549         VirtualAccount userAccount = IPort(localPortAddress).fetchVirtualAccount(
                address(uint160(bytes20(_payload[PARAMS_START:PARAMS_START_SIGNED])))
            );

            // Toggle Router Virtual Account use for tx execution
554         IPort(localPortAddress).toggleVirtualAccountApproved(userAccount, localRouterAddress);


587         VirtualAccount userAccount = IPort(localPortAddress).fetchVirtualAccount(
                address(uint160(bytes20(_payload[PARAMS_START:PARAMS_START_SIGNED])))
            );

            // Toggle Router Virtual Account use for tx execution
592         IPort(localPortAddress).toggleVirtualAccountApproved(userAccount, localRouterAddress);

627         VirtualAccount userAccount = IPort(localPortAddress).fetchVirtualAccount(
                address(uint160(bytes20(_payload[PARAMS_START:PARAMS_START_SIGNED])))
            );

            // Toggle Router Virtual Account use for tx execution
632         IPort(localPortAddress).toggleVirtualAccountApproved(userAccount, localRouterAddress);


1072        hTokens[i] = IPort(localPortAddress).getLocalTokenFromGlobal(_globalAddresses[i], _dstChainId);
1073        tokens[i] = IPort(localPortAddress).getUnderlyingTokenFromLocal(hTokens[i], _dstChainId);
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L549-L554

## [G-11] Use do while loops instead of for loops
A do while loop will cost less gas since the condition is not being checked for the first iteration.
[Reffrence](https://code4rena.com/reports/2023-05-ajna#g-09-use-do-while-loops-instead-of-for-loops)

There are 15 instances of this issue:
```solidity
File:  BaseBranchRouter.sol
192  for (uint256 i = 0; i < _hTokens.length;) {
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L192

```solidity
File: BranchBridgeAgent.sol
447  for (uint256 i = 0; i < deposit.tokens.length;) {

513  for (uint256 i = 0; i < numOfAssets;) {
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L447

```solidity
File:  BranchPort.sol
257  for (uint256 i = 0; i < length;) {

305  for (uint256 i = 0; i < length;) {    
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L257

```solidity
File:  MulticallRootRouter.sol
278   for (uint256 i = 0; i < outputParams.outputTokens.length;) {

367   for (uint256 i = 0; i < outputParams.outputTokens.length;) {

455   for (uint256 i = 0; i < outputParams.outputTokens.length;) {

557   for (uint256 i = 0; i < outputTokens.length;) {            
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L278

```solidity
File:  RootBridgeAgent.sol
318   for (uint256 i = 0; i < settlement.hTokens.length;) {

399   for (uint256 i = 0; i < length;) {

1070  for (uint256 i = 0; i < hTokens.length;) {
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L318

```solidity
File:  VirtualAccount.sol
70   for (uint256 i = 0; i < length;) {

90   for (uint256 i = 0; i < length;) {    
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L70

```solidity
File:  RootBridgeAgentExecutor.sol
281  for (uint256 i = 0; i < uint256(uint8(numOfAssets));) {
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L281


## [G-12] OR in if-condition can be rewritten to two single if conditions
Refactoring the if-condition in a way it won't be containing || operator will save more gas.
There are 1 instances of this issue:
```solidity
File:  BranchPort.sol
363     if (_minimumReservesRatio >= DIVISIONER || _minimumReservesRatio < MIN_RESERVE_RATIO) {
            revert InvalidMinimumReservesRatio();
        }

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L363-L365

`Fix code`

```diff
+  if (_minimumReservesRatio >= DIVISIONER) revert InvalidMinimumReservesRatio();
+  if (_minimumReservesRatio < MIN_RESERVE_RATIO) revert InvalidMinimumReservesRatio();
```


## [G-13] Replace state variable reads and writes within loops with local variable reads and writes.
Reading and writing local variables is cheap, whereas reading and writing state variables that are stored in contract storage is expensive.
```
function badCode() external {                
  for(uint256 i; i < myArray.length; i++) { // state reads
    myCounter++; // state reads and writes
  }        
}
```
```
function goodCode() external {
    uint256 length = myArray.length; // one state read
    uint256 local_mycounter = myCounter; // one state read
    for(uint256 i; i < length; i++) { // local reads
        local_mycounter++; // local reads and writes  
    }
    myCounter = local_mycounter; // one state write
}
```
There are 2 instances of this issue:
```solidity
File:  BranchBridgeAgent.sol
// @audit deposit is storage
447  for (uint256 i = 0; i < deposit.tokens.length;) {
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L447

```solidity
File:  RootBridgeAgent.sol
// @audit settlement is storage
318  for (uint256 i = 0; i < settlement.hTokens.length;) {
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L318


## [G-14] >=/<= costs less gas than >/<
The compiler uses opcodes GT and ISZERO for code that uses >, but only requires LT for >=. A similar behaviour applies for >, which uses opcodes LT and ISZERO, but only requires GT for <=.
[Reffrence](https://gist.github.com/CloudEllie/05fda0fdecba2ed7d85f68c709e46c8d#gas-18--costs-less-gas-than-)

`Note` Bots find only > this but i find < issue

There are 17 instances of this issue:

```solidity
File:  BaseBranchRouter.sol
192  for (uint256 i = 0; i < _hTokens.length;) {
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L192

```solidity
File: BranchBridgeAgent.sol
447  for (uint256 i = 0; i < deposit.tokens.length;) {

513  for (uint256 i = 0; i < numOfAssets;) {
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L447

```solidity
File:  BranchPort.sol
257  for (uint256 i = 0; i < length;) {

305  for (uint256 i = 0; i < length;) {

363  if (_minimumReservesRatio >= DIVISIONER || _minimumReservesRatio < MIN_RESERVE_RATIO) {        
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L257

```solidity
File:  MulticallRootRouter.sol
278   for (uint256 i = 0; i < outputParams.outputTokens.length;) {

367   for (uint256 i = 0; i < outputParams.outputTokens.length;) {

455   for (uint256 i = 0; i < outputParams.outputTokens.length;) {

557   for (uint256 i = 0; i < outputTokens.length;) {            
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L278

```solidity
File:  RootBridgeAgent.sol
318   for (uint256 i = 0; i < settlement.hTokens.length;) {

399   for (uint256 i = 0; i < length;) {

1070  for (uint256 i = 0; i < hTokens.length;) {

1142   if (_amount < _deposit) revert InvalidInputParams();

1158  if (IERC20hTokenRoot(_globalAddress).getTokenBalance(_dstChainId) < _deposit) {
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L318

```solidity
File:  VirtualAccount.sol
70   for (uint256 i = 0; i < length;) {

90   for (uint256 i = 0; i < length;) {    
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L70


## [G-15] Use Assembly To Check For address(0)
generally more gas-efficient to use assembly to check for a zero address (address(0)).

`Note` bots only find check address zero with == but i find !=

There are 35 instances of this issue:
```solidity
File:  ArbitrumBranchPort.sol
39   require(_rootPortAddress != address(0), "Root Port Address cannot be 0");
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol#L39

```solidity
File:  BaseBranchRouter.sol
61   require(_localBridgeAgentAddress != address(0), "Bridge Agent address cannot be 0");
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L61

```solidity
File:  BranchBridgeAgent.sol
125  require(_rootBridgeAgentAddress != address(0), "Root Bridge Agent Address cannot be the zero address.");

127  require(_localRouterAddress != address(0), "Local Router Address cannot be the zero address.");
     require(_localPortAddress != address(0), "Local Port Address cannot be the zero address.");
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L125

```solidity
File:  BranchPort.sol
109  require(_owner != address(0), "Owner is zero address");

126  require(_coreBranchRouter != address(0), "CoreBranchRouter is zero address");
     require(_bridgeAgentFactory != address(0), "BridgeAgentFactory is zero address");

332  require(coreBranchRouterAddress != address(0), "CoreRouter address is zero");
     require(_newCoreRouter != address(0), "New CoreRouter address is zero");     
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L109

```solidity
File:  MulticallRootRouter.sol
93      require(_localPortAddress != address(0), "Local Port Address cannot be 0");
        require(_multicallAddress != address(0), "Multicall Address cannot be 0");

110     require(_bridgeAgentAddress != address(0), "Bridge Agent Address cannot be 0");        
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L93

```solidity
File:  RootBridgeAgent.sol
111     require(_lzEndpointAddress != address(0), "Layerzero Enpoint Address cannot be zero address");
        require(_localPortAddress != address(0), "Port Address cannot be zero address");
        require(_localRouterAddress != address(0), "Router Address cannot be zero address");

323     if (_hToken != address(0)) {        
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L111

```solidity
File:  RootPort.sol
130     require(_bridgeAgentFactory != address(0), "Bridge Agent Factory cannot be 0 address.");
        require(_coreRootRouter != address(0), "Core Root Router cannot be 0 address.");

152     require(_coreRootBridgeAgent != address(0), "Core Root Bridge Agent cannot be 0 address.");
        require(_coreLocalBranchBridgeAgent != address(0), "Core Local Branch Bridge Agent cannot be 0 address.");
        require(_localBranchPortAddress != address(0), "Local Branch Port Address cannot be 0 address.");    
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L130

```solidity
File:  factories/ArbitrumBranchBridgeAgentFactory.sol
57      require(_coreRootBridgeAgent != address(0), "Core Root Bridge Agent Address cannot be 0");
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ArbitrumBranchBridgeAgentFactory.sol#L57

```solidity
File:  factories/BranchBridgeAgentFactory.sol
61  require(_rootBridgeAgentFactoryAddress != address(0), "Root Bridge Agent Factory Address cannot be 0");

66      require(_localCoreBranchRouterAddress != address(0), "Core Branch Router Address cannot be 0");
        require(_localPortAddress != address(0), "Port Address cannot be 0");
        require(_owner != address(0), "Owner cannot be 0");

88  require(_coreRootBridgeAgent != address(0), "Core Root Bridge Agent cannot be 0");        
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/BranchBridgeAgentFactory.sol#L61

```solidity
File:  factories/ERC20hTokenBranchFactory.sol
43  require(_localPortAddress != address(0), "Port address cannot be 0");

61  require(_coreRouter != address(0), "CoreRouter address cannot be 0");
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenBranchFactory.sol#L43

```solidity
File:  factories/ERC20hTokenRootFactory.sol
35    require(_rootPortAddress != address(0), "Root Port Address cannot be 0");

50    require(_coreRouter != address(0), "CoreRouter address cannot be 0");
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenRootFactory.sol#L35

```solidity
File:  factories/RootBridgeAgentFactory.sol
32   require(_rootPortAddress != address(0), "Root Port Address cannot be 0");
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/RootBridgeAgentFactory.sol#L32

```solidity
File:  token/ERC20hTokenRoot.sol
39   require(_rootPortAddress != address(0), "Root Port Address cannot be 0");
     require(_factoryAddress != address(0), "Factory Address cannot be 0");
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/token/ERC20hTokenRoot.sol#L39

## [G-16] Use ERC721A instead ERC721
ERC721A standard, ERC721A is an improvement standard for ERC721 tokens. It was proposed by the Azuki team and used for developing their NFT collection. Compared with ERC721, ERC721A is a more gas-efficient standard to mint a lot of of NFTs simultaneously. It allows developers to mint multiple NFTs at the same gas price. This has been a great improvement due to Ethereum's sky-rocketing gas fee.
[Reffrence](https://nextrope.com/erc721-vs-erc721a-2/)
There are 1 instances of this issue:
```solidity
File: VirtualAccount.sol
7  import {ERC721} from "solmate/tokens/ERC721.sol";
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L7






## [G-17] Using Fixed Bytes Is Cheaper Than Using String
As a rule of thumb, use bytes for arbitrary-length raw byte data and string for arbitrary-length string (UTF-8) data.
If you can limit the length to a certain number of bytes, always use one of bytes1 to bytes32 because they are much cheaper.
There are 2 instances of this issue:
```solidity
File:  factories/ERC20hTokenBranchFactory.sol
26  string public chainName;

29  string public chainSymbol;
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenBranchFactory.sol#L26

## [G‑18] Empty blocks should be removed or emit something
The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting.
If the contract is meant to be extended, the contract should be abstract and the function signatures be added without any default implementation.

There are 1 instances of this issue:

```solidity
File:   ArbitrumBranchBridgeAgent.sol
89       function retrySettlement(uint32, bytes calldata, GasParams[2] calldata, bool) external payable override lock {}
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchBridgeAgent.sol#L89


## [G-19] Use Modifiers Instead of Functions To Save Gas
Example of two contracts with modifiers and internal view function:
```
// SPDX-License-Identifier: MIT
pragma solidity 0.8.9;
contract Inlined {
    function isNotExpired(bool _true) internal view {
        require(_true == true, "Exchange: EXPIRED");
    }
function foo(bool _test) public returns(uint){
            isNotExpired(_test);
            return 1;
    }
}
// SPDX-License-Identifier: MIT
pragma solidity 0.8.9;
contract Modifier {
modifier isNotExpired(bool _true) {
        require(_true == true, "Exchange: EXPIRED");
        _;
    }
function foo(bool _test) public isNotExpired(_test)returns(uint){
        return 1;
    }
}
```
Differences:
```
Deploy Modifier.sol
108727
Deploy Inlined.sol
110473
Modifier.foo
21532
Inlined.foo
21556
```
This with 0.8.9 compiler and optimization enabled. As you can see it's cheaper to deploy with a modifier, and it will save you about 30 gas. But sometimes modifiers increase code size of the contract.

There are 5 instances of this issue:
```solidity
File:  RootPort.sol
211   function isGlobalToken(address _globalAddress, uint256 _srcChainId) external view override returns (bool) {

216   function isLocalToken(address _localAddress, uint256 _srcChainId) external view override returns (bool) {

221   function isLocalToken(address _localAddress, uint256 _srcChainId, uint256 _dstChainId)

230   function isUnderlyingToken(address _underlyingToken, uint256 _srcChainId) external view override returns (bool) {
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L211

```solidity
File:  VirtualAccount.sol
147  function isContract(address addr) internal view returns (bool) {
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L147











## [G-20] State variables can be packed to use fewer storage slots
The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to eachother in storage and this will pack the values together into a single 32 byte storage slot (if the values combined are <= 32 bytes). If the variables packed together are retrieved together in functions we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a Gwarmaccess (100 gas) versus a Gcoldsload (2100 gas).

There are 1 instances of this issue:
```solidity
File:  interfaces/BridgeAgentConstants.sol
contract BridgeAgentConstants {
    // Settlement / Deposit Execution Status

    uint8 internal constant STATUS_READY = 0;

    uint8 internal constant STATUS_DONE = 1;

    uint8 internal constant STATUS_RETRIEVE = 2;

    // Settlement / Deposit Redeeem Status

    uint8 internal constant STATUS_FAILED = 1;

    uint8 internal constant STATUS_SUCCESS = 0;

    // Payload Encoding / Decoding

    uint256 internal constant PARAMS_START = 1;

    uint256 internal constant PARAMS_START_SIGNED = 21;

    uint256 internal constant PARAMS_TKN_START = 5;

    uint256 internal constant PARAMS_TKN_START_SIGNED = 25;

    uint256 internal constant PARAMS_ENTRY_SIZE = 32;

    uint256 internal constant PARAMS_ADDRESS_SIZE = 20;

    uint256 internal constant PARAMS_TKN_SET_SIZE = 109;

    uint256 internal constant PARAMS_TKN_SET_SIZE_MULTIPLE = 128;

    uint256 internal constant ADDRESS_END_OFFSET = 12;

    uint256 internal constant PARAMS_AMT_OFFSET = 64;

    uint256 internal constant PARAMS_DEPOSIT_OFFSET = 96;

    uint256 internal constant PARAMS_END_OFFSET = 6;

    uint256 internal constant PARAMS_END_SIGNED_OFFSET = 26;

    uint256 internal constant PARAMS_SETTLEMENT_OFFSET = 129;

    // Deposit / Settlement Multiple Max

    uint256 internal constant MAX_TOKENS_LENGTH = 255;
}

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/BridgeAgentConstants.sol#L10-L58

`Fixe code`: the above BridgeAgentConstants state varible spend 16 storage slots in EVM  but the fix code is only use 1 storage slot in EVM

```solidity
contract BridgeAgentConstants {
    // Settlement / Deposit Execution Status

    uint8 internal constant STATUS_READY = 0;

    uint8 internal constant STATUS_DONE = 1;

    uint8 internal constant STATUS_RETRIEVE = 2;

    // Settlement / Deposit Redeem Status

    uint8 internal constant STATUS_FAILED = 1;

    uint8 internal constant STATUS_SUCCESS = 0;

    // Payload Encoding / Decoding

    uint8 internal constant PARAMS_START = 1;

    uint8 internal constant PARAMS_START_SIGNED = 21;

    uint8 internal constant PARAMS_TKN_START = 5;

    uint8 internal constant PARAMS_TKN_START_SIGNED = 25;

    uint8 internal constant PARAMS_ENTRY_SIZE = 32;

    uint8 internal constant PARAMS_ADDRESS_SIZE = 20;

    uint8 internal constant PARAMS_TKN_SET_SIZE = 109;

    uint8 internal constant PARAMS_TKN_SET_SIZE_MULTIPLE = 128;

    uint8 internal constant ADDRESS_END_OFFSET = 12;

    uint8 internal constant PARAMS_AMT_OFFSET = 64;

    uint8 internal constant PARAMS_DEPOSIT_OFFSET = 96;

    uint8 internal constant PARAMS_END_OFFSET = 6;

    uint8 internal constant PARAMS_END_SIGNED_OFFSET = 26;

    uint8 internal constant PARAMS_SETTLEMENT_OFFSET = 129;

    // Deposit / Settlement Multiple Max

    uint8 internal constant MAX_TOKENS_LENGTH = 255;
}
```


## [G-21] abi.encode() is less efficient than abi.encodepacked()
In terms of efficiency, abi.encodePacked() is generally considered to be more gas-efficient than abi.encode(), because it skips the step of adding function signatures and other metadata to the encoded data. However, this comes at the cost of reduced safety, as abi.encodePacked() does not perform any type checking or padding of data.
[Refference](https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison)

Note: If we change abi.encode to abi.encodePacked in this scenario, there is still no chance of collision 
There are 4 instances of this issue:
```solidity
File:  ArbitrumCoreBranchRouter.sol
112  bytes memory data = abi.encode(newBridgeAgent, _rootBridgeAgent);
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumCoreBranchRouter.sol#L112

```solidity
File:  CoreBranchRouter.sol
72  bytes memory params = abi.encode(_underlyingAddress, newToken, newToken.name(), newToken.symbol(), decimals);

178 bytes memory params = abi.encode(_globalAddress, newToken);

226 bytes memory params = abi.encode(newBridgeAgent, _rootBridgeAgent);
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L72


## [G-22] When possible, use assembly instead of unchecked{++i}
You can also use unchecked{++i;} for even more gas savings but this will not check to see if i overflows. For best gas savings, use inline assembly, however, this limits the functionality you can achieve.

```
//loop with unchecked{++i}
function uncheckedPlusPlusI() public pure {
    uint256 j = 0;
    for (uint256 i; i < 10; ) {
        j++;
        unchecked {
            ++i;
        }
    }
}
```
Gas: 1329
```
//loop with inline assembly
function inlineAssemblyLoop() public pure {
    assembly {
        let j := 0
        for {
            let i := 0
        } lt(i, 10) {
            i := add(i, 0x01)
        } {
            j := add(j, 0x01)
        }
    }
}
```
Gas: 709


There are 15 instances of this issue:
```solidity
File:  BaseBranchRouter.sol
195  unchecked {
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L195

```solidity
File:  BranchBridgeAgent.sol
450  unchecked {

563  unchecked {
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L450

```solidity
File:  BranchPort.sol
270  unchecked {

308  unchecked {
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L270


```solidity
File:  MulticallRootRouter.sol
281  unchecked {

370  unchecked {

458  unchecked {

560  unchecked {
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L281

```solidity
File:  RootBridgeAgent.sol
337  unchecked {

412  unchecked {

1083 unchecked {
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L337

```solidity
File:  RootBridgeAgentExecutor.sol
332  unchecked {
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L332

```solidity
File:  VirtualAccount.sol
78  unchecked {

105 unchecked {
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L78

