# gas optimizm

# Summary
|  no    | issue  | insteance  |
|------|--------|------------|
|[G-01]|Empty blocks should be removed or emit something|1|
|[G-02]|Use Modifiers Instead of Functions To Save Gas|5|
|[G-03]|>=/<= costs less gas than >/<|17|
|[G-04]|Use Assembly To Check For address(0)|35|
|[G-05]|Using Fixed Bytes Is Cheaper Than Using String|2|
|[G-06]|abi.encode() is less efficient than abi.encodepacked()|4|
|[G-07]|When possible, use assembly instead of unchecked{++i}|15|
|[G-08]|Use assembly in place of abi.decode to extract calldata values more efficiently|28|
|[G-09]|Use do while loops instead of for loops|15|
|[G-10]|A modifier used only once and not being inherited should be inlined to save gas|6|
|[G-11]| OR in if-condition can be rewritten to two single if conditions|1|
|[G-12]|Use ERC721A instead ERC721|1|
|[G-13]|Can Make The Variable Outside The Loop To Save Gas |5|
|[G-14]|Access mappings directly rather than using accessor functions|2|



## [G‑01] Empty blocks should be removed or emit something

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting.
If the contract is meant to be extended, the contract should be abstract and the function signatures be added without any default implementation.


```solidity
89       function retrySettlement(uint32, bytes calldata, GasParams[2] calldata, bool) external payable override lock {}
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchBridgeAgent.sol#L89


## [G-02] Use Modifiers Instead of Functions To Save Gas

By using modifiers  you can keep track of gas consumption and perform gas-saving operations based on your contract's specific requirements. 


There are 5 instances of this issue:
```solidity
File:  RootPort.sol
211   function isGlobalToken(address _globalAddress, uint256 _srcChainId) external view override returns (bool) {

216   function isLocalToken(address _localAddress, uint256 _srcChainId) external view override returns (bool) {

221   function isLocalToken(address _localAddress, uint256 _srcChainId, uint256 _dstChainId)

230   function isUnderlyingToken(address _underlyingToken, uint256 _srcChainId) external view override returns (bool) {
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L211
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L216
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L221
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L230


```solidity
147  function isContract(address addr) internal view returns (bool) {
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L147



## [G-03] >=/<= costs less gas than >/<

The compiler uses opcodes GT and ISZERO for code that uses >, but only requires LT for >=. A similar behaviour applies for >, which uses opcodes LT and ISZERO, but only requires GT for <=.



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
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L513


```solidity
File:  BranchPort.sol
257  for (uint256 i = 0; i < length;) {

305  for (uint256 i = 0; i < length;) {

363  if (_minimumReservesRatio >= DIVISIONER || _minimumReservesRatio < MIN_RESERVE_RATIO) {        
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L257
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L305
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L363


```solidity
File:  MulticallRootRouter.sol
278   for (uint256 i = 0; i < outputParams.outputTokens.length;) {

367   for (uint256 i = 0; i < outputParams.outputTokens.length;) {

455   for (uint256 i = 0; i < outputParams.outputTokens.length;) {

557   for (uint256 i = 0; i < outputTokens.length;) {            
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L278
https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L367
https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L455
https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L557


```solidity
File:  RootBridgeAgent.sol
318   for (uint256 i = 0; i < settlement.hTokens.length;) {

399   for (uint256 i = 0; i < length;) {

1070  for (uint256 i = 0; i < hTokens.length;) {

1142   if (_amount < _deposit) revert InvalidInputParams();

1158  if (IERC20hTokenRoot(_globalAddress).getTokenBalance(_dstChainId) < _deposit) {
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L318
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L399
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1070
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1142
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1158



```solidity
File:  VirtualAccount.sol
70   for (uint256 i = 0; i < length;) {

90   for (uint256 i = 0; i < length;) {    
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L70
https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L90



## [G-04] Use Assembly To Check For address(0)

Using assembly in Solidity can provide more fine-grained control over gas usage and can be used to optimize certain operations. 

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



## [G-05] Using Fixed Bytes Is Cheaper Than Using String


If you can limit the length to a certain number of bytes, always use one of bytes1 to bytes32 because they are much cheaper.

```solidity
File:  factories/ERC20hTokenBranchFactory.sol
26  string public chainName;

29  string public chainSymbol;
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenBranchFactory.sol#L26
https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenBranchFactory.sol#L29



## [G-06] abi.encode() is less efficient than abi.encodepacked()

 abi.encodePacked() is generally considered to be more gas-efficient than abi.encode(), because it skips the step of adding function signatures and other metadata to the encoded data. However, this comes at the cost of reduced safety, as abi.encodePacked() does not perform any type checking or padding of data.

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
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L178
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L226



## [G-07] When possible, use assembly instead of unchecked{++i}

You can also use unchecked{++i;} for even more gas savings but this will not check to see if i overflows. For best gas savings, use inline assembly, however, this limits the functionality you can achieve.



```solidity
195  unchecked {
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L195

```solidity
File:  BranchBridgeAgent.sol
450  unchecked {

563  unchecked {
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L450
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L563


```solidity
File:  BranchPort.sol
270  unchecked {

308  unchecked {
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L270
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L308



```solidity
File:  MulticallRootRouter.sol
281  unchecked {

370  unchecked {

458  unchecked {

560  unchecked {
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L281
https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L370
https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L458
https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L560



```solidity
File:  RootBridgeAgent.sol
337  unchecked {

412  unchecked {

1083 unchecked {
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L337
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L412
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1083


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
https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L105





## [G-08] Use assembly in place of abi.decode to extract calldata values more efficiently


Instead of using abi.decode, we can use assembly to decode our desired calldata values directly. This will allow us to avoid decoding calldata values that we will not use.

```solidity
File:   ArbitrumCoreBranchRouter.sol
134  ) = abi.decode(_data[1:], (address, address, address, address, address, GasParams));

147   (address bridgeAgentFactoryAddress) = abi.decode(_data[1:], (address));

153   (address branchBridgeAgent) = abi.decode(_data[1:], (address));

158    (address underlyingToken, uint256 minimumReservesRatio) = abi.decode(_data[1:], (address, uint256));

164   abi.decode(_data[1:], (address, address, uint256, bool));
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumCoreBranchRouter.sol#L134
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumCoreBranchRouter.sol#L147
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumCoreBranchRouter.sol#L153
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumCoreBranchRouter.sol#L158
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumCoreBranchRouter.sol#L164


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
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L108
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L116
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L122
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L128
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L135
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L141


```solidity
File:  CoreRootRouter.sol
309   = abi.decode(_encodedData[1:], (address, address, string, string, uint8));

315   (address globalAddress, address localAddress) = abi.decode(_encodedData[1:], (address, address));

321  (address newBranchBridgeAgent, address rootBridgeAgent) = abi.decode(_encodedData[1:], (address, address));

339   abi.decode(_encodedData[1:], (address, address, uint16, GasParams[2]));
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L309
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L315
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L321
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L339


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



## [G-09] Use do while loops instead of for loops

Using do-while loops instead of for loops can potentially help save gas in certain scenarios. The gas-saving advantage of do-while loops lies in the fact that they allow for more efficient gas consumption when the number of iterations is variable or uncertain.



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
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L513


```solidity
File:  BranchPort.sol
257  for (uint256 i = 0; i < length;) {

305  for (uint256 i = 0; i < length;) {    
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L257
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L305


```solidity
File:  MulticallRootRouter.sol
278   for (uint256 i = 0; i < outputParams.outputTokens.length;) {

367   for (uint256 i = 0; i < outputParams.outputTokens.length;) {

455   for (uint256 i = 0; i < outputParams.outputTokens.length;) {

557   for (uint256 i = 0; i < outputTokens.length;) {            
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L278
https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L367
https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L455
https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L557


```solidity
File:  RootBridgeAgent.sol
318   for (uint256 i = 0; i < settlement.hTokens.length;) {

399   for (uint256 i = 0; i < length;) {

1070  for (uint256 i = 0; i < hTokens.length;) {
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L318
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L399
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1070

```solidity
File:  VirtualAccount.sol
70   for (uint256 i = 0; i < length;) {

90   for (uint256 i = 0; i < length;) {    
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L70
https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L790


```solidity
File:  RootBridgeAgentExecutor.sol
281  for (uint256 i = 0; i < uint256(uint8(numOfAssets));) {
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L281




## [G-10] A modifier used only once and not being inherited should be inlined to save gas



By inlining a modifier that is used only once and not being inherited, you can eliminate the overhead of the generated code and reduce the gas cost of your contract.

```solidity
File:  BranchBridgeAgent.sol
930  modifier requiresEndpoint(address _endpoint, bytes calldata _srcAddress) {
        _requiresEndpoint(_endpoint, _srcAddress);
        _;
    }

947 modifier requiresRouter() {
        if (msg.sender != localRouterAddress) revert UnrecognizedRouter();
        _;
    }    
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L930
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L947


```solidity
File:  BranchPort.sol
553  modifier requiresBridgeAgentFactory() {
        if (!isBridgeAgentFactory[msg.sender]) revert UnrecognizedBridgeAgentFactory();
        _;
    }

559  modifier requiresPortStrategy(address _token) {
        if (!isStrategyToken[_token]) revert UnrecognizedStrategyToken();
        if (!isPortStrategy[msg.sender][_token]) revert UnrecognizedPortStrategy();
        _;
    }    
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L553
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L559


```solidity
File:  RootBridgeAgent.sol
1226   modifier requiresPort() {
        if (msg.sender != localPortAddress) revert UnrecognizedPort();
        _;
    }

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1226

```solidity
File:  RootPort.sol
557  modifier requiresBridgeAgentFactory() {
        if (!isBridgeAgentFactory[msg.sender]) revert UnrecognizedBridgeAgentFactory();
        _;
    }
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L557




## [G-11] OR in if-condition can be rewritten to two single if conditions

Refactoring the if-condition in a way it won't be containing || operator will save more gas.

```solidity
File:  BranchPort.sol
363     if (_minimumReservesRatio >= DIVISIONER || _minimumReservesRatio < MIN_RESERVE_RATIO) {
            revert InvalidMinimumReservesRatio();
        }

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L363-L365




## [G-12] Use ERC721A instead ERC721

ERC721A standard, ERC721A is an improvement standard for ERC721 tokens.  Compared with ERC721, ERC721A is a more gas-efficient 

```solidity
7  import {ERC721} from "solmate/tokens/ERC721.sol";
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L7



## [G-13] Can Make The Variable Outside The Loop To Save Gas 

When you declare a variable inside a loop, Solidity creates a new instance of the variable for each iteration of the loop. This can lead to unnecessary gas costs, especially if the loop is executed frequently or iterates over a large number of elements.

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
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1076


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



## [G-14] Access mappings directly rather than using accessor functions

Accessing mappings directly instead of using accessor functions can potentially save gas in Solidity contracts. This gas-saving technique is applicable when you only need to read values from a mapping and don't require any additional logic or checks.

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
