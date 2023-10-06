## Summary

Total **27 instances** over **5 issues**with **40213** gas saved:

|ID|Issue|Instances|Gas|
|:--:|:---|:--:|:--:|
| [[G&#x2011;01]](#g01-use-unchecked-block-for-safe-subtractions) | Use `unchecked` block for safe subtractions | 1 | 85 |
| [[G&#x2011;02]](#g02-do-not-cache-state-variables-that-are-used-only-once) | Do not cache state variables that are used only once | 2 | 6 |
| [[G&#x2011;03]](#g03-unlimited-gas-consumption-risk-due-to-external-call-recipients) | Unlimited gas consumption risk due to external call recipients | 13 | - |
| [[G&#x2011;04]](#g04-use-selfbalance-instead-of-addressxbalance) | Use `selfbalance()` instead of `address(x).balance` | 9 | 135 |
| [[G&#x2011;05]](#g05-consider-using-bytes32-rather-than-a-string) | Consider using `bytes32` rather than a `string` | 2 | 40000 |

## Gas Optimizations

### [G&#x2011;01] Use `unchecked` block for safe subtractions


If it can be confirmed that the subtraction operation will not overflow, using an unchecked block can save gas.
For example, `require(x <= y); z = y - x;` can be optimized to `require(x <= y); unchecked { z = y - x; }`.

There is 1 instance:

- *RootBridgeAgent.sol* ( [#L1149](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L1149) ):

```solidity
/// @audit checked on line 1142
1149:         if (_amount - _deposit > 0) {
```

### [G&#x2011;02] Do not cache state variables that are used only once


It's cheaper to access the state variable directly if it is accessed only once. This can save the 3 gas cost of the extra stack allocation.

There are 2 instances:

- *RootPort.sol* ( [#L195](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L195), [#L206](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L206) ):

```solidity
195:         address globalAddress = getGlobalTokenFromLocal[_localAddress][_srcChainId];

206:         address localAddress = getLocalTokenFromGlobal[_globalAddress][_srcChainId];
```

### [G&#x2011;03] Unlimited gas consumption risk due to external call recipients


When calling an external function without specifying a gas limit , the called contract may consume all the remaining gas, causing the tx to be reverted. To mitigate this, it is recommended to explicitly set a gas limit when making low level external calls.

There are 13 instances (click to show):

- *MulticallRootRouter.sol* ( [#L239](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/MulticallRootRouter.sol#L239), [#L248](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/MulticallRootRouter.sol#L248), [#L275](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/MulticallRootRouter.sol#L275), [#L328](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/MulticallRootRouter.sol#L328), [#L337](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/MulticallRootRouter.sol#L337), [#L364](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/MulticallRootRouter.sol#L364), [#L416](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/MulticallRootRouter.sol#L416), [#L425](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/MulticallRootRouter.sol#L425), [#L452](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/MulticallRootRouter.sol#L452) ):

```solidity
239:             IVirtualAccount(userAccount).call(calls);

248:             IVirtualAccount(userAccount).call(calls);

275:             IVirtualAccount(userAccount).call(calls);

328:             IVirtualAccount(userAccount).call(calls);

337:             IVirtualAccount(userAccount).call(calls);

364:             IVirtualAccount(userAccount).call(calls);

416:             IVirtualAccount(userAccount).call(calls);

425:             IVirtualAccount(userAccount).call(calls);

452:             IVirtualAccount(userAccount).call(calls);
```

- *RootBridgeAgent.sol* ( [#L835](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L835), [#L927](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L927) ):

```solidity
835:             callee.call{value: msg.value}("");

927:             callee.call{value: _value}("");
```

- *VirtualAccount.sol* ( [#L74](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/VirtualAccount.sol#L74), [#L101](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/VirtualAccount.sol#L101) ):

```solidity
74:             if (isContract(_call.target)) (success, returnData[i]) = _call.target.call(_call.callData);

101:             if (isContract(_call.target)) (success, returnData[i]) = _call.target.call{value: val}(_call.callData);
```

### [G&#x2011;04] Use `selfbalance()` instead of `address(x).balance`


Use assembly when getting a contract's balance of ETH.
You can use `selfbalance()` instead of `address(x).balance` when getting your contract's balance of ETH to save gas.
Additionally, you can use `balance(address)` instead of `address.balance()` when getting an external contract's balance of ETH.
*Saves 15 gas when checking internal balance, 6 for external*.

There are 9 instances:

- *BranchBridgeAgent.sol* ( [#L717](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L717), [#L737](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L737), [#L787](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L787) ):

```solidity
717:         (bool success,) = bridgeAgentExecutorAddress.call{value: address(this).balance}(_calldata);

737:         (bool success,) = bridgeAgentExecutorAddress.call{value: address(this).balance}(_calldata);

787:         ILayerZeroEndpoint(lzEndpointAddress).send{value: address(this).balance}(
```

- *BranchBridgeAgentExecutor.sol* ( [#L92](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgentExecutor.sol#L92), [#L126](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgentExecutor.sol#L126) ):

```solidity
92:             _recipient.safeTransferETH(address(this).balance);

126:             _recipient.safeTransferETH(address(this).balance);
```

- *RootBridgeAgent.sol* ( [#L695](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L695), [#L754](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L754), [#L779](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L779), [#L940](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L940) ):

```solidity
695:                 address(this).balance

754:         (bool success,) = bridgeAgentExecutorAddress.call{value: address(this).balance}(_calldata);

779:         (bool success,) = bridgeAgentExecutorAddress.call{value: address(this).balance}(_calldata);

940:         ILayerZeroEndpoint(lzEndpointAddress).send{value: address(this).balance}(
```

### [G&#x2011;05] Consider using `bytes32` rather than a `string`


Using the `bytes` types for fixed-length strings is more efficient than having the EVM have to incur the overhead of string processing. Consider whether the value _needs_ to be a `string`. A good reason to keep it as a `string` would be if the variable is defined in an interface that this project does not own.

There are 2 instances:

- *ERC20hTokenBranchFactory.sol* ( [#L26](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/factories/ERC20hTokenBranchFactory.sol#L26), [#L29](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/factories/ERC20hTokenBranchFactory.sol#L29) ):

```solidity
26:     string public chainName;

29:     string public chainSymbol;
```

