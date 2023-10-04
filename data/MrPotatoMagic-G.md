# Gas Optimizations Report

| Gas Optimizations | Issue                                                                                        | Instances |
|-------------------|----------------------------------------------------------------------------------------------|-----------|
| [G-01]            | `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables                       | 1         |
| [G-02]            | Keep variable declaration outside for loop to avoid creating new instances on each iteration | 2         |
| [G-03]            | Cache `_amount - _deposit` to save gas                                                       | 1         |
| [G-04]            | `dParams.amount > 0` check is not required                                                   | 1         |
| [G-05]            | Use do-while loop instead of for-loop to save gas on `executeSigned()` function execution    | 1         |
| [G-06]            | Use `++i` instead of `i++` in unchecked block of for loop                                    | 1         |
| [G-07]            | Zeroing out owner `deposit.owner = address(0);` not required                                 | 1         |
| [G-08]            | Cache out `deposit.owner` from if conditions to save gas                                     | 1         |

### Total issues: 9 instances across 8 issues

### Total deployment cost saved: 41308 gas

### Total function execution cost saved (per call):  21045 gas

### Total gas saved: 62353 gas

(Note: **Finding [G-05]** in this report costs alot of deployment gas, thus it is at the discretion of the sponsor whether to include or ignore the finding)

### Total deployment cost saved (excluding [G-05]): 52522 gas saved

### Total function execution cost saved (excluding [G-05]): 21030 gas saved

### Total gas saved (excluding [G-05]): 73552 gas

## [G-01] `<x> += <y>` costs more gas than `<x> = <x> + <y>` for state variables

**Total gas saved: 43875 gas**

There is 1 instance of this:

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/token/ERC20hTokenRoot.sol#L58

Deployment cost: 945566 - 901647 = 43919 gas saved

Function execution cost: 3489 - 3533 = -44 gas extra 
```solidity
File: src/token/ERC20hTokenRoot.sol
58:  getTokenBalance[chainId] += amount;
```

## [G-02] Keep variable declaration outside for loop to avoid creating new instances on each iteration

**Total gas saved: 2200 gas**

There are 2 instances of this issue:

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

**Total gas saved: -308 gas**

There are 1 instance of this:

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

**Total gas saved: 2240 gas**

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

**Total gas saved: -11199 gas**

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

## [G-06] Use `++i` instead of `i++` in unchecked block of for loop

**Note: This instance is not included in the [[G-10] bot finding](https://gist.github.com/itsmetechjay/9bcacd6e8beea0abaf9590abc6b1d10e#g10-i-costs-less-gas-than-i-especially-when-its-used-in-for-loops---ii---too) and saves more gas per call than what is mentioned in the bot finding.**

**Total gas saved: 20722 gas**

There is 1 instance of this:

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L309

Function execution cost: 97465 - 76743 = 20722 gas saved (per call)
```solidity
File: src/BranchPort.sol
308:          unchecked {
309:              i++;
310:          }
```

## [G-07] Zeroing out owner `deposit.owner = address(0);` not required

**Total gas saved: 3118 gas**

There is 1 instance of this:

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L444

Deployment cost: 3993024 - 3990027 = 2997 gas saved

Function execution cost: 26245 - 26124 = 121 gas saved per call (For higher gas prices, more gas is saved per call)

Line 444 below is not required since in the same function on Line 456, we delete the deposit token info at _depositNonce, which by default deletes `deposit.owner`
```solidity
File: src/BranchBridgeAgent.sol
444: deposit.owner = address(0);
456: delete getDeposit[_depositNonce];
```

## [G-08] Cache out `deposit.owner` from if conditions to save gas

**Total gas saved: 1705 gas**

There is 1 instance of this:

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L440C1-L442C1

Deployment cost: 3993024 - 3991427 = 1597 gas saved

Function execution cost: 26245 - 26137 = 108 gas saved per call (For higher gas prices, more gas is saved per call)

Instead of this:
```solidity
File: src/BranchBridgeAgent.sol
440:       if (deposit.owner == address(0)) revert DepositRedeemUnavailable();
441:       if (deposit.owner != msg.sender) revert NotDepositOwner();
```
Use this:
```solidity
File: src/BranchBridgeAgent.sol
439:       address depositOwner = deposit.owner;
440:       if (depositOwner == address(0)) revert DepositRedeemUnavailable();
441:       if (depositOwner != msg.sender) revert NotDepositOwner();
```