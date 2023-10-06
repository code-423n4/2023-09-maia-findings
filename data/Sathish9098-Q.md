# QA FINDINGS

##

## [L-1] The ``_checkTimeLimit`` function reverts unexpected way if ``amount`` exceeds ``dailyLimit``

### IMPACT
The ``_checkTimeLimit`` function reverts unexpectedly when ``amount`` exceeds ``dailyLimit`` indicates that there might be a problem or an unexpected behavior in the ``_checkTimeLimit`` function when the value of the ``amount`` surpasses the defined ``dailyLimit``. When assigning values to ``strategyDailyLimitRemaining[msg.sender][_token]`` state variable. If the ``amount`` exceeds the ``dailyLimit`` the result is negative value. So the over all function may be reverted 

### POC

```solidity
FILE: 2023-09-maia/src/BranchPort.sol

 function _checkTimeLimit(address _token, uint256 _amount) internal {
        uint256 dailyLimit = strategyDailyLimitRemaining[msg.sender][_token];
        if (block.timestamp - lastManaged[msg.sender][_token] >= 1 days) {
            dailyLimit = strategyDailyLimitAmount[msg.sender][_token];
            unchecked {
                lastManaged[msg.sender][_token] = (block.timestamp / 1 days) * 1 days;
            }
        }
        strategyDailyLimitRemaining[msg.sender][_token] = dailyLimit - _amount;
    }

```
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L485-L494

##

## [L-2] ``_minimumReserves()`` Function Returns 0 for ``Non-StrategyTokens``

### Impact
_minimumReserves() always returns return value as zero if the given token is not a ``StrategyToken ``

### POC

```solidity
FILE: 2023-09-maia/src/BranchPort.sol

468: function _minimumReserves(uint256 _strategyTokenDebt, uint256 _currBalance, address _token)
469:        internal
470:        view
471:        returns (uint256)
472:    {
473:        return ((_currBalance + _strategyTokenDebt) * getMinimumTokenReserveRatio[_token]) / DIVISIONER;
474:    }

```
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L468-L474

### Recommended Mitigation

Add if condition to check getMinimumTokenReserveRatio[_token] . If the value is 0 then directly return 0

```diff

+ if(getMinimumTokenReserveRatio[_token] ==0){
+ return 0;
+ }

```
##

## [L-3] Overflow could occur over time as the ``depositNonce`` is incremented with each function call

### Impact 

Functions like callOutSystem(),callOut(),callOutSigned() all functions incrementing the  ``depositNonce`` for every function call. 

The ``depositNonce`` used for encoding data in the callOutSystem(),callOut(),callOutSigned() function may eventually overflow, given that it's of type ``uint32``.

An overflow in the ``depositNonce`` could lead to unexpected behavior and potentially disrupt the ``protocol's functionality``

### POC

```solidity
FILE: 2023-09-maia/src/BranchBridgeAgent.sol

180: bytes memory payload = abi.encodePacked(bytes1(0x00), depositNonce++, _params);

220: bytes memory payload = abi.encodePacked(bytes1(0x01), depositNonce++, _params);

269: bytes memory payload = abi.encodePacked(bytes1(0x04), msg.sender, depositNonce++, _params);

```
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L188

### Recommended Mitigation
Use uint256 for ``depositNonce `` variable 

##

## [L-4] ``updatePortStrategy()`` not check the ``isPortStrategy[_portStrategy][_token]`` is allowed are not 

### Impact
``updatePortStrategy`` function allows updating the ``strategyDailyLimitAmount`` even if ``isPortStrategy[_portStrategy][_token]`` is set to ``false``, which might not align with the intended behavior of the protocol.

### POC

```solidity
FILE: 2023-09-maia/src/BranchPort.sol

 function togglePortStrategy(address _portStrategy, address _token) external override requiresCoreRouter {
        isPortStrategy[_portStrategy][_token] = !isPortStrategy[_portStrategy][_token];

        emit PortStrategyToggled(_portStrategy, _token);
    }

function updatePortStrategy(address _portStrategy, address _token, uint256 _dailyManagementLimit)
        external
        override
        requiresCoreRouter
    {
        strategyDailyLimitAmount[_portStrategy][_token] = _dailyManagementLimit;

        emit PortStrategyUpdated(_portStrategy, _token, _dailyManagementLimit);
    }

```
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L396-L400

### Recommended Mitigation
Add check to ensure ``isPortStrategy[_portStrategy][_token]`` is not false 

```diff

function updatePortStrategy(address _portStrategy, address _token, uint256 _dailyManagementLimit)
        external
        override
        requiresCoreRouter
    {
        // Check if the strategy is allowed for the given token
+        require(isPortStrategy[_portStrategy][_token], "Strategy not allowed for the token");

        strategyDailyLimitAmount[_portStrategy][_token] = _dailyManagementLimit;

        emit PortStrategyUpdated(_portStrategy, _token, _dailyManagementLimit);
    }

```

## 

## [L-5] Avoid double casting

### Impact
Consider refactoring the following code, as double casting may introduce unexpected truncations and/or rounding issues.

Furthermore, double type casting can make the code less readable and harder to maintain, increasing the likelihood of errors and misunderstandings during development and debugging.

```POC
FILE: 2023-09-maia/src/RootBridgeAgentExecutor.sol

125:  PARAMS_END_OFFSET + uint256(uint8(bytes1(_payload[PARAMS_START]))) * PARAMS_TKN_SET_SIZE_MULTIPLE

213:  + uint256(uint8(bytes1(_payload[PARAMS_START_SIGNED]))) * PARAMS_TKN_SET_SIZE_MULTIPLE //@audit double casting

222:  + uint256(uint8(bytes1(_payload[PARAMS_START_SIGNED]))) * PARAMS_TKN_SET_SIZE_MULTIPLE 

281: for (uint256 i = 0; i < uint256(uint8(numOfAssets));) { 

```
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgentExecutor.sol#L125

##

##

## [L-6] Lack of sanity checks in place when assigning values to critical variables, such as ``immutable variables``

### Impact
It is important to perform sanity checks when assigning values to unit-to-critical variables, such as immutable variables and new deployments. This is because errors in these variables can have serious consequences, such as leading to the loss of funds or the disruption of critical services.

```solidity
FILE: 2023-09-maia/src/ArbitrumBranchPort.sol

constructor(uint16 _localChainId, address _rootPortAddress, address _owner) BranchPort(_owner) {
        require(_rootPortAddress != address(0), "Root Port Address cannot be 0");

        localChainId = _localChainId;
        rootPortAddress = _rootPortAddress;
    }

```
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/ArbitrumBranchPort.sol#L38-L43

```solidity
FILE: 2023-09-maia/src/MulticallRootRouter.sol

constructor(uint256 _localChainId, address _localPortAddress, address _multicallAddress) {
        require(_localPortAddress != address(0), "Local Port Address cannot be 0");
        require(_multicallAddress != address(0), "Multicall Address cannot be 0");

        localChainId = _localChainId;
        localPortAddress = _localPortAddress;
        multicallAddress = _multicallAddress;
        _initializeOwner(msg.sender);
    }

```
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/MulticallRootRouter.sol#L96

```solidity
FILE: 2023-09-maia/src/ArbitrumBranchBridgeAgent.sol

constructor(
        uint16 _localChainId,
        address _rootBridgeAgentAddress,
        address _localRouterAddress,
        address _localPortAddress
    )
        BranchBridgeAgent(
            _localChainId,
            _localChainId,
            _rootBridgeAgentAddress,
            address(0),
            _localRouterAddress,
            _localPortAddress
        )
    {}

```
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/ArbitrumBranchBridgeAgent.sol#L43-L57

### Recommended Mitigation
Add necessary checks like ``> 0`` ,``> MIN`` , ``< MAX``

```solidity
require(_localChainId > 0, Wrong chainid) ;

```

##

## [L-7] The function name "_checkTimeLimit" conflicts with its implementation

The name "_checkTimeLimit" implies that the function is primarily responsible for checking a time limit but doesn't convey that it also updates state values

### POC
```solidity
FILE: 2023-09-maia/src/BranchPort.sol

function _checkTimeLimit(address _token, uint256 _amount) internal {
        uint256 dailyLimit = strategyDailyLimitRemaining[msg.sender][_token];
        if (block.timestamp - lastManaged[msg.sender][_token] >= 1 days) {
            dailyLimit = strategyDailyLimitAmount[msg.sender][_token];
            unchecked {
                lastManaged[msg.sender][_token] = (block.timestamp / 1 days) * 1 days;
            }
        }
        strategyDailyLimitRemaining[msg.sender][_token] = dailyLimit - _amount;
    }

```
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L485-L494

### Recommended Mitigation
Modify the function name

```diff

+ function _updateAndCheckTimeLimit(address _token, uint256 _amount) internal {

```
##

## [L-8] Division/multiplication in ``unchecked`` block is unsafe

The additions/multiplications may silently overflow because they're in `unchecked` blocks with no preceding value checks, which may lead to unexpected results

```solidity
FILE: 2023-09-maia/src/BranchPort.sol

489: unchecked {
490:                lastManaged[msg.sender][_token] = (block.timestamp / 1 days) * 1 days;
491:            }

```
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L489-L491

##

## [L-9] Loss of precision 

In the expression lastManaged[msg.sender][_token] = (block.timestamp / 1 days) * 1 days;, there is a possibility of precision loss due to integer division.

The expression block.timestamp / 1 days performs integer division, which means it truncates the fractional part of the result. Since block.timestamp is a timestamp in seconds, dividing it by 1 days (which is also in seconds) will give you the number of full days that have passed since the timestamp.

However, when you multiply this result by 1 days again, it effectively sets the timestamp to the start of the day (i.e., midnight) on the day of the original timestamp. Any fractional seconds from the original block.timestamp are discarded, leading to precision loss.

																																								
```solidity
FILE: 2023-09-maia/src/BranchPort.sol

489: unchecked {
490:                lastManaged[msg.sender][_token] = (block.timestamp / 1 days) * 1 days;
491:            }

```
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L489-L491

##

## [L-10] Missing disallowlist for ERC721

Lately, there has been a rise in cases where NFTs are being stolen. These stolen NFTs are then added to platforms, allowing them to be easily converted into liquidity.

Some popular marketplaces, such as Opensea, have already taken steps to combat this issue by introducing a disallowlist feature. This means that if an NFT is reported as stolen, it won't be listed or sold on their platform.

This may increase the project centralization, but it can help solve this problem, if this issue is a concern.

```
File: src/VirtualAccount.sol

import {ERC721} from "solmate/tokens/ERC721.sol";

```

https://github.com/code-423n4/2023-09-maia/blob/6c21f6b739252367c5fe770fed3ff76eb3ca2c86/src/VirtualAccount.sol#L7	

##

## [L-11] Using zero as a parameter

Taking zero as a valid argument without checks can lead to severe security issues in some cases.

If using a zero argument is mandatory, consider using descriptive constants or an enum instead of passing zero directly on function calls, as that might be error-prone, to fully describe the caller's intention.

```solidity
FILE: 2023-09-maia/src/RootBridgeAgent.sol

837: IBranchBridgeAgent(callee).lzReceive(0, "", 0, _payload); //@audit 0 as parameter

933: IBranchBridgeAgent(callee).lzReceive(0, "", 0, payload); //@audit 0 as parameter



```
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L837	

##

## [L-12] Enum values should be used instead of constant array indexes

Create a commented enum value to use instead of constant array indexes, this makes the code far easier to understand.

```solidity
FILE: 2023-09-maia/src/BranchBridgeAgent.sol

  deposit.hTokens[0],
                    deposit.tokens[0],
                    deposit.amounts[0],
                    deposit.deposits[0],

376:  deposit.hTokens[0],
377:                    deposit.tokens[0],
378:                    deposit.amounts[0],
379:                    deposit.deposits[0],


```																						
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L366C40-L369

##

## [L-13] Vulnerable versions of packages are being used

This project is using specific package versions which are vulnerable to the specific CVEs listed below. Consider switching to more recent versions of these packages that don't have these vulnerabilities.

```IERC1155Receiver`` uses the vulnerable version ``v4.5.0``

```
// SPDX-License-Identifier: MIT
// OpenZeppelin Contracts (last updated v4.5.0) (token/ERC1155/IERC1155Receiver.sol)

pragma solidity ^0.8.0;

import "../../utils/introspection/IERC165.sol";

/**
 * @dev _Available since v3.1._
 */
interface IERC1155Receiver is IERC165 {

``
``IERC721Receiver `` contract uses ``v4.6.0``

```

// SPDX-License-Identifier: MIT
// OpenZeppelin Contracts (last updated v4.6.0) (token/ERC721/IERC721Receiver.sol)

pragma solidity ^0.8.0;

/**
 * @title ERC721 token receiver interface
 * @dev Interface for any contract that wants to support safeTransfers
 * from ERC721 asset contracts.
 */
interface IERC721Receiver {

```

### Recommended Mitigation
Use at least ``OpenZeppelin v4.9.2``

##

## [L-14] Governance functions should be controlled by time locks

Governance functions (such as upgrading contracts, setting critical parameters) should be controlled using time locks to introduce a delay between a proposal and its execution. This gives users time to exit before a potentially dangerous or malicious operation is applied.

```solidity

File: src/BranchPort.sol

331      function setCoreRouter(address _newCoreRouter) external override requiresCoreRouter {
332          require(coreBranchRouterAddress != address(0), "CoreRouter address is zero");
333          require(_newCoreRouter != address(0), "New CoreRouter address is zero");
334          coreBranchRouterAddress = _newCoreRouter;
335:     }

362      function addStrategyToken(address _token, uint256 _minimumReservesRatio) external override requiresCoreRouter {
363          if (_minimumReservesRatio >= DIVISIONER || _minimumReservesRatio < MIN_RESERVE_RATIO) {
364              revert InvalidMinimumReservesRatio();
365          }
366  
367          strategyTokens.push(_token);
368          getMinimumTokenReserveRatio[_token] = _minimumReservesRatio;
369          isStrategyToken[_token] = true;
370  
371          emit StrategyTokenAdded(_token, _minimumReservesRatio);
372:     }

382      function addPortStrategy(address _portStrategy, address _token, uint256 _dailyManagementLimit)
383          external
384          override
385          requiresCoreRouter
386      {
387          if (!isStrategyToken[_token]) revert UnrecognizedStrategyToken();
388          portStrategies.push(_portStrategy);
389          strategyDailyLimitAmount[_portStrategy][_token] = _dailyManagementLimit;
390          isPortStrategy[_portStrategy][_token] = true;
391  
392          emit PortStrategyAdded(_portStrategy, _token, _dailyManagementLimit);
393:     }

403      function updatePortStrategy(address _portStrategy, address _token, uint256 _dailyManagementLimit)
404          external
405          override
406          requiresCoreRouter
407      {
408          strategyDailyLimitAmount[_portStrategy][_token] = _dailyManagementLimit;
409  
410          emit PortStrategyUpdated(_portStrategy, _token, _dailyManagementLimit);
411:     }

414      function setCoreBranchRouter(address _coreBranchRouter, address _coreBranchBridgeAgent)
415          external
416          override
417          requiresCoreRouter
418      {
419          coreBranchRouterAddress = _coreBranchRouter;
420          isBridgeAgent[_coreBranchBridgeAgent] = true;
421          bridgeAgents.push(_coreBranchBridgeAgent);
422  
423          emit CoreBranchSet(_coreBranchRouter, _coreBranchBridgeAgent);
424:     }

```
https://github.com/code-423n4/2023-09-maia/blob/6c21f6b739252367c5fe770fed3ff76eb3ca2c86/src/BranchPort.sol#L331-L335

```solidity
FILE: File: src/RootPort.sol

259      function setLocalAddress(address _globalAddress, address _localAddress, uint256 _srcChainId)
260          external
261          override
262          requiresCoreRootRouter
263      {
264          if (_localAddress == address(0)) revert InvalidLocalAddress();
265  
266          getGlobalTokenFromLocal[_localAddress][_srcChainId] = _globalAddress;
267          getLocalTokenFromGlobal[_globalAddress][_srcChainId] = _localAddress;
268  
269          emit GlobalTokenAdded(_localAddress, _globalAddress, _srcChainId);
270:     }

382      function addBridgeAgent(address _manager, address _bridgeAgent) external override requiresBridgeAgentFactory {
383          if (isBridgeAgent[_bridgeAgent]) revert AlreadyAddedBridgeAgent();
384  
385          bridgeAgents.push(_bridgeAgent);
386          getBridgeAgentManager[_bridgeAgent] = _manager;
387          isBridgeAgent[_bridgeAgent] = true;
388  
389          emit BridgeAgentAdded(_bridgeAgent, _manager);
390:     }

483      function addEcosystemToken(address _ecoTokenGlobalAddress) external override onlyOwner {
484          // Check if token already added
485          if (isGlobalAddress[_ecoTokenGlobalAddress]) revert AlreadyAddedEcosystemToken();
486  
487          // Check if token is already a underlying token in current chain
488          if (getUnderlyingTokenFromLocal[_ecoTokenGlobalAddress][localChainId] != address(0)) {
489              revert AlreadyAddedEcosystemToken();
490          }
491  
492          // Check if token is already a local branch token in current chain
493          if (getLocalTokenFromUnderlying[_ecoTokenGlobalAddress][localChainId] != address(0)) {
494              revert AlreadyAddedEcosystemToken();
495          }
496  
497          // Update State
498          // 1. Add new global token to global address mapping
499          isGlobalAddress[_ecoTokenGlobalAddress] = true;
500          // 2. Add new branch local token address to global token mapping
501          getGlobalTokenFromLocal[_ecoTokenGlobalAddress][localChainId] = _ecoTokenGlobalAddress;
502          // 3. Add new global token to branch local token address mapping
503          getLocalTokenFromGlobal[_ecoTokenGlobalAddress][localChainId] = _ecoTokenGlobalAddress;
504  
505          emit EcosystemTokenAdded(_ecoTokenGlobalAddress);
506:     }

509      function setCoreRootRouter(address _coreRootRouter, address _coreRootBridgeAgent) external override onlyOwner {
510          if (_coreRootRouter == address(0)) revert InvalidCoreRootRouter();
511          if (_coreRootBridgeAgent == address(0)) revert InvalidCoreRootBridgeAgent();
512  
513          coreRootRouterAddress = _coreRootRouter;
514          coreRootBridgeAgentAddress = _coreRootBridgeAgent;
515          getBridgeAgentManager[_coreRootBridgeAgent] = owner();
516  
517          emit CoreRootSet(_coreRootRouter, _coreRootBridgeAgent);
518:     }

```
https://github.com/code-423n4/2023-09-maia/blob/6c21f6b739252367c5fe770fed3ff76eb3ca2c86/src/RootPort.sol#L259-L270

																	
																		


																																															










																							
																										






																																																																																																																																						

																																																																				