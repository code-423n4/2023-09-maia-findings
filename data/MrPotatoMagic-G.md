# Gas Optimizations Report

## [G-01] `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables

There are _ instances of this:

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/token/ERC20hTokenRoot.sol#L58

Deployment cost: 945566 - 901647 = 43919 gas saved

Function execution cost: 3489 - 3533 = -44 gas extra 
```solidity
File: src/token/ERC20hTokenRoot.sol
58:  getTokenBalance[chainId] += amount;
```

## [G-02] Keep variable declaration outside for loop to avoid creating new instances on each iteration

There is _ instances of this issue:

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/VirtualAccount.sol#L71

Deployment cost: 766914 - 764914 = 2000 gas saved
```solidity
File: src/VirtualAccount.sol
70: for (uint256 i = 0; i < length;) {
71:     bool success;
```

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/VirtualAccount.sol#L99

Deployment cost: 766914 - 766714 = 200 gas saved
```solidity
File: src/VirtualAccount.sol
99: bool success;
```

## [G-03] Cache `_amount - _deposit` to save gas

There are _ instances of this:

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L284C1-L286C70

Deployment cost: 3337070 - 3337470 = -400 gas extra (including this if team is fine with bearing this little cost during deployment to optimize the code below)

Function execution cost: 38536 - 38444 = 92 gas saved (only 4 runs to offset deployment cost)

Instead of this:
```solidity
File: src/RootPort.sol
284:         if (_amount - _deposit > 0) {
285:             unchecked {
286:                 _hToken.safeTransfer(_recipient, _amount - _deposit);
287:             }
288:         }
```
Use this:
```solidity
File: src/RootPort.sol
283:         uint256 temp = _amount - _deposit;
284:         if (temp > 0) {
285:             unchecked {
286:                 _hToken.safeTransfer(_recipient, temp);
287:             }
288:         }
```

## [G-04] `dParams.amount > 0` check is not required 

There is 1 instance of this:

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L357C1-L374C10

Deployment cost: 5472059 - 5469850 = 2209 gas saved

Function execution cost: 4227 - 4196 = 31 gas saved (per call)

In the code snippet below, the check on Line 357 `dParams.amount < dParams.deposit` ensures amount is greater than deposit and the check on Line 370 `dParams.deposit > 0` ensures deposit is greater than 0. Thus, the check on Line 363 `_dParams.amount > 0` can be removed since deposit is always greater than 0 and amount is always greater than deposit.

Instead of this:
```solidity
File: src/RootBridgeAgent.sol
357:         if (_dParams.amount < _dParams.deposit) revert InvalidInputParams();
358: 
359:         // Cache local port address
360:         address _localPortAddress = localPortAddress;
361: 
362:         // Check local exists.
363:         if (_dParams.amount > 0) {
364:             if (!IPort(_localPortAddress).isLocalToken(_dParams.hToken, _srcChainId)) {
365:                 revert InvalidInputParams();
366:             }
367:         }
368: 
369:         // Check underlying exists.
370:         if (_dParams.deposit > 0) {
371:             if (IPort(_localPortAddress).getLocalTokenFromUnderlying(_dParams.token, _srcChainId) != _dParams.hToken) {
372:                 revert InvalidInputParams();
373:             }
374:         }
```
Use this:
```solidity
File: src/RootBridgeAgent.sol
357:         if (_dParams.amount < _dParams.deposit) revert InvalidInputParams();
358: 
359:         // Cache local port address
360:         address _localPortAddress = localPortAddress;
361: 
362:         // Check local exists.
363:         
364:         if (!IPort(_localPortAddress).isLocalToken(_dParams.hToken, _srcChainId)) {
365:             revert InvalidInputParams();
366:         }
367:         
368: 
369:         // Check underlying exists.
370:         if (_dParams.deposit > 0) {
371:             if (IPort(_localPortAddress).getLocalTokenFromUnderlying(_dParams.token, _srcChainId) != _dParams.hToken) {
372:                 revert InvalidInputParams();
373:             }
374:         }
```

## [G-05] Use do-while loop instead of for-loop to save gas on `executeSigned()` function execution

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/MulticallRootRouter.sol#L557C8-L563C10

Deployment cost: 1790955 - 1802169 = -11214 gas extra (including this if team is fine with bearing this little cost during deployment to optimize the code below)

Function execution cost: 304498 - 304483 = 15 gas saved (approx 747 calls required to offset deployment cost)
Instead of this:
```solidity
File: src/MulticallRootRouter.sol
557:         for (uint256 i = 0; i < outputTokens.length;) {
558:             // Approve Root Port to spend output hTokens.
559:             outputTokens[i].safeApprove(_bridgeAgentAddress, amountsOut[i]);
560:             unchecked {
561:                 ++i;
562:             }
563:         }
```
Use this:
```solidity
File: src/MulticallRootRouter.sol
566:         uint256 i;
567:         do {
568:             outputTokens[i].safeApprove(_bridgeAgentAddress, amountsOut[i]);
569:             unchecked {
570:                 ++i;
571:             }
572:         } while (i < outputTokens.length);
```