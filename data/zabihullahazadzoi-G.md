# Gas Optimization

- Note: Instances mentioned in the issues (G-20), (G-15), (G-33), (G-3), (G-34) were missed from the bot.


### Summary
|Number|Issue|Instances|Estimated Gas Saved|
|-|:-|:-:|-:|
| [G-1]|Use `selfbalance()` instead of `address(this).balance` | 9 | - |
| [G-2]|Avoid contract existence checks by using low level calls | 109 | 10900 |
| [G-3]|Avoid updating storage when the value hasn’t changed | 93 | 74400 |
| [G-4]|Multiple accesses of the same mapping/array key/index should be cached | 23 | 966 |
| [G-5]|The result of a function call should be cached rather than re-calling the function | 62 | 6200 |
| [G-6]|State variables that are used multiple times in a function should be cached in stack variables | 27 | 2700 |
| [G-7]|Gas savings can be achieved by changing the model for assigning value to the structure | 10 | 2600 |
| [G-8]|Do not shrink Variables | 49 | - |
| [G-9]|Empty blocks should be removed or emit something` | 8 | - |
| [G-10]|Don't initialize variables with default value | 17 | - |
| [G-11]|A modifier used only once and not being inherited should be inlined to save gas | 10 | - |
| [G-12]|Usage of ints/uints smaller than 32 bytes incurs overhead | 216 | 11880 |
| [G-13]|Low level call can be optimized with assembly | 8 | 1984 |
| [G-14]|A mapping is more efficient than an array | 110 | - |
| [G-15]|Optimize names to save gas | 20 | 440 |
| [G-16]|Operator += costs more gas than <x> = <x> + <y> for variables | 6 | 678 |
| [G-17]|Change public state variable visibility to private | 94 | - |
| [G-18]|Duplicated require()/revert() checks should be refactored to a modifier Or function to save gas | 12 | - |
| [G-19]|State variable read in a loop | 4 | 388 |
| [G-20]|State variables only set in the constructor should be declared immutable  | 39 | 81783 |
| [G-21]|State variables only set in their definitions should be declared constant | 5 | 10485 |
| [G-22]|Unlimited gas consumption risk due to external call recipients | 8 | - |
| [G-23]|Use != 0 instead of > 0 for unsigned integer comparison | 21 | 84 |
| [G-24]|Unused named return variables without optimizer waste gas | 13 | 117 |
| [G-25]|Remove or replace unused state variables | 20 | - |
| [G-26]|Using bitmap to store bool states can save gas | 10 | - |
| [G-27]|Use do while loops instead of for loops | 15 | - |
| [G-28]|Use hardcode address instead of address(this) | 32 | - |
| [G-29]|Newer versions of solidity are more gas efficient | 44 | - |
| [G-30]|Use a more recent version of solidity | 44 | - |
| [G-31]|Use unchecked block for safe subtractions | 5 | 425 |
| [G-32]|Use assembly to write address/contract type storage values | 44 | 2200 |
| [G-33]|Use assembly to emit events | 10 | 380 |
| [G-34]|Using assembly to check for zero can save gas | 11 | 66 |



## [G-1] Use `selfbalance()` instead of `address(this).balance`
Use assembly when getting a contract's balance of ETH.

You can use `selfbalance()` instead of `address(this).balance` when getting your contract's balance of ETH to save gas.
Additionally, you can use `balance(address)` instead of `address.balance()` when getting an external contract's balance of ETH.

*Saves 15 gas when checking internal balance, 6 for external*


 ```solidity
File: src/BranchBridgeAgent.sol

 717:             value: address(this).balance

 737:             value: address(this).balance

 787:             value: address(this).balance

```
https://github.com/code-423n4/2023-09-maia/blob/6c21f6b739252367c5fe770fed3ff76eb3ca2c86/src/BranchBridgeAgent.sol#L717


```solidity
File: src/BranchBridgeAgentExecutor.sol

 92:             _recipient.safeTransferETH(address(this).balance);

 126:             _recipient.safeTransferETH(address(this).balance);

```
https://github.com/code-423n4/2023-09-maia/blob/6c21f6b739252367c5fe770fed3ff76eb3ca2c86/src/BranchBridgeAgentExecutor.sol#L92


```solidity
File: src/RootBridgeAgent.sol

 695:                 address(this).balance

 754:         (bool success,) = bridgeAgentExecutorAddress.call{value: address(this).balance}(_calldata);

 779:         (bool success,) = bridgeAgentExecutorAddress.call{value: address(this).balance}(_calldata);

 940:         ILayerZeroEndpoint(lzEndpointAddress).send{value: address(this).balance}(

```
 https://github.com/code-423n4/2023-09-maia/blob/6c21f6b739252367c5fe770fed3ff76eb3ca2c86/src/RootBridgeAgent.sol#L695
 

## [G-2] Avoid contract existence checks by using low level calls
Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.

Note: instances below were missed from bot.

 ```solidity
File: src/ArbitrumBranchBridgeAgent.sol

 /// @audit  depositToPort()
 70:         IArbPort(localPortAddress).depositToPort(msg.sender, msg.sender, underlyingAddress, amount);

 /// @audit  withdrawFromPort()
 80:         IArbPort(localPortAddress).withdrawFromPort(msg.sender, msg.sender, localAddress, amount);

 /// @audit  lzReceive()
 105:         IRootBridgeAgent(_rootBridgeAgentAddress).lzReceive(rootChainId, "", 0, _calldata);

 /// @audit  lzReceive()
 114:         IRootBridgeAgent(rootBridgeAgentAddress).lzReceive(

```
https://github.com/code-423n4/2023-09-maia/blob/6c21f6b739252367c5fe770fed3ff76eb3ca2c86/src/ArbitrumBranchBridgeAgent.sol#L70


```solidity
File: src/ArbitrumBranchPort.sol


 /// @audit  mintToLocalBranch()
 69:         IRootPort(_rootPortAddress).mintToLocalBranch(_recipient, _globalToken, _deposit);

 /// @audit  isGlobalToken()
 83:         if (!IRootPort(_rootPortAddress).isGlobalToken(_globalAddress, localChainId)) revert UnknownGlobalToken();


 /// @audit  burnFromLocalBranch()
 92:         IRootPort(_rootPortAddress).burnFromLocalBranch(_depositor, _globalAddress, _amount);

 /// @audit  bridgeToLocalBranchFromRoot()
 108:         IRootPort(rootPortAddress).bridgeToLocalBranchFromRoot(_recipient, _localAddress, _amount);

 /// @audit  bridgeToRootFromLocalBranch()
 134:                 IRootPort(rootPortAddress).bridgeToRootFromLocalBranch(_depositor, _localAddress, _amount - _deposit);

```
https://github.com/code-423n4/2023-09-maia/blob/6c21f6b739252367c5fe770fed3ff76eb3ca2c86/src/ArbitrumBranchPort.sol#L69


```solidity
File: src/ArbitrumCoreBranchRouter.sol


 /// @audit  callOutSystem()
 65:         IBridgeAgent(localBridgeAgentAddress).callOutSystem(

 /// @audit  isBridgeAgentFactory()
 97:             !IPort(_localPortAddress).isBridgeAgentFactory(

 /// @audit  createBridgeAgent()
 102:         address newBridgeAgent = IBridgeAgentFactory(_branchBridgeAgentFactory)

 /// @audit  callOutSystem()
 118:         IBridgeAgent(localBridgeAgentAddress).callOutSystem(

```
https://github.com/code-423n4/2023-09-maia/blob/6c21f6b739252367c5fe770fed3ff76eb3ca2c86/src/ArbitrumCoreBranchRouter.sol#65


```solidity
File: src/BaseBranchRouter.sol

 /// @audit  getDepositEntry()
 75:         return IBridgeAgent(localBridgeAgentAddress).getDepositEntry(_depositNonce);

 /// @audit  approve()
 168:                 ERC20(_hToken).approve(_localPortAddress, _amount - _deposit);

 /// @audit  approve()
 175:             ERC20(_token).approve(_localPortAddress, _deposit);

```
https://github.com/code-423n4/2023-09-maia/blob/6c21f6b739252367c5fe770fed3ff76eb3ca2c86/src/BaseBranchRouter.sol#L75


```solidity
File: src/BranchBridgeAgent.sol

 /// @audit  bridgeInMultiple()
 657:         IPort(localPortAddress).bridgeInMultiple(

 /// @audit  bridgeOut()
 988:         IPort(localPortAddress).bridgeOut(

 /// @audit  bridgeOutMultiple()
 1048:         IPort(localPortAddress).bridgeOutMultiple(

 /// @audit  bridgeIn()
 1088:                 IPort(localPortAddress).bridgeIn(

 /// @audit  withdraw()
 1097:             IPort(localPortAddress).withdraw(_recipient, _token, _deposit);

```
https://github.com/code-423n4/2023-09-maia/blob/6c21f6b739252367c5fe770fed3ff76eb3ca2c86/src/BranchBridgeAgent.sol#657


```solidity
File: src/BranchBridgeAgentExecutor.sol

 /// @audit  clearTokens()
 114:         SettlementMultipleParams memory sParams = BranchBridgeAgent(payable(msg.sender)).clearTokens(

```
https://github.com/code-423n4/2023-09-maia/blob/6c21f6b739252367c5fe770fed3ff76eb3ca2c86/src/BranchBridgeAgentExecutor.sol#114


```solidity
File: src/BranchPort.sol

 /// @audit  withdraw()
 178:         IPortStrategy(msg.sender).withdraw(address(this), _token, _amount);

 /// @audit  withdraw()
 210:         IPortStrategy(_strategy).withdraw(address(this), _token, amountToWithdraw);

 /// @audit  mint()
 503:         ERC20hTokenBranch(_localAddress).mint(_recipient, _amount);

 /// @audit  burn()
 527:             ERC20hTokenBranch(_localAddress).burn(_hTokenAmount);

```
https://github.com/code-423n4/2023-09-maia/blob/6c21f6b739252367c5fe770fed3ff76eb3ca2c86/src/BranchPort.sol#178


```solidity
File: src/CoreBranchRouter.sol


 /// @audit  setCoreBranchRouter()
 143:             IPort(localPortAddress).setCoreBranchRouter(coreBranchRouter, coreBranchBridgeAgent);

 /// @audit  isBridgeAgentFactory()
 211:         if (!IPort(_localPortAddress).isBridgeAgentFactory(_branchBridgeAgentFactory)) {

 /// @audit  isBridgeAgent()
 221:         if (!IPort(_localPortAddress).isBridgeAgent(newBridgeAgent)) {

 /// @audit  isBridgeAgentFactory()
 249:         if (IPort(_localPortAddress).isBridgeAgentFactory(_newBridgeAgentFactoryAddress)) {

 /// @audit  toggleBridgeAgentFactory()
 251:             IPort(_localPortAddress).toggleBridgeAgentFactory(_newBridgeAgentFactoryAddress);

 /// @audit  addBridgeAgentFactory()
 254:             IPort(_localPortAddress).addBridgeAgentFactory(_newBridgeAgentFactoryAddress);

 /// @audit  isBridgeAgent()
 268:         if (!IPort(_localPortAddress).isBridgeAgent(_branchBridgeAgent)) revert UnrecognizedBridgeAgent();

 /// @audit  toggleBridgeAgent()
 271:         IPort(_localPortAddress).toggleBridgeAgent(_branchBridgeAgent);

 /// @audit  isStrategyToken()
 289:         if (IPort(_localPortAddress).isStrategyToken(_underlyingToken)) {

 /// @audit  toggleStrategyToken()
 291:             IPort(_localPortAddress).toggleStrategyToken(_underlyingToken);

 /// @audit  addStrategyToken()
 294:             IPort(_localPortAddress).addStrategyToken(_underlyingToken, _minimumReservesRatio);

 /// @audit  isPortStrategy()
 317:         if (!IPort(_localPortAddress).isPortStrategy(_portStrategy, _underlyingToken)) {

 /// @audit  addPortStrategy()
 319:             IPort(_localPortAddress).addPortStrategy(_portStrategy, _underlyingToken, _dailyManagementLimit);

 /// @audit  updatePortStrategy()
 322:             IPort(_localPortAddress).updatePortStrategy(_portStrategy, _underlyingToken, _dailyManagementLimit);

 /// @audit  togglePortStrategy()
 325:             IPort(_localPortAddress).togglePortStrategy(_portStrategy, _underlyingToken);

```
https://github.com/code-423n4/2023-09-maia/blob/6c21f6b739252367c5fe770fed3ff76eb3ca2c86/src/CoreBranchRouter.sol#143


```solidity
File: src/CoreRootRouter.sol

 /// @audit  isChainId()
 117:         if (!IPort(rootPortAddress).isChainId(_dstChainId)) revert InvalidChainId();

 /// @audit  isBranchBridgeAgentAllowed()
 123:         if (!IBridgeAgent(_rootBridgeAgent).isBranchBridgeAgentAllowed(_dstChainId)) revert UnauthorizedChainId();

 /// @audit  isBridgeAgentFactory()
 163:         if (!IPort(rootPortAddress).isBridgeAgentFactory(_rootBridgeAgentFactory)) {

 /// @audit  isGlobalAddress()
 414:         if (!IPort(rootPortAddress).isGlobalAddress(_globalAddress)) {

 /// @audit  isGlobalToken()
 419:         if (IPort(rootPortAddress).isGlobalToken(_globalAddress, _dstChainId)) {


 /// @audit  isGlobalAddress()
 461:         if (IPort(rootPortAddress).isGlobalAddress(_underlyingAddress)) revert TokenAlreadyAdded();

 /// @audit  isLocalToken()
 462:         if (IPort(rootPortAddress).isLocalToken(_underlyingAddress, _srcChainId)) revert TokenAlreadyAdded();

 /// @audit  isUnderlyingToken()
 463:         if (IPort(rootPortAddress).isUnderlyingToken(_underlyingAddress, _srcChainId)) revert TokenAlreadyAdded();

 /// @audit  setAddresses()
 469:         IPort(rootPortAddress).setAddresses(

 /// @audit  isLocalToken()
 483:         if (IPort(rootPortAddress).isLocalToken(_localAddress, _dstChainId)) revert TokenAlreadyAdded();

 /// @audit  setLocalAddress()
 486:         IPort(rootPortAddress).setLocalAddress(_globalAddress, _localAddress, _dstChainId);

 /// @audit  syncBranchBridgeAgentWithRoot()
 503:         IPort(rootPortAddress).syncBranchBridgeAgentWithRoot(_newBranchBridgeAgent, _rootBridgeAgent, _srcChainId);

```
https://github.com/code-423n4/2023-09-maia/blob/6c21f6b739252367c5fe770fed3ff76eb3ca2c86/src/CoreRootRouter.sol#117


```solidity
File: src/MulticallRootRouter.sol

 /// @audit  call()
 239:             IVirtualAccount(userAccount).call(calls);

 /// @audit  call()
 248:             IVirtualAccount(userAccount).call(calls);

 /// @audit  withdrawERC20()
 251:             IVirtualAccount(userAccount).withdrawERC20(outputParams.outputToken, outputParams.amountOut);

 /// @audit  userAddress()
 255:                 IVirtualAccount(userAccount).userAddress(),

 /// @audit  call()
 275:             IVirtualAccount(userAccount).call(calls);

 /// @audit  withdrawERC20()
 279:                 IVirtualAccount(userAccount).withdrawERC20(outputParams.outputTokens[i], outputParams.amountsOut[i]);

 /// @audit  userAddress()
 288:                 IVirtualAccount(userAccount).userAddress(),

 /// @audit  call()
 328:             IVirtualAccount(userAccount).call(calls);

 /// @audit  call()
 337:             IVirtualAccount(userAccount).call(calls);

 /// @audit  withdrawERC20()
 340:             IVirtualAccount(userAccount).withdrawERC20(outputParams.outputToken, outputParams.amountOut);

 /// @audit  userAddress()
 344:                 IVirtualAccount(userAccount).userAddress(),

 /// @audit  call()
 364:             IVirtualAccount(userAccount).call(calls);

 /// @audit  withdrawERC20()
 368:                 IVirtualAccount(userAccount).withdrawERC20(outputParams.outputTokens[i], outputParams.amountsOut[i]);

 /// @audit  userAddress()
 377:                 IVirtualAccount(userAccount).userAddress(),

 /// @audit  call()
 416:             IVirtualAccount(userAccount).call(calls);

 /// @audit  call()
 425:             IVirtualAccount(userAccount).call(calls);

 /// @audit  withdrawERC20()
 428:             IVirtualAccount(userAccount).withdrawERC20(outputParams.outputToken, outputParams.amountOut);

 /// @audit  userAddress()
 432:                 IVirtualAccount(userAccount).userAddress(),

 /// @audit  call()
 452:             IVirtualAccount(userAccount).call(calls);

 /// @audit  withdrawERC20()
 456:                 IVirtualAccount(userAccount).withdrawERC20(outputParams.outputTokens[i], outputParams.amountsOut[i]);

 /// @audit  userAddress()
 465:                 IVirtualAccount(userAccount).userAddress(),

```
https://github.com/code-423n4/2023-09-maia/blob/6c21f6b739252367c5fe770fed3ff76eb3ca2c86/src/MulticallRootRouter.sol#239


```solidity
File: src/RootBridgeAgent.sol


 /// @audit  bridgeToRoot()
 328:                 IPort(localPortAddress).bridgeToRoot(

 /// @audit  getGlobalTokenFromLocal()
 330:                     IPort(localPortAddress).getGlobalTokenFromLocal(_hToken, _dstChainId),

 /// @audit  isLocalToken()
 364:             if (!IPort(_localPortAddress).isLocalToken(_dParams.hToken, _srcChainId)) {

 /// @audit  bridgeToRoot()
 377:         IPort(_localPortAddress).bridgeToRoot(

 /// @audit  getGlobalTokenFromLocal()
 379:             IPort(_localPortAddress).getGlobalTokenFromLocal(_dParams.hToken, _srcChainId),


 /// @audit  toggleVirtualAccountApproved()
 554:             IPort(localPortAddress).toggleVirtualAccountApproved(userAccount, localRouterAddress);

 /// @audit  toggleVirtualAccountApproved()
 574:             IPort(localPortAddress).toggleVirtualAccountApproved(userAccount, localRouterAddress);

 /// @audit  toggleVirtualAccountApproved()
 592:             IPort(localPortAddress).toggleVirtualAccountApproved(userAccount, localRouterAddress);

 /// @audit  toggleVirtualAccountApproved()
 614:             IPort(localPortAddress).toggleVirtualAccountApproved(userAccount, localRouterAddress);

 /// @audit  toggleVirtualAccountApproved()
 632:             IPort(localPortAddress).toggleVirtualAccountApproved(userAccount, localRouterAddress);

 /// @audit  toggleVirtualAccountApproved()
 654:             IPort(localPortAddress).toggleVirtualAccountApproved(userAccount, localRouterAddress);

 /// @audit  lzReceive()
 837:             IBranchBridgeAgent(callee).lzReceive(0, "", 0, _payload);

 /// @audit  lzReceive()
 929:             IBranchBridgeAgent(callee).lzReceive(0, "", 0, payload);

 /// @audit  burn()
 1161:             IPort(localPortAddress).burn(_depositor, _globalAddress, _deposit, _dstChainId);

```
https://github.com/code-423n4/2023-09-maia/blob/6c21f6b739252367c5fe770fed3ff76eb3ca2c86/src/RootBridgeAgent.sol#328


```solidity
File: src/RootBridgeAgentExecutor.sol

 /// @audit  executeResponse()
 56:         IRouter(_router).executeResponse(_payload[PARAMS_TKN_START:], _srcChainId);

 /// @audit  bridgeIn()
 245:         RootBridgeAgent(payable(msg.sender)).bridgeIn(_recipient, _dParams, _srcChainId);

 /// @audit  bridgeInMultiple()
 347:         RootBridgeAgent(payable(msg.sender)).bridgeInMultiple(_recipient, dParams, _srcChainId);

```
https://github.com/code-423n4/2023-09-maia/blob/6c21f6b739252367c5fe770fed3ff76eb3ca2c86/src/RootBridgeAgentExecutor.sol#56


```solidity
File: src/RootPort.sol

 /// @audit  syncBranchBridgeAgent()
 161:         IBridgeAgent(_coreRootBridgeAgent).syncBranchBridgeAgent(_coreLocalBranchBridgeAgent, localChainId);

 /// @audit  mint()
 290:         if (_deposit > 0) if (!ERC20hTokenRoot(_hToken).mint(_recipient, _deposit, _srcChainId)) revert UnableToMint();

 /// @audit  burn()
 321:         ERC20hTokenRoot(_hToken).burn(_from, _amount, _srcChainId);

 /// @audit  burn()
 332:         ERC20hTokenRoot(_hToken).burn(_from, _amount, localChainId);

 /// @audit  mint()
 342:         if (!ERC20hTokenRoot(_hToken).mint(_to, _amount, localChainId)) revert UnableToMint();

 /// @audit  isBranchBridgeAgentAllowed()
 401:         if (!IBridgeAgent(_rootBridgeAgent).isBranchBridgeAgentAllowed(_branchChainId)) {

 /// @audit  syncBranchBridgeAgent()
 404:         IBridgeAgent(_rootBridgeAgent).syncBranchBridgeAgent(_newBranchBridgeAgent, _branchChainId);

 /// @audit  hTokenFactoryAddress()
 452:             IERC20hTokenRootFactory(ICoreRootRouter(coreRootRouterAddress).hTokenFactoryAddress()).createToken(

 /// @audit  syncBranchBridgeAgent()
 458:         IBridgeAgent(ICoreRootRouter(coreRootRouterAddress).bridgeAgentAddress()).syncBranchBridgeAgent(

 /// @audit  bridgeAgentAddress()
 458:         IBridgeAgent(ICoreRootRouter(coreRootRouterAddress).bridgeAgentAddress()).syncBranchBridgeAgent(

 /// @audit  syncBranchBridgeAgent()
 547:         IBridgeAgent(coreRootBridgeAgentAddress).syncBranchBridgeAgent(_coreBranchBridgeAgent, _dstChainId);

```
https://github.com/code-423n4/2023-09-maia/blob/6c21f6b739252367c5fe770fed3ff76eb3ca2c86/src/RootPort.sol#161


```solidity
File: src/VirtualAccount.sol

 /// @audit  transferFrom()
 70:         ERC721(_token).transferFrom(address(this), msg.sender, _tokenId);

 /// @audit  isRouterApproved()
 189:         if (!IRootPort(localPortAddress).isRouterApproved(this, msg.sender)) {

```
https://github.com/code-423n4/2023-09-maia/blob/6c21f6b739252367c5fe770fed3ff76eb3ca2c86/src/VirtualAccount.sol#70


```solidity
File: src/factories/ArbitrumBranchBridgeAgentFactory.sol

 /// @audit  addBridgeAgent()
 65:         IPort(localPortAddress).addBridgeAgent(newCoreBridgeAgent);

 /// @audit  addBridgeAgent()
 98:         IPort(localPortAddress).addBridgeAgent(newBridgeAgent);

```
https://github.com/code-423n4/2023-09-maia/blob/6c21f6b739252367c5fe770fed3ff76eb3ca2c86/src/factories/ArbitrumBranchBridgeAgentFactory.sol#65


```solidity
File: src/factories/BranchBridgeAgentFactory.sol

 /// @audit  addBridgeAgent()
 101:         IPort(localPortAddress).addBridgeAgent(newCoreBridgeAgent);

 /// @audit  addBridgeAgent()
 139:         IPort(localPortAddress).addBridgeAgent(newBridgeAgent);

```

```solidity
File: src/factories/RootBridgeAgentFactory.sol

 /// @audit  addBridgeAgent()
 58:         IRootPort(rootPortAddress).addBridgeAgent(msg.sender, newBridgeAgent);

```


## [G-3] Avoid updating storage when the value hasn’t changed
If the old value is equal to the new value, not re-storing the value will avoid a Gsreset (2900 gas), potentially at the expense of a Gcoldsload (2100 gas) or a Gwarmaccess (100 gas)

Note: instaces below were missed by bot.


 ```solidity
File: src/ArbitrumBranchPort.sol

 41:         localChainId = _localChainId;

 42:         rootPortAddress = _rootPortAddress;

```

```solidity
File: src/BaseBranchRouter.sol

 62:         localBridgeAgentAddress = _localBridgeAgentAddress;

 63:         localPortAddress = IBridgeAgent(_localBridgeAgentAddress).localPortAddress();

 64:         bridgeAgentExecutorAddress = IBridgeAgent(_localBridgeAgentAddress).bridgeAgentExecutorAddress();

```

```solidity
File: src/BranchBridgeAgent.sol

 136:         rootChainId = _rootChainId;

 137:         rootBridgeAgentAddress = _rootBridgeAgentAddress;

 138:         lzEndpointAddress = _lzEndpointAddress;

 139:         localRouterAddress = _localRouterAddress;

 140:         localPortAddress = _localPortAddress;

 141:         bridgeAgentExecutorAddress = DeployBranchBridgeAgentExecutor.deploy();

 142:         depositNonce = 1;

 144:         rootBridgeAgentPath = abi.encodePacked(

 818:                     executionState[nonce] = STATUS_RETRIEVE;

 829:             getDeposit[nonce].status = STATUS_FAILED;

 860:         executionState[_settlementNonce] = STATUS_DONE;

 885:         executionState[_settlementNonce] = STATUS_DONE;

 897:                 executionState[_settlementNonce] = STATUS_RETRIEVE;

 985:         depositNonce = _depositNonce + 1;

 1045:         depositNonce = _depositNonce + 1;

```

```solidity
File: src/BranchPort.sol

 130:         isBridgeAgentFactory[_bridgeAgentFactory] = true;

 155:         getStrategyTokenDebt[_token] = _strategyTokenDebt + _amount;

 207:         getStrategyTokenDebt[_token] = strategyTokenDebt - amountToWithdraw;

 322:         isBridgeAgent[_bridgeAgent] = true;

 341:         isBridgeAgentFactory[_newBridgeAgentFactory] = true;

 349:         isBridgeAgentFactory[_newBridgeAgentFactory] = !isBridgeAgentFactory[_newBridgeAgentFactory];

 356:         isBridgeAgent[_bridgeAgent] = !isBridgeAgent[_bridgeAgent];

 376:         isStrategyToken[_token] = !isStrategyToken[_token];

```

```solidity
File: src/CoreBranchRouter.sol

 31:         hTokenFactoryAddress = _hTokenFactoryAddress;

```

```solidity
File: src/CoreRootRouter.sol

 72:         rootChainId = _rootChainId;

 73:         rootPortAddress = _rootPortAddress;

 76:         _setup = true;

 85:         _setup = false;

 86:         bridgeAgentAddress = payable(_bridgeAgentAddress);

 87:         bridgeAgentExecutorAddress = IBridgeAgent(_bridgeAgentAddress).bridgeAgentExecutorAddress();

 88:         hTokenFactoryAddress = _hTokenFactory;

```

```solidity
File: src/MulticallRootRouter.sol

 96:         localChainId = _localChainId;

 97:         localPortAddress = _localPortAddress;

 98:         multicallAddress = _multicallAddress;

 112:         bridgeAgentAddress = payable(_bridgeAgentAddress);

 113:         bridgeAgentExecutorAddress = IBridgeAgent(_bridgeAgentAddress).bridgeAgentExecutorAddress();

```

```solidity
File: src/RootBridgeAgent.sol

 115:         factoryAddress = msg.sender;

 116:         localChainId = _localChainId;

 117:         lzEndpointAddress = _lzEndpointAddress;

 118:         localPortAddress = _localPortAddress;

 119:         localRouterAddress = _localRouterAddress;

 120:         bridgeAgentExecutorAddress = DeployRootBridgeAgentExecutor.deploy(address(this));

 121:         settlementNonce = 1;

 723:             getSettlement[nonce].status = STATUS_FAILED;

 978:         settlementNonce = _settlementNonce + 1;

 1064:         settlementNonce = _settlementNonce + 1;

 1172:         isBranchBridgeAgentAllowed[_branchChainId] = true;

 1181:         getBranchBridgeAgent[_branchChainId] = _newBranchBridgeAgent;

 1182:         getBranchBridgeAgentPath[_branchChainId] = abi.encodePacked(_newBranchBridgeAgent, address(this));

```

```solidity
File: src/RootPort.sol

 112:         localChainId = _localChainId;

 113:         isChainId[_localChainId] = true;

 116:         _setup = true;

 117:         _setupCore = true;

 133:         _setup = false;

 135:         isBridgeAgentFactory[_bridgeAgentFactory] = true;

 138:         coreRootRouterAddress = _coreRootRouter;

 157:         _setupCore = false;

 159:         coreRootBridgeAgentAddress = _coreRootBridgeAgent;

 160:         localBranchPortAddress = _localBranchPortAddress;

 162:         getBridgeAgentManager[_coreRootBridgeAgent] = owner();

 363:         getUserAccount[_user] = newAccount;

 415:         isBridgeAgent[_bridgeAgent] = !isBridgeAgent[_bridgeAgent];

 425:         isBridgeAgentFactory[_bridgeAgentFactory] = true;

 432:         isBridgeAgentFactory[_bridgeAgentFactory] = !isBridgeAgentFactory[_bridgeAgentFactory];


```

```solidity
File: src/VirtualAccount.sol

 36:         userAddress = _userAddress;

 37:         localPortAddress = _localPortAddress;

```

```solidity
File: src/factories/BranchBridgeAgentFactory.sol

 71:         rootChainId = _rootChainId;

 72:         rootBridgeAgentFactoryAddress = _rootBridgeAgentFactoryAddress;

 73:         lzEndpointAddress = _lzEndpointAddress;

 74:         localCoreBranchRouterAddress = _localCoreBranchRouterAddress;

 75:         localPortAddress = _localPortAddress;

```

```solidity
File: src/factories/ERC20hTokenBranchFactory.sol

 44:         chainName = string.concat(_chainName, " Ulysses");

 45:         chainSymbol = string.concat(_chainSymbol, "-u");

 46:         localChainId = _localChainId;

 47:         localPortAddress = _localPortAddress;

 74:         localCoreRouterAddress = _coreRouter;

```

```solidity
File: src/factories/ERC20hTokenRootFactory.sol

 36:         localChainId = _localChainId;

 37:         rootPortAddress = _rootPortAddress;

 51:         coreRootRouterAddress = _coreRouter;

```

```solidity
File: src/factories/RootBridgeAgentFactory.sol

 34:         rootChainId = _rootChainId;

 35:         lzEndpointAddress = _lzEndpointAddress;

 36:         rootPortAddress = _rootPortAddress;

```

```solidity
File: src/token/ERC20hTokenRoot.sol

 42:         localChainId = _localChainId;

 43:         factoryAddress = _factoryAddress;

 58:         getTokenBalance[chainId] += amount;

 70:         getTokenBalance[chainId] -= amount;

```


## [G-4] Multiple accesses of the same mapping/array key/index should be cached
The instances below point to the second+ access of a value inside a mapping/array, within a function. Caching a mapping’s value in a local storage or calldata variable when the value is accessed multiple times, saves ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations. Caching an array’s struct avoids recalculating the array offsets into memory/calldata


 ```solidity
File: src/ArbitrumCoreBranchRouter.sol

 173:         } else if (_data[0] == 0x03) {

```

```solidity
File: src/BranchBridgeAgent.sol

 521:         delete getDeposit[_depositNonce];

 1007:         addressArray[0] = _token;

 1013:         uintArray[0] = _deposit;

```

```solidity
File: src/BranchPort.sol

 124:         require(!isBridgeAgentFactory[_bridgeAgentFactory], "Contract already initialized");

 205:         getPortStrategyTokenDebt[_strategy][_token] = portStrategyTokenDebt - amountToWithdraw;

 259:             if (_amounts[i] - _deposits[i] > 0) {

 261:                     _bridgeIn(_recipient, _localAddresses[i], _amounts[i] - _deposits[i]);

 397:         isPortStrategy[_portStrategy][_token] = !isPortStrategy[_portStrategy][_token];

 490:                 lastManaged[msg.sender][_token] = (block.timestamp / 1 days) * 1 days;

 493:         strategyDailyLimitRemaining[msg.sender][_token] = dailyLimit - _amount;

```

```solidity
File: src/CoreBranchRouter.sol

 100:         } else if (_params[0] == 0x02) {

```

```solidity
File: src/RootBridgeAgent.sol

 343:         delete getSettlement[_settlementNonce];

 709:                     executionState[_srcChainId][nonce] = STATUS_RETRIEVE;

 786:                 executionState[_srcChainId][_depositNonce] = STATUS_RETRIEVE;

 1020:         addressArray[0] = underlyingAddress;

 1026:         uintArray[0] = _deposit;

 1072:             hTokens[i] = IPort(localPortAddress).getLocalTokenFromGlobal(_globalAddresses[i], _dstChainId);

 1080:                 msg.sender, _globalAddresses[i], hTokens[i], tokens[i], _amounts[i], _deposits[i], destChainId

 1080:                 msg.sender, _globalAddresses[i], hTokens[i], tokens[i], _amounts[i], _deposits[i], destChainId

```

```solidity
File: src/RootBridgeAgentExecutor.sol

 125:                     PARAMS_END_OFFSET + uint256(uint8(bytes1(_payload[PARAMS_START]))) * PARAMS_TKN_SET_SIZE_MULTIPLE

 213:                         + uint256(uint8(bytes1(_payload[PARAMS_START_SIGNED]))) * PARAMS_TKN_SET_SIZE_MULTIPLE

```

```solidity
File: src/RootPort.sol

 374:         isRouterApproved[_userAccount][_router] = !isRouterApproved[_userAccount][_router];

```
 

## [G-5]  The result of a function call should be cached rather than re-calling the function
The function calls in solidity are expensive. If the same result of the same function calls are to be used several times, the result should be cached to reduce the gas consumption of repeated calls.


 ```solidity
File: src/ArbitrumBranchBridgeAgent.sol

 127:         if (_endpoint != rootBridgeAgentAddress) revert LayerZeroUnauthorizedEndpoint();

```

```solidity
File: src/ArbitrumBranchPort.sol

 72:         IRootPort(_rootPortAddress).mintToLocalBranch(_recipient, _globalToken, _deposit);

 97:         IRootPort(_rootPortAddress).burnFromLocalBranch(_depositor, _globalAddress, _amount);

```

```solidity
File: src/ArbitrumCoreBranchRouter.sol

 61:                 ERC20(_underlyingAddress).name()

 124:         if (!IPort(_localPortAddress).isBridgeAgent(newBridgeAgent)) {

 174:             address bridgeAgentFactoryAddress = abi.decode(

```

```solidity
File: src/BranchBridgeAgent.sol

 499:         if (deposit.owner == address(0)) revert DepositRedeemUnavailable();

 750:                 revert AlreadyExecutedTransaction();

 784:             _execute(

 788:                 abi.encodeWithSelector(

 1040:         if (_hTokens.length != _tokens.length) revert InvalidInput();

 1088:                 IPort(localPortAddress).bridgeIn(

 1127:             revert LayerZeroUnauthorizedEndpoint();

 1134:         ) revert LayerZeroUnauthorizedCaller();

```

```solidity
File: src/BranchPort.sol

 300:         if (length != _underlyingAddresses.length) revert InvalidInputArrays();

```

```solidity
File: src/CoreBranchRouter.sol

 68:             ERC20(_underlyingAddress).name(), ERC20(_underlyingAddress).symbol(), decimals, true

 108:             ) = abi.decode(_params[1:], (address, address, address, address, address, GasParams));

 221:         if (!IPort(_localPortAddress).isBridgeAgent(newBridgeAgent)) {

 254:             IPort(_localPortAddress).addBridgeAgentFactory(_newBridgeAgentFactoryAddress);

 268:         if (!IPort(_localPortAddress).isBridgeAgent(_branchBridgeAgent)) revert UnrecognizedBridgeAgent();

 294:             IPort(_localPortAddress).addStrategyToken(_underlyingToken, _minimumReservesRatio);

 319:             IPort(_localPortAddress).addPortStrategy(_portStrategy, _underlyingToken, _dailyManagementLimit);

```

```solidity
File: src/CoreRootRouter.sol

 117:         if (!IPort(rootPortAddress).isChainId(_dstChainId)) revert InvalidChainId();

 120:         if (IBridgeAgent(_rootBridgeAgent).getBranchBridgeAgent(_dstChainId) != address(0)) revert InvalidChainId();

 123:         if (!IBridgeAgent(_rootBridgeAgent).isBranchBridgeAgentAllowed(_dstChainId)) revert UnauthorizedChainId();

 315:             (address globalAddress, address localAddress) = abi.decode(_encodedData[1:], (address, address));

 414:         if (!IPort(rootPortAddress).isGlobalAddress(_globalAddress)) {

 427:             ERC20(_globalAddress).symbol(),

 462:         if (IPort(rootPortAddress).isLocalToken(_underlyingAddress, _srcChainId)) revert TokenAlreadyAdded();

 462:         if (IPort(rootPortAddress).isLocalToken(_underlyingAddress, _srcChainId)) revert TokenAlreadyAdded();

 486:         IPort(rootPortAddress).setLocalAddress(_globalAddress, _localAddress, _dstChainId);

```

```solidity
File: src/MulticallRootRouter.sol

 157:             ) = abi.decode(_decode(encodedData[1:]), (IMulticall.Call[], OutputParams, uint16, GasParams));

 157:             ) = abi.decode(_decode(encodedData[1:]), (IMulticall.Call[], OutputParams, uint16, GasParams));

 160:             _multicall(callData);

 245:                 abi.decode(_decode(encodedData[1:]), (Call[], OutputParams, uint16, GasParams));

 245:                 abi.decode(_decode(encodedData[1:]), (Call[], OutputParams, uint16, GasParams));

 248:             IVirtualAccount(userAccount).call(calls);

 334:                 abi.decode(_decode(encodedData[1:]), (Call[], OutputParams, uint16, GasParams));

 334:                 abi.decode(_decode(encodedData[1:]), (Call[], OutputParams, uint16, GasParams));

 337:             IVirtualAccount(userAccount).call(calls);

 422:                 abi.decode(_decode(encodedData[1:]), (Call[], OutputParams, uint16, GasParams));

 422:                 abi.decode(_decode(encodedData[1:]), (Call[], OutputParams, uint16, GasParams));

 425:             IVirtualAccount(userAccount).call(calls);

```

```solidity
File: src/RootBridgeAgent.sol

 308:         if (settlementOwner == address(0)) revert SettlementRedeemUnavailable();

 328:                 IPort(localPortAddress).bridgeToRoot(

 365:                 revert InvalidInputParams();

 379:             IPort(_localPortAddress).getGlobalTokenFromLocal(_dParams.hToken, _srcChainId),

 473:                 revert AlreadyExecutedTransaction();

 481:             _execute(

 483:                 abi.encodeWithSelector(

 554:             IPort(localPortAddress).toggleVirtualAccountApproved(userAccount, localRouterAddress);

 603:                 abi.encodeWithSelector(

 639:             _execute(

 984:         address underlyingAddress = IPort(localPortAddress).getUnderlyingTokenFromLocal(localAddress, _dstChainId);

 1060:         if (_globalAddresses.length != _amounts.length) revert InvalidInputParamsLength();

 1073:             tokens[i] = IPort(localPortAddress).getUnderlyingTokenFromLocal(hTokens[i], _dstChainId);

 1142:         if (_amount < _deposit) revert InvalidInputParams();

```

```solidity
File: src/RootPort.sol

 398:         if (IBridgeAgent(_rootBridgeAgent).getBranchBridgeAgent(_branchChainId) != address(0)) {

 452:             IERC20hTokenRootFactory(ICoreRootRouter(coreRootRouterAddress).hTokenFactoryAddress()).createToken(

 489:             revert AlreadyAddedEcosystemToken();

```

```solidity
File: src/VirtualAccount.sol

 124:             if (!success) revert CallFailed();

```

```solidity
File: src/factories/ERC20hTokenBranchFactory.sol

 67:             ERC20(_wrappedNativeTokenAddress).symbol(),

```


## [G-6]  State variables that are used multiple times in a function should be cached in stack variables
When performing multiple operations on a state variable in a function, it is recommended to cache it first. Either multiple reads or multiple writes to a state variable can save gas by caching it on the stack. Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses. Saves 100 gas per instance.


 ```solidity
File: src/ArbitrumBranchPort.sol

 86:         if (!IRootPort(_rootPortAddress).isGlobalToken(_globalAddress, localChainId)) revert UnknownGlobalToken();

```

```solidity
File: src/BranchBridgeAgent.sol

 521:         delete getDeposit[_depositNonce];

 793:                     localRouterAddress,

 1088:                 IPort(localPortAddress).bridgeIn(

```

```solidity
File: src/BranchPort.sol

 124:         require(!isBridgeAgentFactory[_bridgeAgentFactory], "Contract already initialized");

 205:         getPortStrategyTokenDebt[_strategy][_token] = portStrategyTokenDebt - amountToWithdraw;

 397:         isPortStrategy[_portStrategy][_token] = !isPortStrategy[_portStrategy][_token];

 490:                 lastManaged[msg.sender][_token] = (block.timestamp / 1 days) * 1 days;

 493:         strategyDailyLimitRemaining[msg.sender][_token] = dailyLimit - _amount;

```

```solidity
File: src/CoreRootRouter.sol

 117:         if (!IPort(rootPortAddress).isChainId(_dstChainId)) revert InvalidChainId();

 414:         if (!IPort(rootPortAddress).isGlobalAddress(_globalAddress)) {

 469:         IPort(rootPortAddress).setAddresses(

 486:         IPort(rootPortAddress).setLocalAddress(_globalAddress, _localAddress, _dstChainId);

```

```solidity
File: src/RootBridgeAgent.sol

 330:                     IPort(localPortAddress).getGlobalTokenFromLocal(_hToken, _dstChainId),

 343:         delete getSettlement[_settlementNonce];

 646:                     localRouterAddress,

 709:                     executionState[_srcChainId][nonce] = STATUS_RETRIEVE;

 786:                 executionState[_srcChainId][_depositNonce] = STATUS_RETRIEVE;

 984:         address underlyingAddress = IPort(localPortAddress).getUnderlyingTokenFromLocal(localAddress, _dstChainId);

 1073:             tokens[i] = IPort(localPortAddress).getUnderlyingTokenFromLocal(hTokens[i], _dstChainId);

 1161:             IPort(localPortAddress).burn(_depositor, _globalAddress, _deposit, _dstChainId);

```

```solidity
File: src/RootPort.sol

 374:         isRouterApproved[_userAccount][_router] = !isRouterApproved[_userAccount][_router];

 503:         getLocalTokenFromGlobal[_ecoTokenGlobalAddress][localChainId] = _ecoTokenGlobalAddress;

```

```solidity
File: src/factories/BranchBridgeAgentFactory.sol

 101:         IPort(localPortAddress).addBridgeAgent(newCoreBridgeAgent);

 139:         IPort(localPortAddress).addBridgeAgent(newBridgeAgent);

```

```solidity
File: src/factories/ERC20hTokenBranchFactory.sol

 102:             ? new ERC20hTokenBranch(chainName, chainSymbol, _name, _symbol, _decimals, localPortAddress)

```

```solidity
File: src/factories/RootBridgeAgentFactory.sol

 58:         IRootPort(rootPortAddress).addBridgeAgent(msg.sender, newBridgeAgent);

```


## [G-7] Gas savings can be achieved by changing the model for assigning value to the structure
By changing the pattern of assigning value to the structure, gas savings of ~130 per instance are achieved. In addition, this use will provide significant savings in distribution costs. 
 `structure = Structure({parameter1: value1})` 
 can be changed to 
 `structure.parameter1 = value1`


 ```solidity
File: src/ArbitrumCoreBranchRouter.sol

 74:             GasParams(0, 0)

 169:                 GasParams(0, 0)

```

```solidity
File: src/BranchBridgeAgent.sol

 666:             SettlementMultipleParams(
                     numOfAssets,
                     _recipient,
                     nonce,
                     _hTokens,
                     _tokens,
                     _amounts,
                     _deposits

```

```solidity
File: src/BranchBridgeAgentExecutor.sol

 72:         SettlementParams memory sParams = SettlementParams({
                settlementNonce: uint32(bytes4(_payload[PARAMS_START_SIGNED:PARAMS_TKN_START_SIGNED])),
                recipient: _recipient,
                hToken: address(uint160(bytes20(_payload[PARAMS_TKN_START_SIGNED:45]))),
                token: address(uint160(bytes20(_payload[45:65]))),
                amount: uint256(bytes32(_payload[65:97])),
                deposit: uint256(bytes32(_payload[97:PARAMS_SETTLEMENT_OFFSET]))

```

```solidity
File: src/MulticallRootRouter.sol

 529:             SettlementInput(outputToken, amountOut, depositOut),

 571:             SettlementMultipleInput(outputTokens, amountsOut, depositsOut),

```

```solidity
File: src/RootBridgeAgent.sol

 402:                 DepositParams({
                         hToken: _dParams.hTokens[i],
                         token: _dParams.tokens[i],
                         amount: _dParams.amounts[i],
                         deposit: _dParams.deposits[i],
                         depositNonce: 0

```

```solidity
File: src/RootBridgeAgentExecutor.sol

 88:         DepositParams memory dParams = DepositParams({
                depositNonce: uint32(bytes4(_payload[PARAMS_START:PARAMS_TKN_START])),
                hToken: address(uint160(bytes20(_payload[PARAMS_TKN_START:PARAMS_TKN_START_SIGNED]))),
                token: address(uint160(bytes20(_payload[PARAMS_TKN_START_SIGNED:45]))),
                amount: uint256(bytes32(_payload[45:77])),
                deposit: uint256(bytes32(_payload[77:PARAMS_TKN_SET_SIZE]))

 173:         DepositParams memory dParams = DepositParams({
                 depositNonce: uint32(bytes4(_payload[PARAMS_START_SIGNED:PARAMS_TKN_START_SIGNED])),
                 hToken: address(uint160(bytes20(_payload[PARAMS_TKN_START_SIGNED:45]))),
                 token: address(uint160(bytes20(_payload[45:65]))),
                 amount: uint256(bytes32(_payload[65:97])),
                 deposit: uint256(bytes32(_payload[97:PARAMS_SETTLEMENT_OFFSET]))

 338:         dParams = DepositMultipleParams({
                 numberOfAssets: numOfAssets,
                 depositNonce: nonce,
                 hTokens: hTokens,
                 tokens: tokens,
                 amounts: amounts,
                 deposits: deposits

```


## [G-8] Do not shrink Variables
This means that if you use uint8 or any variable smaller than 256 bits, EVM has to first convert it uint256 to work on it and the conversion costs extra gas! You may wonder, What were the devs thinking? Why did they create smaller variables then? The answer lies in packing. In solidity, you can pack multiple small variables into one slot, but if you are defining a lone variable and can’t pack it, it’s optimal to use a uint256.


 ```solidity
File: src/ArbitrumBranchPort.sol

 22:     uint16 public immutable localChainId;

```

```solidity
File: src/BranchBridgeAgent.sol

 46:     uint16 public immutable rootChainId;

 49:     uint16 public immutable localChainId;

 77:     uint32 public depositNonce;

 233:         uint32 _depositNonce = depositNonce;

 268:         uint32 _depositNonce = depositNonce;

 323:         uint32 _depositNonce = depositNonce;

 360:         uint32 _depositNonce = depositNonce;

 579:         uint8 numOfAssets = uint8(bytes1(_sParams[0]));

 582:         uint32 nonce = uint32(bytes4(_sParams[PARAMS_START:PARAMS_TKN_START]));

 710:         uint32 nonce;

```

```solidity
File: src/CoreBranchRouter.sol

 64:         uint8 decimals = ERC20(_underlyingAddress).decimals();

 93:                 uint8 decimals,

```

```solidity
File: src/CoreRootRouter.sol

 308:             (address underlyingAddress, address localAddress, string memory name, string memory symbol, uint8 decimals)

 338:             (address refundee, address globalAddress, uint16 dstChainId, GasParams[2] memory gasParams) =

```

```solidity
File: src/MulticallRootRouter.sol

 155:                 uint16 dstChainId,

 179:                 uint16 dstChainId,

 244:             (Call[] memory calls, OutputParams memory outputParams, uint16 dstChainId, GasParams memory gasParams) =

 270:                 uint16 dstChainId,

 333:             (Call[] memory calls, OutputParams memory outputParams, uint16 dstChainId, GasParams memory gasParams) =

 359:                 uint16 dstChainId,

 421:             (Call[] memory calls, OutputParams memory outputParams, uint16 dstChainId, GasParams memory gasParams) =

 447:                 uint16 dstChainId,

```

```solidity
File: src/RootBridgeAgent.sol

 40:     uint16 public immutable localChainId;

 75:     uint32 public settlementNonce;

 325:                 uint24 _dstChainId = settlement.dstChainId;

 441:         uint32 nonce;

 454:             uint16 srcChainId = _srcChainId;

 477:             uint16 srcChainId = _srcChainId;

 500:             uint16 srcChainId = _srcChainId;

 523:             uint16 srcChainId = _srcChainId;

 557:             uint16 srcChainId = _srcChainId;

 595:             uint16 srcChainId = _srcChainId;

 635:             uint16 srcChainId = _srcChainId;

 1004:         uint32 cachedSettlementNonce = _settlementNonce;

 1076:             uint16 destChainId = _dstChainId;

```

```solidity
File: src/RootBridgeAgentExecutor.sol

 273:         uint8 numOfAssets = uint8(bytes1(_dParams[0]));

 274:         uint32 nonce = uint32(bytes4(_dParams[PARAMS_START:5]));

```

```solidity
File: src/factories/BranchBridgeAgentFactory.sol

 21:     uint16 public immutable localChainId;

 24:     uint16 public immutable rootChainId;

```

```solidity
File: src/factories/ERC20hTokenBranchFactory.sol

 14:     uint24 public immutable localChainId;

```

```solidity
File: src/factories/ERC20hTokenRootFactory.sol

 14:     uint16 public immutable localChainId;

```

```solidity
File: src/factories/RootBridgeAgentFactory.sol

 13:     uint16 public immutable rootChainId;

```

```solidity
File: src/interfaces/BridgeAgentConstants.sol

 13:     uint8 internal constant STATUS_READY = 0;

 15:     uint8 internal constant STATUS_DONE = 1;

 17:     uint8 internal constant STATUS_RETRIEVE = 2;

 21:     uint8 internal constant STATUS_FAILED = 1;

 23:     uint8 internal constant STATUS_SUCCESS = 0;

```

```solidity
File: src/token/ERC20hTokenRoot.sol

 14:     uint16 public immutable override localChainId;

```


## [G-9] Empty blocks should be removed or emit something`
 Removing empty blocks help conserve computational resources on the blockchain network, reducing transaction costs and improving overall efficiency.


 ```solidity
File: src/ArbitrumBranchBridgeAgent.sol

 57:     {}

 89:     function retrySettlement(uint32, bytes calldata, GasParams[2] calldata, bool) external payable override lock {}

```

```solidity
File: src/ArbitrumCoreBranchRouter.sol

 44:     constructor() CoreBranchRouter(address(0)) {}

```

```solidity
File: src/BranchBridgeAgent.sol

 154:     receive() external payable {}

```

```solidity
File: src/MulticallRootRouterLibZip.sol

 31:     {}

```

```solidity
File: src/RootBridgeAgent.sol

 128:     receive() external payable {}

```

```solidity
File: src/VirtualAccount.sol

 44:     receive() external payable {}

```

```solidity
File: src/factories/ArbitrumBranchBridgeAgentFactory.sol

 46:     {}

```

## [G-10] Don't initialize variables with default value


 ```solidity
File: src/BaseBranchRouter.sol

 192:         for (uint256 i = 0; i < _hTokens.length;) {

```

```solidity
File: src/BranchBridgeAgent.sol

 506:         for (uint256 i = 0; i < deposit.tokens.length; ) {

 591:         for (uint256 i = 0; i < numOfAssets; ) {

```

```solidity
File: src/BranchPort.sol

 257:         for (uint256 i = 0; i < length;) {

 305:         for (uint256 i = 0; i < length;) {

```

```solidity
File: src/MulticallRootRouter.sol

 278:             for (uint256 i = 0; i < outputParams.outputTokens.length;) {

 367:             for (uint256 i = 0; i < outputParams.outputTokens.length;) {

 455:             for (uint256 i = 0; i < outputParams.outputTokens.length;) {

 557:         for (uint256 i = 0; i < outputTokens.length;) {

```

```solidity
File: src/RootBridgeAgent.sol

 318:         for (uint256 i = 0; i < settlement.hTokens.length;) {

 399:         for (uint256 i = 0; i < length;) {

 1070:         for (uint256 i = 0; i < hTokens.length;) {

```

```solidity
File: src/RootBridgeAgentExecutor.sol

 281:         for (uint256 i = 0; i < uint256(uint8(numOfAssets));) {

```

```solidity
File: src/VirtualAccount.sol

 85:         for (uint256 i = 0; i < length; ) {

 108:         for (uint256 i = 0; i < length; ) {

```

```solidity
File: src/interfaces/BridgeAgentConstants.sol

 13:     uint8 internal constant STATUS_READY = 0;

 23:     uint8 internal constant STATUS_SUCCESS = 0;

```


## [G-11] A modifier used only once and not being inherited should be inlined to save gas
When you use a modifier in Solidity, Solidity generates code to check the conditions of the modifier and execute the modified function if the conditions are met. This generated code can consume gas, especially if the modifier is used frequently or if the modified function is called multiple times. By inlining a modifier that is used only once and not being inherited, you can eliminate the overhead of the generated code and reduce the gas cost of your contract.


 ```solidity
File: src/BranchBridgeAgent.sol

 1114:     modifier requiresEndpoint(address _endpoint, bytes calldata _srcAddress) {

 1138:     modifier requiresRouter() {

```

```solidity
File: src/BranchPort.sol

 553:     modifier requiresBridgeAgentFactory() {

 559:     modifier requiresPortStrategy(address _token) {

```

```solidity
File: src/RootBridgeAgent.sol

 1204:     modifier requiresEndpoint(address _endpoint, uint16 _srcChain, bytes calldata _srcAddress) virtual {

 1226:     modifier requiresPort() {

 1232:     modifier requiresManager() {

```

```solidity
File: src/RootPort.sol

 557:     modifier requiresBridgeAgentFactory() {

```

```solidity
File: src/factories/ERC20hTokenBranchFactory.sol

 112:     modifier requiresCoreRouter() {

```

```solidity
File: src/factories/ERC20hTokenRootFactory.sol

 96:     modifier requiresCoreRouterOrPort() {

```


## [G-12] Usage of ints/uints smaller than 32 bytes incurs overhead
Using ints/uints smaller than 32 bytes may cost more gas. This is because the EVM operates on 32 bytes at a time, so if an element is smaller than 32 bytes, the EVM must perform more operations to reduce the size of the element from 32 bytes to the desired size.


 ```solidity
File: src/ArbitrumBranchBridgeAgent.sol

 11:     function deploy(uint16 _localChainId, address _daoAddress, address _localRouterAddress, address _localPortAddress)

 44:         uint16 _localChainId,

 89:     function retrySettlement(uint32, bytes calldata, GasParams[2] calldata, bool) external payable override lock {}

 112:     function _performFallbackCall(address payable, uint32 _settlementNonce) internal override {

```

```solidity
File: src/ArbitrumBranchPort.sol

 22:     uint16 public immutable localChainId;

 38:     constructor(uint16 _localChainId, address _rootPortAddress, address _owner) BranchPort(_owner) {

```

```solidity
File: src/BaseBranchRouter.sol

 74:     function getDepositEntry(uint32 _depositNonce) external view override returns (Deposit memory) {

```

```solidity
File: src/BranchBridgeAgent.sol

 17:         uint16 _rootChainId,

 18:         uint16 _localChainId,

 46:     uint16 public immutable rootChainId;

 49:     uint16 public immutable localChainId;



 111:         uint16 _rootChainId,

 112:         uint16 _localChainId,

 162:         uint32 _depositNonce

 233:         uint32 _depositNonce = depositNonce;

 268:         uint32 _depositNonce = depositNonce;

 323:         uint32 _depositNonce = depositNonce;

 360:         uint32 _depositNonce = depositNonce;

 396:         uint32 _depositNonce,

 474:         uint32 _depositNonce,

 493:     function redeemDeposit(uint32 _depositNonce) external override lock {

 530:         uint32 _settlementNonce,

 579:         uint8 numOfAssets = uint8(bytes1(_sParams[0]));

 582:         uint32 nonce = uint32(bytes4(_sParams[PARAMS_START:PARAMS_TKN_START]));

 683:         uint16,

 685:         uint64,


 880:         uint32 _settlementNonce,

 946:         uint32 _settlementNonce

 977:         uint32 _depositNonce,

 1031:         uint32 _depositNonce,

```

```solidity
File: src/CoreBranchRouter.sol

 64:         uint8 decimals = ERC20(_underlyingAddress).decimals();

 93:                 uint8 decimals,

 96:             ) = abi.decode(_params[1:], (address, string, string, uint8, address, GasParams));

 170:         uint8 _decimals,

```

```solidity
File: src/CoreRootRouter.sol

 108:         uint16 _dstChainId,

 160:         uint16 _dstChainId,

 189:         uint16 _dstChainId,

 216:         uint16 _dstChainId,

 247:         uint16 _dstChainId,

 274:         uint16 _dstChainId,

 297:     function executeResponse(bytes calldata _encodedData, uint16 _srcChainId)

 308:             (address underlyingAddress, address localAddress, string memory name, string memory symbol, uint8 decimals)

 309:             = abi.decode(_encodedData[1:], (address, address, string, string, uint8));

 332:     function execute(bytes calldata _encodedData, uint16) external payable override requiresExecutor {

 338:             (address refundee, address globalAddress, uint16 dstChainId, GasParams[2] memory gasParams) =

 339:                 abi.decode(_encodedData[1:], (address, address, uint16, GasParams[2]));

 350:     function executeDepositSingle(bytes memory, DepositParams memory, uint16)

 360:     function executeDepositMultiple(bytes calldata, DepositMultipleParams memory, uint16)

 370:     function executeSigned(bytes memory, address, uint16) external payable override requiresExecutor {

 375:     function executeSignedDepositSingle(bytes memory, DepositParams memory, address, uint16)

 385:     function executeSignedDepositMultiple(bytes memory, DepositMultipleParams memory, address, uint16)

 409:         uint16 _dstChainId,

 457:         uint8 _decimals,

 458:         uint16 _srcChainId

 481:     function _setLocalToken(address _globalAddress, address _localAddress, uint16 _dstChainId) internal {

```

```solidity
File: src/MulticallRootRouter.sol

 123:     function executeResponse(bytes memory, uint16) external payable override {

 137:     function execute(bytes calldata encodedData, uint16) external payable override lock requiresExecutor {

 155:                 uint16 dstChainId,

 157:             ) = abi.decode(_decode(encodedData[1:]), (IMulticall.Call[], OutputParams, uint16, GasParams));

 179:                 uint16 dstChainId,

 181:             ) = abi.decode(_decode(encodedData[1:]), (IMulticall.Call[], OutputMultipleParams, uint16, GasParams));

 203:     function executeDepositSingle(bytes calldata, DepositParams calldata, uint16) external payable override {

 209:     function executeDepositMultiple(bytes calldata, DepositMultipleParams calldata, uint16) external payable {

 223:     function executeSigned(bytes calldata encodedData, address userAccount, uint16)

 244:             (Call[] memory calls, OutputParams memory outputParams, uint16 dstChainId, GasParams memory gasParams) =

 245:                 abi.decode(_decode(encodedData[1:]), (Call[], OutputParams, uint16, GasParams));

 270:                 uint16 dstChainId,

 272:             ) = abi.decode(_decode(encodedData[1:]), (Call[], OutputMultipleParams, uint16, GasParams));

 312:     function executeSignedDepositSingle(bytes calldata encodedData, DepositParams calldata, address userAccount, uint16)

 333:             (Call[] memory calls, OutputParams memory outputParams, uint16 dstChainId, GasParams memory gasParams) =

 334:                 abi.decode(_decode(encodedData[1:]), (Call[], OutputParams, uint16, GasParams));

 359:                 uint16 dstChainId,

 361:             ) = abi.decode(_decode(encodedData[1:]), (Call[], OutputMultipleParams, uint16, GasParams));

 405:         uint16

 421:             (Call[] memory calls, OutputParams memory outputParams, uint16 dstChainId, GasParams memory gasParams) =

 422:                 abi.decode(_decode(encodedData[1:]), (Call[], OutputParams, uint16, GasParams));

 447:                 uint16 dstChainId,

 449:             ) = abi.decode(_decode(encodedData[1:]), (Call[], OutputMultipleParams, uint16, GasParams));

 514:         uint16 dstChainId,

 550:         uint16 dstChainId,

```

```solidity
File: src/RootBridgeAgent.sol

 40:     uint16 public immutable localChainId;

 106:         uint16 _localChainId,

 135:     function getSettlementEntry(uint32 _settlementNonce) external view override returns (Settlement memory) {

 144:         uint16 _dstChainId

 163:         uint16 _dstChainId,

 178:         uint16 _dstChainId,

 205:         uint16 _dstChainId,

 234:         uint32 _settlementNonce,

 274:     function retrieveSettlement(uint32 _settlementNonce, GasParams calldata _gParams) external payable lock {

 299:     function redeemSettlement(uint32 _settlementNonce) external override lock {

 325:                 uint24 _dstChainId = settlement.dstChainId;

 423:     function lzReceive(uint16 _srcChainId, bytes calldata _srcAddress, uint64, bytes calldata _payload) public {

 423:     function lzReceive(uint16 _srcChainId, bytes calldata _srcAddress, uint64, bytes calldata _payload) public {

 436:         uint16 _srcChainId,

 454:             uint16 srcChainId = _srcChainId;

 477:             uint16 srcChainId = _srcChainId;

 500:             uint16 srcChainId = _srcChainId;

 523:             uint16 srcChainId = _srcChainId;

 557:             uint16 srcChainId = _srcChainId;

 595:             uint16 srcChainId = _srcChainId;

 635:             uint16 srcChainId = _srcChainId;

 664:             (nonce, owner, params, gParams) = abi.decode(_payload[PARAMS_START:], (uint32, address, bytes, GasParams));

 749:     function _execute(uint256 _depositNonce, bytes memory _calldata, uint16 _srcChainId) private {

 770:         uint32 _depositNonce,

 773:         uint16 _srcChainId

 809:         uint16 _dstChainId,

 863:         uint32 _settlementNonce,

 866:         uint16 _dstChainId,

 938:     function _performFallbackCall(address payable _refundee, uint32 _depositNonce, uint16 _dstChainId) internal {

 938:     function _performFallbackCall(address payable _refundee, uint32 _depositNonce, uint16 _dstChainId) internal {

 967:         uint32 _settlementNonce,

 970:         uint16 _dstChainId,

 1004:         uint32 cachedSettlementNonce = _settlementNonce;

 1046:         uint32 _settlementNonce,

 1049:         uint16 _dstChainId,

 1076:             uint16 destChainId = _dstChainId;

 1138:         uint16 _dstChainId

 1204:     modifier requiresEndpoint(address _endpoint, uint16 _srcChain, bytes calldata _srcAddress) virtual {

```

```solidity
File: src/RootBridgeAgentExecutor.sol

 50:     function executeSystemRequest(address _router, bytes calldata _payload, uint16 _srcChainId)

 66:     function executeNoDeposit(address _router, bytes calldata _payload, uint16 _srcChainId)

 82:     function executeWithDeposit(address _router, bytes calldata _payload, uint16 _srcChainId)

 115:     function executeWithDepositMultiple(address _router, bytes calldata _payload, uint16 _srcChainId)

 150:     function executeSignedNoDeposit(address _account, address _router, bytes calldata _payload, uint16 _srcChainId)

 167:     function executeSignedWithDeposit(address _account, address _router, bytes calldata _payload, uint16 _srcChainId)

 205:         uint16 _srcChainId

 243:     function _bridgeIn(address _recipient, DepositParams memory _dParams, uint16 _srcChainId) internal {

 268:     function _bridgeInMultiple(address _recipient, bytes calldata _dParams, uint16 _srcChainId)

 273:         uint8 numOfAssets = uint8(bytes1(_dParams[0]));

 274:         uint32 nonce = uint32(bytes4(_dParams[PARAMS_START:5]));

```

```solidity
File: src/RootPort.sol

 443:         uint8 _wrappedGasTokenDecimals,

 525:         uint16 _dstChainId,

 539:     function syncNewCoreBranchRouter(address _coreBranchRouter, address _coreBranchBridgeAgent, uint16 _dstChainId)

```

```solidity
File: src/factories/ArbitrumBranchBridgeAgentFactory.sol

 31:         uint16 _rootChainId,

```

```solidity
File: src/factories/BranchBridgeAgentFactory.sol

 21:     uint16 public immutable localChainId;

 24:     uint16 public immutable rootChainId;

 53:         uint16 _localChainId,

 54:         uint16 _rootChainId,

```

```solidity
File: src/factories/ERC20hTokenBranchFactory.sol

 42:     constructor(uint16 _localChainId, address _localPortAddress, string memory _chainName, string memory _chainSymbol) {

 96:     function createToken(string memory _name, string memory _symbol, uint8 _decimals, bool _addPrefix)

```

```solidity
File: src/factories/ERC20hTokenRootFactory.sol

 14:     uint16 public immutable localChainId;

 34:     constructor(uint16 _localChainId, address _rootPortAddress) {

 76:     function createToken(string memory _name, string memory _symbol, uint8 _decimals)

```

```solidity
File: src/factories/RootBridgeAgentFactory.sol

 13:     uint16 public immutable rootChainId;

 31:     constructor(uint16 _rootChainId, address _lzEndpointAddress, address _rootPortAddress) {

```

```solidity
File: src/interfaces/BridgeAgentConstants.sol

 13:     uint8 internal constant STATUS_READY = 0;

 15:     uint8 internal constant STATUS_DONE = 1;

 17:     uint8 internal constant STATUS_RETRIEVE = 2;

 21:     uint8 internal constant STATUS_FAILED = 1;

 23:     uint8 internal constant STATUS_SUCCESS = 0;

```

```solidity
File: src/interfaces/BridgeAgentStructs.sol

 14:     uint8 status;

 40:     uint8 numberOfAssets; //Number of assets to deposit.

 41:     uint32 depositNonce; //Deposit nonce.

 50:     uint32 depositNonce; //Deposit nonce.

 58:     uint16 dstChainId; //Destination chain for interaction.

 59:     uint80 status; //Status of the settlement

 81:     uint32 settlementNonce; // Settlement nonce.

 90:     uint8 numberOfAssets; // Number of assets to deposit.

 92:     uint32 settlementNonce; // Settlement nonce.

```

```solidity
File: src/interfaces/IBranchBridgeAgent.sol

 109:     function getDepositEntry(uint32 depositNonce) external view returns (Deposit memory);

 250:         uint32 depositNonce,

 263:     function retrieveDeposit(uint32 depositNonce, GasParams calldata gasParams) external payable;

 270:     function redeemDeposit(uint32 depositNonce) external;

 286:         uint32 settlementNonce,

```

```solidity
File: src/interfaces/IBranchRouter.sol

 79:     function getDepositEntry(uint32 depositNonce) external view returns (Deposit memory);

```

```solidity
File: src/interfaces/IERC20hTokenBranchFactory.sol

 24:     function createToken(string memory _name, string memory _symbol, uint8 _decimals, bool _addPrefix)

```

```solidity
File: src/interfaces/IERC20hTokenRoot.sol

 19:     function localChainId() external view returns (uint16);

```

```solidity
File: src/interfaces/IERC20hTokenRootFactory.sol

 24:     function createToken(string memory _name, string memory _symbol, uint8 _decimals)

```

```solidity
File: src/interfaces/ILayerZeroEndpoint.sol

 16:         uint16 _dstChainId,

 32:         uint16 _srcChainId,

 35:         uint64 _nonce,

 43:     function getInboundNonce(uint16 _srcChainId, bytes calldata _srcAddress) external view returns (uint64);

 43:     function getInboundNonce(uint16 _srcChainId, bytes calldata _srcAddress) external view returns (uint64);

 47:     function getOutboundNonce(uint16 _dstChainId, address _srcAddress) external view returns (uint64);

 47:     function getOutboundNonce(uint16 _dstChainId, address _srcAddress) external view returns (uint64);

 56:         uint16 _dstChainId,

 64:     function getChainId() external view returns (uint16);

 70:     function retryPayload(uint16 _srcChainId, bytes calldata _srcAddress, bytes calldata _payload) external;

 75:     function hasStoredPayload(uint16 _srcChainId, bytes calldata _srcAddress) external view returns (bool);

 98:     function getConfig(uint16 _version, uint16 _chainId, address _userApplication, uint256 _configType)

 98:     function getConfig(uint16 _version, uint16 _chainId, address _userApplication, uint256 _configType)

 105:     function getSendVersion(address _userApplication) external view returns (uint16);

 109:     function getReceiveVersion(address _userApplication) external view returns (uint16);

```

```solidity
File: src/interfaces/ILayerZeroReceiver.sol

 11:     function lzReceive(uint16 _srcChainId, bytes calldata _srcAddress, uint64 _nonce, bytes calldata _payload) external;

 11:     function lzReceive(uint16 _srcChainId, bytes calldata _srcAddress, uint64 _nonce, bytes calldata _payload) external;

```

```solidity
File: src/interfaces/ILayerZeroUserApplicationConfig.sol

 11:     function setConfig(uint16 _version, uint16 _chainId, uint256 _configType, bytes calldata _config) external;

 11:     function setConfig(uint16 _version, uint16 _chainId, uint256 _configType, bytes calldata _config) external;

 15:     function setSendVersion(uint16 _version) external;

 19:     function setReceiveVersion(uint16 _version) external;

 24:     function forceResumeReceive(uint16 _srcChainId, bytes calldata _srcAddress) external;

```

```solidity
File: src/interfaces/IRootBridgeAgent.sol

 104:     function settlementNonce() external view returns (uint32 nonce);

 111:     function getSettlementEntry(uint32 _settlementNonce) external view returns (Settlement memory);

 153:         uint16 _dstChainId

 171:         uint16 _dstChainId,

 192:         uint16 _dstChainId,

 216:         uint16 _dstChainId,

 237:         uint32 _settlementNonce,

 250:     function retrieveSettlement(uint32 _settlementNonce, GasParams calldata _gParams) external payable;

 257:     function redeemSettlement(uint32 _depositNonce) external;

 311:         uint16 _srcChainId,

```

```solidity
File: src/interfaces/IRootPort.sol

 17:         uint16 _dstChainId,

 324:         uint8 _wrappedGasTokenDecimals,

 354:         uint16 _dstChainId,

 364:     function syncNewCoreBranchRouter(address _coreBranchRouter, address _coreBranchBridgeAgent, uint16 _dstChainId)

 389:         address indexed coreBranchRouter, address indexed coreBranchBridgeAgent, uint16 indexed dstChainId

 392:         address indexed coreBranchRouter, address indexed coreBranchBridgeAgent, uint16 indexed dstChainId

```

```solidity
File: src/interfaces/IRootRouter.sol

 25:     function executeResponse(bytes memory params, uint16 srcChainId) external payable;

 33:     function execute(bytes memory params, uint16 srcChainId) external payable;

 42:     function executeDepositSingle(bytes memory params, DepositParams memory dParams, uint16 srcChainId)

 53:     function executeDepositMultiple(bytes memory params, DepositMultipleParams memory dParams, uint16 srcChainId)

 63:     function executeSigned(bytes memory params, address userAccount, uint16 srcChainId) external payable;

 76:         uint16 srcChainId

 90:         uint16 srcChainId

```

```solidity
File: src/token/ERC20hTokenBranch.sol

 18:         uint8 _decimals,

```

```solidity
File: src/token/ERC20hTokenRoot.sol

 14:     uint16 public immutable override localChainId;

 32:         uint16 _localChainId,

 37:         uint8 _decimals

```

## [G-13]Low level call can be optimized with assembly
When using low-level calls, the returnData is copied to memory even if the variable is not utilized. The proper way to handle this is through a low level assembly call. 
```solidity 
 (bool success,) = payable(receiver).call{gas: gas, value: value}("");
 ```
 can be optimized to 
```solidity 
 bool success; 
 assembly { 
  success := call(gas, receiver, value, 0, 0, 0, 0) 
 } 
 ```


 ```solidity
File: src/ArbitrumBranchBridgeAgent.sol

 103:         _rootBridgeAgentAddress.call{value: msg.value}("");

```

```solidity
File: src/BranchBridgeAgent.sol

 863:         (bool success, ) = bridgeAgentExecutorAddress.call{

 888:         (bool success, ) = bridgeAgentExecutorAddress.call{

```

```solidity
File: src/RootBridgeAgent.sol

 754:         (bool success,) = bridgeAgentExecutorAddress.call{value: address(this).balance}(_calldata);

 779:         (bool success,) = bridgeAgentExecutorAddress.call{value: address(this).balance}(_calldata);

 835:             callee.call{value: msg.value}("");

 927:             callee.call{value: _value}("");

```

```solidity
File: src/VirtualAccount.sol

 120:                 (success, returnData[i]) = _call.target.call{value: val}(

```
 

## [G-14]A mapping is more efficient than an array
Fetching data from an array is more expensive than fetching data from a mapping. Fetching data from an array will require iterating over the array until ou reach your desired data. When using a mapping you only need to know the key in order to fetch the data from the exact slot it is stored in. If you need to be able to iterate over your group of data (such as with a for loop), then use an array. If you dont need to be able to iterate over your data, and will be able to fetch values based on a known key, then use a mapping.


 ```solidity
File: src/BaseBranchRouter.sol

 187:         address[] memory _hTokens,

 188:         address[] memory _tokens,

 189:         uint256[] memory _amounts,

 190:         uint256[] memory _deposits

```

```solidity
File: src/BranchBridgeAgent.sol

 585:         address[] memory _hTokens = new address[](numOfAssets);

 586:         address[] memory _tokens = new address[](numOfAssets);

 587:         uint256[] memory _amounts = new uint256[](numOfAssets);

 588:         uint256[] memory _deposits = new uint256[](numOfAssets);

 997:         address[] memory addressArray = new address[](1);

 998:         uint256[] memory uintArray = new uint256[](1);

 1033:         address[] memory _hTokens,

 1034:         address[] memory _tokens,

 1035:         uint256[] memory _amounts,

 1036:         uint256[] memory _deposits

```

```solidity
File: src/BranchPort.sol

 35:     address[] public bridgeAgents;

 45:     address[] public bridgeAgentFactories;

 55:     address[] public strategyTokens;

 71:     address[] public portStrategies;

 248:         address[] memory _localAddresses,

 249:         address[] memory _underlyingAddresses,

 250:         uint256[] memory _amounts,

 251:         uint256[] memory _deposits

 290:         address[] memory _localAddresses,

 291:         address[] memory _underlyingAddresses,

 292:         uint256[] memory _amounts,

 293:         uint256[] memory _deposits

```

```solidity
File: src/MulticallRootRouter.sol

 32:     address[] outputTokens;

 34:     uint256[] amountsOut;

 36:     uint256[] depositsOut;

 144:             (IMulticall.Call[] memory callData) = abi.decode(_decode(encodedData[1:]), (IMulticall.Call[]));

 153:                 IMulticall.Call[] memory callData,

 177:                 IMulticall.Call[] memory callData,

 236:             Call[] memory calls = abi.decode(_decode(encodedData[1:]), (Call[]));

 244:             (Call[] memory calls, OutputParams memory outputParams, uint16 dstChainId, GasParams memory gasParams) =

 268:                 Call[] memory calls,

 325:             Call[] memory calls = abi.decode(_decode(encodedData[1:]), (Call[]));

 333:             (Call[] memory calls, OutputParams memory outputParams, uint16 dstChainId, GasParams memory gasParams) =

 357:                 Call[] memory calls,

 413:             Call[] memory calls = abi.decode(_decode(encodedData[1:]), (Call[]));

 421:             (Call[] memory calls, OutputParams memory outputParams, uint16 dstChainId, GasParams memory gasParams) =

 445:                 Call[] memory calls,

 488:     function _multicall(IMulticall.Call[] memory calls)

 490:         returns (uint256 blockNumber, bytes[] memory returnData)

 547:         address[] memory outputTokens,

 548:         uint256[] memory amountsOut,

 549:         uint256[] memory depositsOut,

```

```solidity
File: src/RootBridgeAgent.sol

 858:         address[] memory _hTokens,

 859:         address[] memory _tokens,

 860:         uint256[] memory _amounts,

 861:         uint256[] memory _deposits,

 1007:         address[] memory addressArray = new address[](1);

 1008:         uint256[] memory uintArray = new uint256[](1);

 1050:         address[] memory _globalAddresses,

 1051:         uint256[] memory _amounts,

 1052:         uint256[] memory _deposits,

 1067:         address[] memory hTokens = new address[](_globalAddresses.length);

 1068:         address[] memory tokens = new address[](_globalAddresses.length);

```

```solidity
File: src/RootBridgeAgentExecutor.sol

 276:         address[] memory hTokens = new address[](numOfAssets);

 277:         address[] memory tokens = new address[](numOfAssets);

 278:         uint256[] memory amounts = new uint256[](numOfAssets);

 279:         uint256[] memory deposits = new uint256[](numOfAssets);

```

```solidity
File: src/RootPort.sol

 67:     address[] public bridgeAgents;

 80:     address[] public bridgeAgentFactories;

```

```solidity
File: src/VirtualAccount.sol

 75:         Call[] calldata calls

 80:         returns (bytes[] memory returnData)

 102:         PayableCall[] calldata calls

 103:     ) public payable returns (bytes[] memory returnData) {

 164:         uint256[] calldata,

 165:         uint256[] calldata,

```

```solidity
File: src/factories/ERC20hTokenBranchFactory.sol

 23:     ERC20hTokenBranch[] public hTokens;

 87:     function getHTokens() external view returns (ERC20hTokenBranch[] memory) {

```

```solidity
File: src/factories/ERC20hTokenRootFactory.sol

 23:     ERC20hTokenRoot[] public hTokens;

 63:     function getHTokens() external view returns (ERC20hTokenRoot[] memory) {

```

```solidity
File: src/interfaces/BridgeAgentStructs.sol

 16:     address[] hTokens;

 17:     address[] tokens;

 18:     uint256[] amounts;

 19:     uint256[] deposits;

 32:     address[] hTokens; //Input Local hTokens Address.

 33:     address[] tokens; //Input Native / underlying Token Address.

 34:     uint256[] amounts; //Amount of Local hTokens deposited for interaction.

 35:     uint256[] deposits; //Amount of native tokens deposited for interaction.

 42:     address[] hTokens; //Input Local hTokens Address.

 43:     address[] tokens; //Input Native / underlying Token Address.

 44:     uint256[] amounts; //Amount of Local hTokens deposited for interaction.

 45:     uint256[] deposits; //Amount of native tokens deposited for interaction.

 62:     address[] hTokens; //Input Local hTokens Addresses.

 63:     address[] tokens; //Input Native / underlying Token Addresses.

 64:     uint256[] amounts; //Amount of Local hTokens deposited for interaction.

 65:     uint256[] deposits; //Amount of native tokens deposited for interaction.

 75:     address[] globalAddresses; //Input Global hTokens Addresses.

 76:     uint256[] amounts; //Amount of Local hTokens deposited for interaction.

 77:     uint256[] deposits; //Amount of native tokens deposited for interaction.

 93:     address[] hTokens; // Input Local hTokens Addresses.

 94:     address[] tokens; // Input Native / underlying Token Addresses.

 95:     uint256[] amounts; // Amount of Local hTokens deposited for interaction.

 96:     uint256[] deposits; // Amount of native tokens deposited for interaction.

```

```solidity
File: src/interfaces/IBranchPort.sol

 105:         address[] memory _localAddresses,

 106:         address[] memory _underlyingAddresses,

 107:         uint256[] memory _amounts,

 108:         uint256[] memory _deposits

 137:         address[] memory _localAddresses,

 138:         address[] memory _underlyingAddresses,

 139:         uint256[] memory _amounts,

 140:         uint256[] memory _deposits

```

```solidity
File: src/interfaces/IMulticall2.sol

 20:     function aggregate(Call[] memory calls) external returns (uint256 blockNumber, bytes[] memory returnData);

 20:     function aggregate(Call[] memory calls) external returns (uint256 blockNumber, bytes[] memory returnData);

```

```solidity
File: src/interfaces/IVirtualAccount.sol

 65:     function call(Call[] calldata callInput) external returns (bytes[] memory);

 65:     function call(Call[] calldata callInput) external returns (bytes[] memory);

 73:     function payableCall(PayableCall[] calldata calls) external payable returns (bytes[] memory);

 73:     function payableCall(PayableCall[] calldata calls) external payable returns (bytes[] memory);

```


## [G-15]Optimize names to save gas
public/external function names and public member variable names can be optimized to save gas. Below are the interfaces/abstract contracts that can be optimized so that the most frequently-called functions use the least amount of gas possible during method lookup. Method IDs that have two leading zero bytes can save 128 gas each during deployment, and renaming functions to have lower method IDs will save 22 gas per call, per sorted position shifted.

Note: These instances where missed from bot.


```solidity
File: src/ArbitrumBranchPort.sol

 /// @audit  (), depositToPort(), withdrawFromPort(), localChainId, rootPortAddress, 
 14: contract ArbitrumBranchPort is BranchPort, IArbitrumBranchPort {

```

```solidity
File: src/ArbitrumCoreBranchRouter.sol

 /// @audit  (), addLocalToken(), executeNoSettlement(), 
 37: contract ArbitrumCoreBranchRouter is CoreBranchRouter {

```

```solidity
File: src/BaseBranchRouter.sol

 /// @audit  (), initialize(), getDepositEntry(), callOut(), callOutAndBridge(), callOutAndBridgeMultiple(), executeNoSettlement(), executeSettlement(), executeSettlementMultiple(), localPortAddress, localBridgeAgentAddress, bridgeAgentExecutorAddress, 
 25: contract BaseBranchRouter is IBranchRouter, Ownable {

```

```solidity
File: src/BranchBridgeAgent.sol

 /// @audit  (), (), getDepositEntry(), getFeeEstimate(), callOutSystem(), callOut(), callOutAndBridge(), callOutAndBridgeMultiple(), callOutSigned(), callOutSignedAndBridge(), callOutSignedAndBridgeMultiple(), retryDeposit(), retrieveDeposit(), redeemDeposit(), retrySettlement(), clearToken(), clearTokens(), lzReceive(), lzReceiveNonBlocking(), rootChainId, localChainId, rootBridgeAgentAddress, lzEndpointAddress, localRouterAddress, localPortAddress, bridgeAgentExecutorAddress, depositNonce, getDeposit, executionState, 
 38: contract BranchBridgeAgent is IBranchBridgeAgent, BridgeAgentConstants {

```


```solidity
File: src/BranchPort.sol

 /// @audit  (), initialize(), renounceOwnership(), manage(), replenishReserves(), replenishReserves(), withdraw(), bridgeIn(), bridgeInMultiple(), bridgeOut(), bridgeOutMultiple(), addBridgeAgent(), setCoreRouter(), addBridgeAgentFactory(), toggleBridgeAgentFactory(), toggleBridgeAgent(), addStrategyToken(), toggleStrategyToken(), addPortStrategy(), togglePortStrategy(), updatePortStrategy(), setCoreBranchRouter(), coreBranchRouterAddress, isBridgeAgent, bridgeAgents, isBridgeAgentFactory, bridgeAgentFactories, isStrategyToken, strategyTokens, getStrategyTokenDebt, getMinimumTokenReserveRatio, isPortStrategy, portStrategies, getPortStrategyTokenDebt, lastManaged, strategyDailyLimitAmount, strategyDailyLimitRemaining, 
 17: contract BranchPort is Ownable, IBranchPort {

```

```solidity
File: src/MulticallRootRouterLibZip.sol

 /// @audit  (), 
 26: contract MulticallRootRouterLibZip is MulticallRootRouter {

```


```solidity
File: src/VirtualAccount.sol

 /// @audit  (), (), withdrawNative(), withdrawERC20(), withdrawERC721(), call(), payableCall(), onERC721Received(), onERC1155Received(), onERC1155BatchReceived(), userAddress, localPortAddress, 
 17: contract VirtualAccount is IVirtualAccount, ERC1155Receiver {

```

```solidity
File: src/factories/ArbitrumBranchBridgeAgentFactory.sol

 /// @audit  (), initialize(), createBridgeAgent(), 
 17: contract ArbitrumBranchBridgeAgentFactory is BranchBridgeAgentFactory {

```


```solidity
File: src/factories/RootBridgeAgentFactory.sol

 /// @audit  (), createBridgeAgent(), rootChainId, rootPortAddress, lzEndpointAddress, 
 11: contract RootBridgeAgentFactory is IRootBridgeAgentFactory {

```

```solidity
File: src/interfaces/BridgeAgentConstants.sol

 10: contract BridgeAgentConstants {

```

```solidity
File: src/interfaces/IBranchBridgeAgentFactory.sol

 /// @audit  createBridgeAgent(), 
 12: interface IBranchBridgeAgentFactory {

```


```solidity
File: src/interfaces/IERC20hTokenBranch.sol

 /// @audit  mint(), burn(), 
 11: interface IERC20hTokenBranch {

```

```solidity
File: src/interfaces/IERC20hTokenBranchFactory.sol

 /// @audit  createToken(), 
 13: interface IERC20hTokenBranchFactory {

```


```solidity
File: src/interfaces/IERC20hTokenRootFactory.sol

 /// @audit  createToken(), 
 13: interface IERC20hTokenRootFactory {

```


```solidity
File: src/interfaces/ILayerZeroReceiver.sol

 /// @audit  lzReceive(), 
 5: interface ILayerZeroReceiver {

```


```solidity
File: src/interfaces/IMulticall2.sol

 /// @audit  aggregate(), 
 9: interface IMulticall2 {

```

```solidity
File: src/interfaces/IPortStrategy.sol

 /// @audit  withdraw(), 
 12: interface IPortStrategy {

```


```solidity
File: src/interfaces/IRootBridgeAgentFactory.sol

 /// @audit  createBridgeAgent(), 
 11: interface IRootBridgeAgentFactory {

```


```solidity
File: src/interfaces/IVirtualAccount.sol

 /// @audit  userAddress(), localPortAddress(), withdrawNative(), withdrawERC20(), withdrawERC721(), call(), payableCall(), 
 27: interface IVirtualAccount is IERC721Receiver {

```

```solidity
File: src/token/ERC20hTokenBranch.sol

 /// @audit  (), mint(), burn(), 
 12: contract ERC20hTokenBranch is ERC20, Ownable, IERC20hTokenBranch {

```

```solidity
File: src/token/ERC20hTokenRoot.sol

 /// @audit  (), mint(), burn(), localChainId, factoryAddress, getTokenBalance, 
 12: contract ERC20hTokenRoot is ERC20, Ownable, IERC20hTokenRoot {

```


## [G-16] Operator += costs more gas than <x> = <x> + <y> for variables
Using the += like operator instead of plus-equals saves 113 gas.


 ```solidity
File: src/BranchPort.sol

 157:         getPortStrategyTokenDebt[msg.sender][_token] += _amount;

 169:         getPortStrategyTokenDebt[msg.sender][_token] -= _amount;

 172:         getStrategyTokenDebt[_token] -= _amount;

```

```solidity
File: src/VirtualAccount.sol

 114:                 valAccumulator += val;

```

```solidity
File: src/token/ERC20hTokenRoot.sol

 58:         getTokenBalance[chainId] += amount;

 70:         getTokenBalance[chainId] -= amount;

```


## [G-17]Change public state variable visibility to private
its generally a good practice to limit the visibility of state variables to the minimum necessary level. This means that you should use the private visibility modifier for state variables whenever possible, and avoid using the public modifier unless it is absolutely necessary. Public state variables can be more expensive to read and write than private state variables, since Solidity generates additional getter and setter functions for public variables. By using private state variables, you can reduce the gas cost of your contract and improve its efficiency. Public state variables can be more expensive to read and write than private state variables, since Solidity generates additional getter and setter functions for public variables. By using private state variables, you can reduce the gas cost of your contract and improve its efficiency.


 ```solidity
File: src/ArbitrumBranchPort.sol

 22:     uint16 public immutable localChainId;

 26:     address public immutable rootPortAddress;

```

```solidity
File: src/BaseBranchRouter.sol

 33:     address public localPortAddress;

 36:     address public override localBridgeAgentAddress;

 39:     address public override bridgeAgentExecutorAddress;

```

```solidity
File: src/BranchBridgeAgent.sol

 46:     uint16 public immutable rootChainId;

 49:     uint16 public immutable localChainId;

 53:     address public immutable rootBridgeAgentAddress;

 60:     address public immutable lzEndpointAddress;

 63:     address public immutable localRouterAddress;

 67:     address public immutable localPortAddress;

 70:     address public immutable bridgeAgentExecutorAddress;

 77:     uint32 public depositNonce;

 80:     mapping(uint256 depositNonce => Deposit depositInfo) public getDeposit;

 87:     mapping(uint256 settlementNonce => uint256 state) public executionState;

```

```solidity
File: src/BranchPort.sol

 25:     address public coreBranchRouterAddress;

 32:     mapping(address bridgeAgent => bool isActiveBridgeAgent) public isBridgeAgent;

 35:     address[] public bridgeAgents;

 42:     mapping(address bridgeAgentFactory => bool isActiveBridgeAgentFactory) public isBridgeAgentFactory;

 45:     address[] public bridgeAgentFactories;

 52:     mapping(address token => bool allowsStrategies) public isStrategyToken;

 55:     address[] public strategyTokens;

 58:     mapping(address token => uint256 debt) public getStrategyTokenDebt;

 61:     mapping(address token => uint256 minimumReserveRatio) public getMinimumTokenReserveRatio;

 68:     mapping(address strategy => mapping(address token => bool isActiveStrategy)) public isPortStrategy;

 71:     address[] public portStrategies;

 74:     mapping(address strategy => mapping(address token => uint256 debt)) public getPortStrategyTokenDebt;

 77:     mapping(address strategy => mapping(address token => uint256 lastManaged)) public lastManaged;

 80:     mapping(address strategy => mapping(address token => uint256 dailyLimitAmount)) public strategyDailyLimitAmount;

 83:     mapping(address strategy => mapping(address token => uint256 dailyLimitRemaining)) public

```

```solidity
File: src/CoreBranchRouter.sol

 20:     address public immutable hTokenFactoryAddress;

```

```solidity
File: src/CoreRootRouter.sol

 47:     uint256 public immutable rootChainId;

 51:     address public immutable rootPortAddress;

 54:     address payable public bridgeAgentAddress;

 57:     address public bridgeAgentExecutorAddress;

 60:     address public hTokenFactoryAddress;

```

```solidity
File: src/MulticallRootRouter.sol

 65:     uint256 public immutable localChainId;

 68:     address public immutable localPortAddress;

 71:     address public immutable multicallAddress;

 74:     address payable public bridgeAgentAddress;

 77:     address public bridgeAgentExecutorAddress;

```

```solidity
File: src/RootBridgeAgent.sol

 40:     uint16 public immutable localChainId;

 43:     address public immutable factoryAddress;

 46:     address public immutable localRouterAddress;

 49:     address public immutable localPortAddress;

 52:     address public immutable lzEndpointAddress;

 55:     address public immutable bridgeAgentExecutorAddress;

 62:     mapping(uint256 chainId => address branchBridgeAgent) public getBranchBridgeAgent;

 65:     mapping(uint256 chainId => bytes branchBridgeAgentPath) public getBranchBridgeAgentPath;

 68:     mapping(uint256 chainId => bool allowed) public isBranchBridgeAgentAllowed;

 75:     uint32 public settlementNonce;

 78:     mapping(uint256 nonce => Settlement settlementInfo) public getSettlement;

 85:     mapping(uint256 chainId => mapping(uint256 nonce => uint256 state)) public executionState;

```

```solidity
File: src/RootPort.sol

 34:     uint256 public immutable localChainId;

 37:     address public localBranchPortAddress;

 40:     address public coreRootRouterAddress;

 43:     address public coreRootBridgeAgentAddress;

 50:     mapping(address user => VirtualAccount account) public getUserAccount;

 54:     mapping(VirtualAccount acount => mapping(address router => bool allowed)) public isRouterApproved;

 61:     mapping(uint256 chainId => bool isActive) public isChainId;

 64:     mapping(address bridgeAgent => bool isActive) public isBridgeAgent;

 67:     address[] public bridgeAgents;

 70:     mapping(address bridgeAgent => address bridgeAgentManager) public getBridgeAgentManager;

 77:     mapping(address bridgeAgentFactory => bool isActive) public isBridgeAgentFactory;

 80:     address[] public bridgeAgentFactories;

 87:     mapping(address token => bool isGlobalToken) public isGlobalAddress;

 90:     mapping(address chainId => mapping(uint256 localAddress => address globalAddress)) public getGlobalTokenFromLocal;

 93:     mapping(address chainId => mapping(uint256 globalAddress => address localAddress)) public getLocalTokenFromGlobal;

 96:     mapping(address chainId => mapping(uint256 underlyingAddress => address localAddress)) public

 100:     mapping(address chainId => mapping(uint256 localAddress => address underlyingAddress)) public

```

```solidity
File: src/VirtualAccount.sol

 21:     address public immutable override userAddress;

 24:     address public immutable override localPortAddress;

```

```solidity
File: src/factories/BranchBridgeAgentFactory.sol

 21:     uint16 public immutable localChainId;

 24:     uint16 public immutable rootChainId;

 27:     address public immutable rootBridgeAgentFactoryAddress;

 30:     address public immutable localCoreBranchRouterAddress;

 33:     address public immutable localPortAddress;

 36:     address public immutable lzEndpointAddress;

```

```solidity
File: src/factories/ERC20hTokenBranchFactory.sol

 14:     uint24 public immutable localChainId;

 17:     address public immutable localPortAddress;

 20:     address public localCoreRouterAddress;

 23:     ERC20hTokenBranch[] public hTokens;

 26:     string public chainName;

 29:     string public chainSymbol;

```

```solidity
File: src/factories/ERC20hTokenRootFactory.sol

 14:     uint16 public immutable localChainId;

 17:     address public immutable rootPortAddress;

 20:     address public coreRootRouterAddress;

 23:     ERC20hTokenRoot[] public hTokens;

```

```solidity
File: src/factories/RootBridgeAgentFactory.sol

 13:     uint16 public immutable rootChainId;

 16:     address public immutable rootPortAddress;

 19:     address public immutable lzEndpointAddress;

```

```solidity
File: src/token/ERC20hTokenRoot.sol

 14:     uint16 public immutable override localChainId;

 17:     address public immutable override factoryAddress;

 20:     mapping(uint256 chainId => uint256 balance) public override getTokenBalance;

```


## [G-18] Duplicated require()/revert() checks should be refactored to a modifier Or function to save gas
Saves deployment costs.


 ```solidity
File: src/ArbitrumBranchBridgeAgent.sol

 127:         if (_endpoint != rootBridgeAgentAddress) revert LayerZeroUnauthorizedEndpoint();

```

```solidity
File: src/ArbitrumBranchPort.sol

 86:         if (!IRootPort(_rootPortAddress).isGlobalToken(_globalAddress, localChainId)) revert UnknownGlobalToken();

```

```solidity
File: src/ArbitrumCoreBranchRouter.sol

 125:             revert UnrecognizedBridgeAgent();

```

```solidity
File: src/BaseBranchRouter.sol

 135:         revert UnrecognizedFunctionId();

```

```solidity
File: src/BranchBridgeAgent.sol

 463:         if (payload.length == 0) revert DepositRetryUnavailableUseCallout();

```

```solidity
File: src/BranchPort.sol

 299:         if (length > 255) revert InvalidInputArrays();

```

```solidity
File: src/CoreBranchRouter.sol

 212:             revert UnrecognizedBridgeAgentFactory();

```

```solidity
File: src/CoreRootRouter.sol

 366:         revert();

```

```solidity
File: src/MulticallRootRouter.sol

 204:         revert();

```

```solidity
File: src/RootBridgeAgent.sol

 282:         if (settlementOwner == address(0)) revert SettlementRetrieveUnavailable();

```

```solidity
File: src/RootPort.sol

 246:         if (_localAddress == address(0)) revert InvalidLocalAddress();

```

```solidity
File: src/VirtualAccount.sol

 92:             if (!success) revert CallFailed();

```

## [G-19]State variable read in a loop
The state variable should be cached in a local variable rather than reading it on every iteration of the for-loop, which will replace each Gwarmaccess (100 gas) with a much cheaper stack read.


 ```solidity
File: src/RootBridgeAgent.sol

 /// @audit  localPortAddress
 328:                 IPort(localPortAddress).bridgeToRoot(

 /// @audit  localPortAddress
 330:                     IPort(localPortAddress).getGlobalTokenFromLocal(_hToken, _dstChainId),

 /// @audit  localPortAddress
 1072:             hTokens[i] = IPort(localPortAddress).getLocalTokenFromGlobal(_globalAddresses[i], _dstChainId);

 /// @audit  localPortAddress
 1073:             tokens[i] = IPort(localPortAddress).getUnderlyingTokenFromLocal(hTokens[i], _dstChainId);

```

## [G-20] State variables only set in the constructor should be declared immutable 
Avoids a Gsset (20000 gas) in the constructor, and replaces the first access in each transaction (Gcoldsload - 2100 gas) and each access thereafter (Gwarmacces - 100 gas) with a PUSH32 (3 gas). While strings are not value types, and therefore cannot be immutable/constant if not hard-coded outside of the constructor, the same behavior can be achieved by making the current contract abstract with virtual functions for the string accessors, and having a child contract override the functions with the hard-coded implementation-specific values.

Note: These instances were missed by bot.


 ```solidity
File: src/ArbitrumBranchPort.sol

 41:         localChainId = _localChainId;

 42:         rootPortAddress = _rootPortAddress;

```

```solidity
File: src/BranchBridgeAgent.sol

 135:         localChainId = _localChainId;

 136:         rootChainId = _rootChainId;

 137:         rootBridgeAgentAddress = _rootBridgeAgentAddress;

 138:         lzEndpointAddress = _lzEndpointAddress;

 139:         localRouterAddress = _localRouterAddress;

 140:         localPortAddress = _localPortAddress;

 141:         bridgeAgentExecutorAddress = DeployBranchBridgeAgentExecutor.deploy();

```

```solidity
File: src/CoreBranchRouter.sol

 31:         hTokenFactoryAddress = _hTokenFactoryAddress;

```

```solidity
File: src/CoreRootRouter.sol

 72:         rootChainId = _rootChainId;

 73:         rootPortAddress = _rootPortAddress;

```

```solidity
File: src/MulticallRootRouter.sol

 96:         localChainId = _localChainId;

 97:         localPortAddress = _localPortAddress;

 98:         multicallAddress = _multicallAddress;

```

```solidity
File: src/RootBridgeAgent.sol

 115:         factoryAddress = msg.sender;

 116:         localChainId = _localChainId;

 117:         lzEndpointAddress = _lzEndpointAddress;

 118:         localPortAddress = _localPortAddress;

 119:         localRouterAddress = _localRouterAddress;

 120:         bridgeAgentExecutorAddress = DeployRootBridgeAgentExecutor.deploy(address(this));

```

```solidity
File: src/RootPort.sol

 112:         localChainId = _localChainId;

```

```solidity
File: src/VirtualAccount.sol

 36:         userAddress = _userAddress;

 37:         localPortAddress = _localPortAddress;

```

```solidity
File: src/factories/BranchBridgeAgentFactory.sol

 70:         localChainId = _localChainId;

 71:         rootChainId = _rootChainId;

 72:         rootBridgeAgentFactoryAddress = _rootBridgeAgentFactoryAddress;

 73:         lzEndpointAddress = _lzEndpointAddress;

 74:         localCoreBranchRouterAddress = _localCoreBranchRouterAddress;

 75:         localPortAddress = _localPortAddress;

```

```solidity
File: src/factories/ERC20hTokenBranchFactory.sol

 46:         localChainId = _localChainId;

 47:         localPortAddress = _localPortAddress;

```

```solidity
File: src/factories/ERC20hTokenRootFactory.sol

 36:         localChainId = _localChainId;

 37:         rootPortAddress = _rootPortAddress;

```

```solidity
File: src/factories/RootBridgeAgentFactory.sol

 34:         rootChainId = _rootChainId;

 35:         lzEndpointAddress = _lzEndpointAddress;

 36:         rootPortAddress = _rootPortAddress;

```

```solidity
File: src/token/ERC20hTokenRoot.sol

 42:         localChainId = _localChainId;

 43:         factoryAddress = _factoryAddress;

```

## [G-21] State variables only set in their definitions should be declared constant
Avoids a Gsset (20000 gas) at deployment, and replaces the first access in each transaction (Gcoldsload - 2100 gas) and each access thereafter (Gwarmacces - 100 gas) with a PUSH32 (3 gas).


 ```solidity
File: src/BaseBranchRouter.sol

 42:     uint256 internal _unlocked = 1;

```

```solidity
File: src/BranchBridgeAgent.sol

 94:     uint256 internal _unlocked = 1;

```

```solidity
File: src/BranchPort.sol

 91:     uint256 internal _unlocked = 1;

```

```solidity
File: src/MulticallRootRouter.sol

 80:     uint256 internal _unlocked = 1;

```

```solidity
File: src/RootBridgeAgent.sol

 92:     uint256 internal _unlocked = 1;

```


## [G-22] Unlimited gas consumption risk due to external call recipients
When calling an external function without specifying a gas limit , the called contract may consume all the remaining gas, causing the tx to be reverted. To mitigate this, it is recommended to explicitly set a gas limit when making low level external calls.


 ```solidity
File: src/ArbitrumBranchBridgeAgent.sol

 103:         _rootBridgeAgentAddress.call{value: msg.value}("");

```

```solidity
File: src/BranchBridgeAgent.sol

 863:         (bool success, ) = bridgeAgentExecutorAddress.call{

 888:         (bool success, ) = bridgeAgentExecutorAddress.call{

```

```solidity
File: src/RootBridgeAgent.sol

 754:         (bool success,) = bridgeAgentExecutorAddress.call{value: address(this).balance}(_calldata);

 779:         (bool success,) = bridgeAgentExecutorAddress.call{value: address(this).balance}(_calldata);

 835:             callee.call{value: msg.value}("");

 927:             callee.call{value: _value}("");

```

```solidity
File: src/VirtualAccount.sol

 120:                 (success, returnData[i]) = _call.target.call{value: val}(

```

## [G-23] Use != 0 instead of > 0 for unsigned integer comparison


 ```solidity
File: src/ArbitrumBranchPort.sol

 132:         if (_deposit > 0) {

 137:         if (_amount - _deposit > 0) {

```

```solidity
File: src/BaseBranchRouter.sol

 165:         if (_amount - _deposit > 0) {

 173:         if (_deposit > 0) {

```

```solidity
File: src/BranchBridgeAgent.sol

 1086:         if (_amount - _deposit > 0) {

 1096:         if (_deposit > 0) {

```

```solidity
File: src/BranchPort.sol

 259:             if (_amounts[i] - _deposits[i] > 0) {

 266:             if (_deposits[i] > 0) {

 525:         if (_hTokenAmount > 0) {

 531:         if (_deposit > 0) {

```

```solidity
File: src/RootBridgeAgent.sol

 363:         if (_dParams.amount > 0) {

 370:         if (_dParams.deposit > 0) {

 1146:         if (_underlyingAddress == address(0)) if (_deposit > 0) revert UnrecognizedUnderlyingAddress();

 1149:         if (_amount - _deposit > 0) {

 1156:         if (_deposit > 0) {

```

```solidity
File: src/RootPort.sol

 284:         if (_amount - _deposit > 0) {

 290:         if (_deposit > 0) if (!ERC20hTokenRoot(_hToken).mint(_recipient, _deposit, _srcChainId)) revert UnableToMint();

```

```solidity
File: src/VirtualAccount.sol

 180:         return size > 0;

```

```solidity
File: src/interfaces/ILayerZeroEndpoint.sol

 3: pragma solidity >=0.5.0;

```

```solidity
File: src/interfaces/ILayerZeroReceiver.sol

 3: pragma solidity >=0.5.0;

```

```solidity
File: src/interfaces/ILayerZeroUserApplicationConfig.sol

 3: pragma solidity >=0.5.0;

```

## [G-24] Unused named return variables without optimizer waste gas
Consider changing the variable to be an unnamed one, since the variable is never assigned, nor is it returned by name. If the optimizer is not turned on, leaving the code as it is will also waste gas for the stack variable.


 ```solidity
File: src/BranchBridgeAgent.sol

 172:     ) external view returns (uint256 _fee) {

```

```solidity
File: src/MulticallRootRouter.sol

 490:         returns (uint256 blockNumber, bytes[] memory returnData)

```

```solidity
File: src/RootBridgeAgent.sol

 145:     ) external view returns (uint256 _fee) {

```

```solidity
File: src/interfaces/IBranchBridgeAgent.sol

 123:         returns (uint256 _fee);

```

```solidity
File: src/interfaces/IBranchBridgeAgentFactory.sol

 28:     ) external returns (address newBridgeAgent);

```

```solidity
File: src/interfaces/IERC20hTokenBranchFactory.sol

 26:         returns (ERC20hTokenBranch newToken);

```

```solidity
File: src/interfaces/IERC20hTokenRootFactory.sol

 26:         returns (ERC20hTokenRoot newToken);

```

```solidity
File: src/interfaces/ILayerZeroEndpoint.sol

 61:     ) external view returns (uint256 nativeFee, uint256 zroFee);

```

```solidity
File: src/interfaces/IMulticall2.sol

 20:     function aggregate(Call[] memory calls) external returns (uint256 blockNumber, bytes[] memory returnData);

```

```solidity
File: src/interfaces/IRootBridgeAgent.sol

 104:     function settlementNonce() external view returns (uint32 nonce);

 154:     ) external view returns (uint256 _fee);

```

```solidity
File: src/interfaces/IRootBridgeAgentFactory.sol

 16:     function createBridgeAgent(address newRootRouterAddress) external returns (address newBridgeAgent);

```

```solidity
File: src/interfaces/IRootPort.sol

 261:     function fetchVirtualAccount(address _user) external returns (VirtualAccount account);

```


## [G-25] Remove or replace unused state variables
Saves a storage slot. If the variable is assigned a non-zero value, saves Gsset (20000 gas). If it’s assigned a zero value, saves Gsreset (2900 gas). If the variable remains unassigned, there is no gas savings unless the variable is public, in which case the compiler-generated non-payable getter deployment cost is saved. If the state variable is overriding an interface’s public function, mark the variable as constant or immutable so that it does not use a storage slot.


 ```solidity
File: src/interfaces/BridgeAgentConstants.sol

 13:     uint8 internal constant STATUS_READY = 0;

 15:     uint8 internal constant STATUS_DONE = 1;

 17:     uint8 internal constant STATUS_RETRIEVE = 2;

 21:     uint8 internal constant STATUS_FAILED = 1;

 23:     uint8 internal constant STATUS_SUCCESS = 0;

 27:     uint256 internal constant PARAMS_START = 1;

 29:     uint256 internal constant PARAMS_START_SIGNED = 21;

 31:     uint256 internal constant PARAMS_TKN_START = 5;

 33:     uint256 internal constant PARAMS_TKN_START_SIGNED = 25;

 35:     uint256 internal constant PARAMS_ENTRY_SIZE = 32;

 37:     uint256 internal constant PARAMS_ADDRESS_SIZE = 20;

 39:     uint256 internal constant PARAMS_TKN_SET_SIZE = 109;

 41:     uint256 internal constant PARAMS_TKN_SET_SIZE_MULTIPLE = 128;

 43:     uint256 internal constant ADDRESS_END_OFFSET = 12;

 45:     uint256 internal constant PARAMS_AMT_OFFSET = 64;

 47:     uint256 internal constant PARAMS_DEPOSIT_OFFSET = 96;

 49:     uint256 internal constant PARAMS_END_OFFSET = 6;

 51:     uint256 internal constant PARAMS_END_SIGNED_OFFSET = 26;

 53:     uint256 internal constant PARAMS_SETTLEMENT_OFFSET = 129;

 57:     uint256 internal constant MAX_TOKENS_LENGTH = 255;

```


## [G-26] Using bitmap to store bool states can save gas
Using a bitmap instead of a bool array or a bool mapping to store boolean states can save gas fees. This is because the bitmap can store 256 boolean values in a single slot instead of 256 slots, which can save gas when writing bool values or when reading multiple bool values from the same slot.


 ```solidity
File: src/BranchPort.sol

 32:     mapping(address bridgeAgent => bool isActiveBridgeAgent) public isBridgeAgent;

 42:     mapping(address bridgeAgentFactory => bool isActiveBridgeAgentFactory) public isBridgeAgentFactory;

 52:     mapping(address token => bool allowsStrategies) public isStrategyToken;

 68:     mapping(address strategy => mapping(address token => bool isActiveStrategy)) public isPortStrategy;

```

```solidity
File: src/RootBridgeAgent.sol

 68:     mapping(uint256 chainId => bool allowed) public isBranchBridgeAgentAllowed;

```

```solidity
File: src/RootPort.sol

 54:     mapping(VirtualAccount acount => mapping(address router => bool allowed)) public isRouterApproved;

 61:     mapping(uint256 chainId => bool isActive) public isChainId;

 64:     mapping(address bridgeAgent => bool isActive) public isBridgeAgent;

 77:     mapping(address bridgeAgentFactory => bool isActive) public isBridgeAgentFactory;

 87:     mapping(address token => bool isGlobalToken) public isGlobalAddress;

```


## [G-27] Use do while loops instead of for loops
A do while loop will cost less gas since the condition is not being checked for the first iteration.


 ```solidity
File: src/BaseBranchRouter.sol

 192:         for (uint256 i = 0; i < _hTokens.length;) {

```

```solidity
File: src/BranchBridgeAgent.sol

 506:         for (uint256 i = 0; i < deposit.tokens.length; ) {

 591:         for (uint256 i = 0; i < numOfAssets; ) {

```

```solidity
File: src/BranchPort.sol

 257:         for (uint256 i = 0; i < length;) {

 305:         for (uint256 i = 0; i < length;) {

```

```solidity
File: src/MulticallRootRouter.sol

 278:             for (uint256 i = 0; i < outputParams.outputTokens.length;) {

 367:             for (uint256 i = 0; i < outputParams.outputTokens.length;) {

 455:             for (uint256 i = 0; i < outputParams.outputTokens.length;) {

 557:         for (uint256 i = 0; i < outputTokens.length;) {

```

```solidity
File: src/RootBridgeAgent.sol

 318:         for (uint256 i = 0; i < settlement.hTokens.length;) {

 399:         for (uint256 i = 0; i < length;) {

 1070:         for (uint256 i = 0; i < hTokens.length;) {

```

```solidity
File: src/RootBridgeAgentExecutor.sol

 281:         for (uint256 i = 0; i < uint256(uint8(numOfAssets));) {

```

```solidity
File: src/VirtualAccount.sol

 85:         for (uint256 i = 0; i < length; ) {

 108:         for (uint256 i = 0; i < length; ) {

```


## [G-28] Use hardcode address instead of address(this)
it can be more gas-efficient to use a hardcoded address instead of the address(this) expression, especially if you need to use the same address multiple times in your contract.


 ```solidity
File: src/ArbitrumBranchBridgeAgent.sol

 126:         if (msg.sender != address(this)) revert LayerZeroUnauthorizedEndpoint();

```

```solidity
File: src/ArbitrumBranchPort.sol

 69:         _underlyingAddress.safeTransferFrom(_depositor, address(this), _deposit);

 133:             _underlyingAddress.safeTransferFrom(_depositor, address(this), _deposit);

```

```solidity
File: src/BaseBranchRouter.sol

 167:                 _hToken.safeTransferFrom(msg.sender, address(this), _amount - _deposit);

 174:             _token.safeTransferFrom(msg.sender, address(this), _deposit);

```

```solidity
File: src/BranchBridgeAgent.sol

 175:             address(this),

 688:         address(this).excessivelySafeCall(

 864:             value: address(this).balance

 889:             value: address(this).balance

 950:             value: address(this).balance

 1125:         if (msg.sender != address(this)) revert LayerZeroUnauthorizedEndpoint();

```

```solidity
File: src/BranchBridgeAgentExecutor.sol

 92:             _recipient.safeTransferETH(address(this).balance);

 126:             _recipient.safeTransferETH(address(this).balance);

```

```solidity
File: src/BranchPort.sol

 175:         uint256 currBalance = ERC20(_token).balanceOf(address(this));

 178:         IPortStrategy(msg.sender).withdraw(address(this), _token, _amount);

 193:         uint256 currBalance = ERC20(_token).balanceOf(address(this));

 210:         IPortStrategy(_strategy).withdraw(address(this), _token, amountToWithdraw);

 436:         uint256 currBalance = ERC20(_token).balanceOf(address(this));

 526:             _localAddress.safeTransferFrom(_depositor, address(this), _hTokenAmount);

 532:             _underlyingAddress.safeTransferFrom(_depositor, address(this), _deposit);

```

```solidity
File: src/RootBridgeAgent.sol

 148:             address(this),

 424:         (bool success,) = address(this).excessivelySafeCall(

 695:                 address(this).balance

 754:         (bool success,) = bridgeAgentExecutorAddress.call{value: address(this).balance}(_calldata);

 779:         (bool success,) = bridgeAgentExecutorAddress.call{value: address(this).balance}(_calldata);

 940:         ILayerZeroEndpoint(lzEndpointAddress).send{value: address(this).balance}(

 1205:         if (msg.sender != address(this)) revert LayerZeroUnauthorizedEndpoint();

 1233:         if (msg.sender != IPort(localPortAddress).getBridgeAgentManager(address(this))) {

```

```solidity
File: src/RootPort.sol

 301:         _hToken.safeTransferFrom(_from, address(this), _amount);

 362:         newAccount = new VirtualAccount{salt: keccak256(abi.encode(_user))}(_user, address(this));

```

```solidity
File: src/VirtualAccount.sol

 70:         ERC721(_token).transferFrom(address(this), msg.sender, _tokenId);

```

```solidity
File: src/factories/ERC20hTokenRootFactory.sol

 83:             address(this),

```

## [G-29] Newer versions of solidity are more gas efficient
The solidity language continues to pursue more efficient gas optimization schemes. Adopting a newer version of solidity can be more gas efficient. 


 ```solidity
File: src/ArbitrumBranchBridgeAgent.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/ArbitrumBranchPort.sol

 3: pragma solidity ^0.8.0;

```

```solidity
File: src/ArbitrumCoreBranchRouter.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/BaseBranchRouter.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/BranchBridgeAgent.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/BranchBridgeAgentExecutor.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/BranchPort.sol

 3: pragma solidity ^0.8.0;

```

```solidity
File: src/CoreBranchRouter.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/CoreRootRouter.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/MulticallRootRouter.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/MulticallRootRouterLibZip.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/RootBridgeAgent.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/RootBridgeAgentExecutor.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/RootPort.sol

 3: pragma solidity ^0.8.0;

```

```solidity
File: src/VirtualAccount.sol

 3: pragma solidity ^0.8.0;

```

```solidity
File: src/factories/ArbitrumBranchBridgeAgentFactory.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/factories/BranchBridgeAgentFactory.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/factories/ERC20hTokenBranchFactory.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/factories/ERC20hTokenRootFactory.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/factories/RootBridgeAgentFactory.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/interfaces/BridgeAgentConstants.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/interfaces/BridgeAgentStructs.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/interfaces/IArbitrumBranchPort.sol

 3: pragma solidity ^0.8.0;

```

```solidity
File: src/interfaces/IBranchBridgeAgent.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/interfaces/IBranchBridgeAgentFactory.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/interfaces/IBranchPort.sol

 3: pragma solidity ^0.8.0;

```

```solidity
File: src/interfaces/IBranchRouter.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/interfaces/ICoreBranchRouter.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/interfaces/IERC20hTokenBranch.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/interfaces/IERC20hTokenBranchFactory.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/interfaces/IERC20hTokenRoot.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/interfaces/IERC20hTokenRootFactory.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/interfaces/ILayerZeroEndpoint.sol

 3: pragma solidity >=0.5.0;

```

```solidity
File: src/interfaces/ILayerZeroReceiver.sol

 3: pragma solidity >=0.5.0;

```

```solidity
File: src/interfaces/ILayerZeroUserApplicationConfig.sol

 3: pragma solidity >=0.5.0;

```

```solidity
File: src/interfaces/IMulticall2.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/interfaces/IPortStrategy.sol

 3: pragma solidity ^0.8.0;

```

```solidity
File: src/interfaces/IRootBridgeAgent.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/interfaces/IRootBridgeAgentFactory.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/interfaces/IRootPort.sol

 3: pragma solidity ^0.8.0;

```

```solidity
File: src/interfaces/IRootRouter.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/interfaces/IVirtualAccount.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/token/ERC20hTokenBranch.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/token/ERC20hTokenRoot.sol

 2: pragma solidity ^0.8.0;

```


## [G-30] Use a more recent version of solidity
- Use a solidity version of at least 0.8.2 to get compiler automatic inlining 
 - Use a solidity version of at least 0.8.3 to get better struct packing and cheaper multiple storage reads. 
 - Use a solidity version of at least 0.8.4 to get custom errors, which are cheaper at deployment than `revert()/require()` strings. 
 - Use a solidity version of at least 0.8.10 to have external calls skip contract existence checks if the external call has a return value.


 ```solidity
File: src/ArbitrumBranchBridgeAgent.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/ArbitrumBranchPort.sol

 3: pragma solidity ^0.8.0;

```

```solidity
File: src/ArbitrumCoreBranchRouter.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/BaseBranchRouter.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/BranchBridgeAgent.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/BranchBridgeAgentExecutor.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/BranchPort.sol

 3: pragma solidity ^0.8.0;

```

```solidity
File: src/CoreBranchRouter.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/CoreRootRouter.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/MulticallRootRouter.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/MulticallRootRouterLibZip.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/RootBridgeAgent.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/RootBridgeAgentExecutor.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/RootPort.sol

 3: pragma solidity ^0.8.0;

```

```solidity
File: src/VirtualAccount.sol

 3: pragma solidity ^0.8.0;

```

```solidity
File: src/factories/ArbitrumBranchBridgeAgentFactory.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/factories/BranchBridgeAgentFactory.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/factories/ERC20hTokenBranchFactory.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/factories/ERC20hTokenRootFactory.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/factories/RootBridgeAgentFactory.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/interfaces/BridgeAgentConstants.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/interfaces/BridgeAgentStructs.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/interfaces/IArbitrumBranchPort.sol

 3: pragma solidity ^0.8.0;

```

```solidity
File: src/interfaces/IBranchBridgeAgent.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/interfaces/IBranchBridgeAgentFactory.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/interfaces/IBranchPort.sol

 3: pragma solidity ^0.8.0;

```

```solidity
File: src/interfaces/IBranchRouter.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/interfaces/ICoreBranchRouter.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/interfaces/IERC20hTokenBranch.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/interfaces/IERC20hTokenBranchFactory.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/interfaces/IERC20hTokenRoot.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/interfaces/IERC20hTokenRootFactory.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/interfaces/ILayerZeroEndpoint.sol

 3: pragma solidity >=0.5.0;

```

```solidity
File: src/interfaces/ILayerZeroReceiver.sol

 3: pragma solidity >=0.5.0;

```

```solidity
File: src/interfaces/ILayerZeroUserApplicationConfig.sol

 3: pragma solidity >=0.5.0;

```

```solidity
File: src/interfaces/IMulticall2.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/interfaces/IPortStrategy.sol

 3: pragma solidity ^0.8.0;

```

```solidity
File: src/interfaces/IRootBridgeAgent.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/interfaces/IRootBridgeAgentFactory.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/interfaces/IRootPort.sol

 3: pragma solidity ^0.8.0;

```

```solidity
File: src/interfaces/IRootRouter.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/interfaces/IVirtualAccount.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/token/ERC20hTokenBranch.sol

 2: pragma solidity ^0.8.0;

```

```solidity
File: src/token/ERC20hTokenRoot.sol

 2: pragma solidity ^0.8.0;

```


## [G-31] Use unchecked block for safe subtractions
If it can be confirmed that the subtraction operation will not overflow, using an unchecked block can save gas. For example, require(x <= y); z = y - x; can be optimized to require(x <= y); unchecked { z = y - x; }.


 ```solidity
File: src/BranchPort.sol

 /// @audit  checked on line 440
 440:             return currBalance > minReserves ? currBalance - minReserves : 0;

 /// @audit  checked on line 440
 457:             return currBalance < minReserves ? minReserves - currBalance : 0;

 /// @audit  checked on line 457
 457:             return currBalance < minReserves ? minReserves - currBalance : 0;

```

```solidity
File: src/RootBridgeAgent.sol

 /// @audit  checked on line 1142
 1149:         if (_amount - _deposit > 0) {

 /// @audit  checked on line 1142
 1151:                 _globalAddress.safeTransferFrom(_depositor, localPortAddress, _amount - _deposit);

```


## [G-32] Use assembly to write address/contract type storage values
Using `assembly { sstore(state.slot, addr) instead of state = addr` can save gas.


 ```solidity
File: src/ArbitrumBranchPort.sol

 42:         rootPortAddress = _rootPortAddress;

```

```solidity
File: src/BaseBranchRouter.sol

 62:         localBridgeAgentAddress = _localBridgeAgentAddress;

 63:         localPortAddress = IBridgeAgent(_localBridgeAgentAddress).localPortAddress();

 64:         bridgeAgentExecutorAddress = IBridgeAgent(_localBridgeAgentAddress).bridgeAgentExecutorAddress();

```

```solidity
File: src/BranchBridgeAgent.sol

 137:         rootBridgeAgentAddress = _rootBridgeAgentAddress;

 138:         lzEndpointAddress = _lzEndpointAddress;

 139:         localRouterAddress = _localRouterAddress;

 140:         localPortAddress = _localPortAddress;

 141:         bridgeAgentExecutorAddress = DeployBranchBridgeAgentExecutor.deploy();

```

```solidity
File: src/BranchPort.sol

 129:         coreBranchRouterAddress = _coreBranchRouter;

 334:         coreBranchRouterAddress = _newCoreRouter;

 419:         coreBranchRouterAddress = _coreBranchRouter;

```

```solidity
File: src/CoreBranchRouter.sol

 31:         hTokenFactoryAddress = _hTokenFactoryAddress;

```

```solidity
File: src/CoreRootRouter.sol

 73:         rootPortAddress = _rootPortAddress;

 86:         bridgeAgentAddress = payable(_bridgeAgentAddress);

 87:         bridgeAgentExecutorAddress = IBridgeAgent(_bridgeAgentAddress).bridgeAgentExecutorAddress();

 88:         hTokenFactoryAddress = _hTokenFactory;

```

```solidity
File: src/MulticallRootRouter.sol

 97:         localPortAddress = _localPortAddress;

 98:         multicallAddress = _multicallAddress;

 112:         bridgeAgentAddress = payable(_bridgeAgentAddress);

 113:         bridgeAgentExecutorAddress = IBridgeAgent(_bridgeAgentAddress).bridgeAgentExecutorAddress();

```

```solidity
File: src/RootBridgeAgent.sol

 115:         factoryAddress = msg.sender;

 117:         lzEndpointAddress = _lzEndpointAddress;

 118:         localPortAddress = _localPortAddress;

 119:         localRouterAddress = _localRouterAddress;

 120:         bridgeAgentExecutorAddress = DeployRootBridgeAgentExecutor.deploy(address(this));

```

```solidity
File: src/RootPort.sol

 138:         coreRootRouterAddress = _coreRootRouter;

 159:         coreRootBridgeAgentAddress = _coreRootBridgeAgent;

 160:         localBranchPortAddress = _localBranchPortAddress;

 513:         coreRootRouterAddress = _coreRootRouter;

 514:         coreRootBridgeAgentAddress = _coreRootBridgeAgent;

```

```solidity
File: src/VirtualAccount.sol

 36:         userAddress = _userAddress;

 37:         localPortAddress = _localPortAddress;

```

```solidity
File: src/factories/BranchBridgeAgentFactory.sol

 72:         rootBridgeAgentFactoryAddress = _rootBridgeAgentFactoryAddress;

 73:         lzEndpointAddress = _lzEndpointAddress;

 74:         localCoreBranchRouterAddress = _localCoreBranchRouterAddress;

 75:         localPortAddress = _localPortAddress;

```

```solidity
File: src/factories/ERC20hTokenBranchFactory.sol

 47:         localPortAddress = _localPortAddress;

 74:         localCoreRouterAddress = _coreRouter;

```

```solidity
File: src/factories/ERC20hTokenRootFactory.sol

 37:         rootPortAddress = _rootPortAddress;

 51:         coreRootRouterAddress = _coreRouter;

```

```solidity
File: src/factories/RootBridgeAgentFactory.sol

 35:         lzEndpointAddress = _lzEndpointAddress;

 36:         rootPortAddress = _rootPortAddress;

```

```solidity
File: src/token/ERC20hTokenRoot.sol

 43:         factoryAddress = _factoryAddress;

```


## [G-33]Use assembly to emit events
To efficiently emit events, it’s possible to utilize assembly by making use of scratch space and the free memory pointer. This approach has the advantage of potentially avoiding the costs associated with memory expansion. However, it’s important to note that in order to safely optimize this process, it is preferable to cache and restore the free memory pointer.

Note: Instances below were missed by bot.


```solidity
File: src/BranchPort.sol

 163:         emit DebtCreated(msg.sender, _token, _amount);

 184:         emit DebtRepaid(msg.sender, _token, _amount);

 218:         emit DebtRepaid(_strategy, _token, amountToWithdraw);

 392:         emit PortStrategyAdded(_portStrategy, _token, _dailyManagementLimit);

 410:         emit PortStrategyUpdated(_portStrategy, _token, _dailyManagementLimit);


```

```solidity
File: src/RootPort.sol

 255:         emit LocalTokenAdded(_underlyingAddress, _localAddress, _globalAddress, _srcChainId);

 269:         emit GlobalTokenAdded(_localAddress, _globalAddress, _srcChainId);

 406:         emit BridgeAgentSynced(_newBranchBridgeAgent, _rootBridgeAgent, _branchChainId);

 535:         emit CoreBranchSet(_coreBranchRouter, _coreBranchBridgeAgent, _dstChainId);

 549:         emit CoreBranchSynced(_coreBranchRouter, _coreBranchBridgeAgent, _dstChainId);

```


## [G-34]Using assembly to check for zero can save gas
Using assembly to check for zero can save gas by allowing more direct access to the evm and reducing some of the overhead associated with high-level operations in solidity.

Note: Instances below were missed by bot.


```solidity
File: src/CoreRootRouter.sol

 120:         if (IBridgeAgent(_rootBridgeAgent).getBranchBridgeAgent(_dstChainId) != address(0)) revert InvalidChainId();

```

```solidity
File: src/RootBridgeAgent.sol
`
 323:             if (_hToken != address(0)) {


 1141:         if (_amount == 0 && _deposit == 0) revert InvalidInputParams();


 1171:         if (getBranchBridgeAgent[_branchChainId] != address(0)) revert AlreadyAddedBridgeAgent();

```

```solidity
File: src/RootPort.sol

 212:         return getLocalTokenFromGlobal[_globalAddress][_srcChainId] != address(0);

 217:         return getGlobalTokenFromLocal[_localAddress][_srcChainId] != address(0);

 226:         return _getLocalToken(_localAddress, _srcChainId, _dstChainId) != address(0);

 231:         return getLocalTokenFromUnderlying[_underlyingToken][_srcChainId] != address(0);

 398:         if (IBridgeAgent(_rootBridgeAgent).getBranchBridgeAgent(_branchChainId) != address(0)) {

 488:         if (getUnderlyingTokenFromLocal[_ecoTokenGlobalAddress][localChainId] != address(0)) {

 493:         if (getLocalTokenFromUnderlying[_ecoTokenGlobalAddress][localChainId] != address(0)) {

```
 