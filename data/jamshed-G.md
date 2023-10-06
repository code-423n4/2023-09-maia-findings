## [G-01] Use != 0 instead of > 0

Using `> 0` uses slightly more gas than using `!= 0`. Use `!= 0` when comparing `uint` variables to zero, which cannot hold values below zero.

https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol#L127C9-L132C38

https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L165

https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L173

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L906

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L912

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L266

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L259

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L531

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L531

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L363

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L370

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1142

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1149

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1156

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L284

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L290

## [G-02] Use `selfbalance()` instead of `address(this).balance`

Use assembly when getting a contract's balance of ETH.

You can use `selfbalance()` instead of `address(this).balance` when getting your contract's balance of ETH to save gas. Additionally, you can use `balance(address)` instead of `address.balance()` when getting an external contract's balance of ETH.

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgentExecutor.sol#L92

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgentExecutor.sol#L126

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L737

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L787

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L940

## [G-03] Use hardcode address instead `address(this)`

Instead of using `address(this)`, it is more gas-efficient to pre-calculate and use the hardcoded `address`. Foundry’s script.sol and solmate’s `LibRlp.sol` contracts can help achieve this.

References: <ins>https://book.getfoundry.sh/reference/forge-std/compute-create-address</ins>

<ins>https://twitter.com/transmissions11/status/1518507047943245824</ins>

https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchBridgeAgent.sol#L126

https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol#L66

https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L167

https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L174

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L424

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L441

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L938

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L424

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1205

[https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1233[](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L178)](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1233%5B%5D%28https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L178%29 "https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1233%5B%5D(https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L178)")

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L301

## [G-04] uint8 is not always cheaper than uint256

The EVM only operates on 32 bytes/ 256 bits at a time. This means that if you use uint8, EVM has to first convert it uint256 to work on it and the conversion costs extra gas! You may wonder, What were the devs thinking? Why did they create smaller variables then? The answer lies in packing. In solidity, you can pack multiple small variables into one slot, but if you are defining a lone variable and can’t pack it, it’s optimal to use a uint256 rather than uint8.

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L501

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L64

https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/BridgeAgentConstants.sol#L13C4-L23C48

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L96

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L243

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L359

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L273

https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenBranchFactory.sol#L96[](https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenBranchFactory.sol#L14)

## [G-05] Caching global variables is expensive than using the variable itself

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L115

## [G-06] Use assembly to validate `msg.sender`

We can use assembly to efficiently validate msg.sender for the didPay and uniswapV3SwapCallback functions with the least amount of opcodes necessary. Additionally, we can use xor() instead of iszero(eq()), saving 3 gas. We can also potentially save gas on the unhappy path by using scratch space to store the error selector, potentially avoiding memory expansion costs.

https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchBridgeAgent.sol#L126

https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L207

[https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L354[](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L441)](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L354%5B%5D%28https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L441%29 "https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L354%5B%5D(https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L441)")

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L938

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L954

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L531

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L112

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L278

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L512

https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L605

https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/BranchBridgeAgentFactory.sol#L121

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L570

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L576[](https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ArbitrumBranchBridgeAgentFactory.sol#L85)

## [G-07] `abi.encode()` is less efficient than `abi.encodePacked()`

In terms of efficiency, `abi.encodePacked()` is generally considered to be more gas-efficient than `abi.encode(),` because it skips the step of adding function signatures and other metadata to the encoded data. However, this comes at the cost.

https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumCoreBranchRouter.sol#L53

https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumCoreBranchRouter.sol#L112

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L48

## [G-08] Don't initialize a variable to it's default value

https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L192

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L447

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L513

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L257

## [G-09] Rearranging order of state variable declarations to pack them will save storage slots and gas

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L53C4-L84C32

## [G-10] Empty Blocks Should Be Removed Or Emit Something

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting. If the contract is meant to be extended, the contract should be abstract and the function signatures be added without any default implementation. If the block is an empty if-statement block to avoid doing subsequent checks in the else-if/else conditions, the else-if/else conditions should be nested under the negation of the if-statement, because they involve different classes of checks, which may lead to the introduction of errors when the code is later modified (if(x){}else if(y){…}else{…} => if(!x){if(y){…}else{…}})

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L149

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L128

https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L44[](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L276)

## [G-11 Multiple accesses of a mapping/array should use a storage pointer

Caching a mapping’s value in a storage pointer when the value is accessed multiple times saves ~40 gas per access due to not having to perform the same offset calculation every time. Help the Optimizer by saving a storage variable’s reference instead of repeatedly fetching it.

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L449

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L734

## [G-12]use Mappings Instead of Arrays

Mappings, are useful when you need to store and access data based on a key, rather than an index. Mappings have a lower gas cost for read and write operations, especially when the size of the mapping is large, since Solidity can perform these operations based on the key directly, without needing to iterate over the entire data structure.

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol

https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L32

[https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol "https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#")

https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenBranchFactory.sol#L23

## [G-13] Use bytes32 rather than string/bytes.

If you can fit your data in 32 bytes, then you should use bytes32 datatype rather than bytes or strings as it is much cheaper in solidity. Basically, Any fixed size variable in solidity is cheaper than variable size.

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L96

https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenBranchFactory.sol#L26C5-L29C31

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L308