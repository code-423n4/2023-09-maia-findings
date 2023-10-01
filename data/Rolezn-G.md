Disclaimer: Several Gas findings now contain an <b>[Enhanced]</b> tag which indicate findings that have enhanced details which include per-instance foundry test gas reports and its gas usage. While some of the instances may be mentioned in the automated report, however, this report provides accurate gas-based test reports in order to help prioritize specific changes that can significantly reduce gas usage.

## GAS Summary<a name="GAS Summary">

### Gas Optimizations
| |Issue|Contexts|Estimated Gas Saved|
|-|:-|:-|:-:|
| [GAS&#x2011;1](#GAS&#x2011;1) | Use assembly in place of `abi.decode` to extract calldata values more efficiently | 2 | 224 |
| [GAS&#x2011;2](#GAS&#x2011;2) | Usage of `uints`/`ints` smaller than 32 bytes (256 bits) incurs overhead <b>[Enhanced]</b> | 4 | 264017 |
| [GAS&#x2011;3](#GAS&#x2011;3) | Use do while loops instead of for loops <b>[Enhanced]</b> | 1 | 6415 |
| [GAS&#x2011;4](#GAS&#x2011;4) | `<array>.length` Should Not Be Looked Up In Every Loop Of A For-loop <b>[Enhanced]</b> | 2 | 210040 |
| [GAS&#x2011;5](#GAS&#x2011;5) | Using XOR (^) and AND (&) bitwise equivalents | 64 | 832 |
| [GAS&#x2011;6](#GAS&#x2011;6) | `address(this)` can be stored in `state variable` that will ultimately cost less, than every time calculating this, specifically when it used multiple times in a `contract` | 32 | - |
| [GAS&#x2011;7](#GAS&#x2011;7) | Gas savings can be achieved by changing the model for assigning value to the structure | 5 | 650 |
| [GAS&#x2011;8](#GAS&#x2011;8) | Using `unchecked` blocks to save gas <b>[Enhanced]</b> | 2 | 1159 |
| [GAS&#x2011;9](#GAS&#x2011;9) | `require()`/`revert()` Strings Longer Than 32 Bytes Cost Extra Gas <b>[Enhanced]</b> | 8 | 207456 |
| [GAS&#x2011;10](#GAS&#x2011;10) | Counting down in `for` statements is more gas efficient | 15 | 3855 |
| [GAS&#x2011;11](#GAS&#x2011;11) | Hash using `assembly` instead of solidity | 1 | 80 |

Total: 161 contexts over 11 issues

## Gas Optimizations


### <a href="#gas-summary">[GAS&#x2011;1]</a><a name="GAS&#x2011;1"> Use assembly in place of `abi.decode` to extract calldata values more efficiently

Instead of using `abi.decode`, we can use assembly to decode our desired calldata values directly. This will allow us to avoid decoding calldata values that we will not use.

For example, for a generic `abi.decode` call:
```solidity
(bytes32 varA, bytes32 varB, uint256 varC, uint256 varD) = abi.decode(metadata, (bytes32, bytes32, uint256, uint256));
```

We can use the following assembly call to extract the relevant metadata:

```solidity
        bytes32 varA;
        bytes32 varB;
        uint256 varC;
        uint256 varD;
        { // used to discard `data` variable and avoid extra stack manipulation
            bytes calldata data = metadata;
            assembly {
                varA := calldataload(add(data.offset, 0x00))
                varB := calldataload(add(data.offset, 0x20))
                varC := calldataload(add(data.offset, 0x40))
                varD := calldataload(add(data.offset, 0x60))
            }
        }
```

#### <ins>Proof Of Concept</ins>

```solidity
File: ArbitrumCoreBranchRouter.sol

158: (address underlyingToken, uint256 minimumReservesRatio) = abi.decode(_data[1:], (address, uint256));

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/ArbitrumCoreBranchRouter.sol#L158

```solidity
File: CoreBranchRouter.sol

128: (address underlyingToken, uint256 minimumReservesRatio) = abi.decode(_params[1:], (address, uint256));

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/CoreBranchRouter.sol#L128






### <a href="#gas-summary">[GAS&#x2011;2]</a><a name="GAS&#x2011;2"> Usage of `uints`/`ints` smaller than 32 bytes (256 bits) incurs overhead

When using elements that are smaller than 32 bytes, your contract's gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.

https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html
Each operation involving a `uint8` costs an extra 22-28 gas (depending on whether the other operand is also a variable of type `uint8`) as compared to ones involving `uint256`, due to the compiler having to clear the higher bits of the memory word before operating on the `uint8`, as well as the associated stack operations of doing so. Use a larger size then downcast where needed

#### <ins>Proof Of Concept</ins>


Optimize to the following in order to decrease a total of $\textcolor{green}{\textsf{-264017}}$ gas usage (includes proof of forge test runs):


```solidity
uint256 public immutable localChainId;
```

https://github.com/code-423n4/2023-09-maia/tree/main/src/RootBridgeAgent.sol#L40


```solidity
uint256 public immutable localChainId;
```

https://github.com/code-423n4/2023-09-maia/tree/main/src/ArbitrumBranchPort.sol#L22


```solidity
uint256 public immutable localChainId;
```

https://github.com/code-423n4/2023-09-maia/tree/main/src/factories/ERC20hTokenBranchFactory.sol#L14


```solidity
uint256 numberOfAssets;
```

https://github.com/code-423n4/2023-09-maia/tree/main/src/interfaces/BridgeAgentStructs.sol#L90

</details>

#### <ins>Forge test</ins>

Running forge test comparison report:

<details>


Ran 9 test suites: 140 tests passed, 0 failed, 0 skipped (140 total tests)
testAddStrategyToken() (gas: -12 (-0.001\%))
 
testManageTwoDayLimits() (gas: -24 (-0.001\%))
 
testReplenishAsUser() (gas: -24 (-0.001\%))
 
testReplenishAsStrategyNotEnoughDebtToRepay() (gas: -24 (-0.001\%))
 
testManage() (gas: -24 (-0.001\%))
 
testReplenishAsStrategy() (gas: -24 (-0.001\%))
 
testRetrySettlementNoFallback() (gas: -154 (-0.001\%))
 
testManageExceedsDailyLimit() (gas: -24 (-0.001\%))
 
testRetryDeposit() (gas: -136 (-0.001\%))
 
testRetrySettlementTriggerFallback() (gas: -160 (-0.001\%))
 
testRetryDepositNotEnoughGasForSettlement() (gas: -136 (-0.001\%))
 
testManageExceedsMinimumReserves() (gas: -24 (-0.001\%))
 
testRedeemSettlement() (gas: -160 (-0.001\%))
 
testRetrySettlement() (gas: -154 (-0.001\%))
 
testAddPortStrategy() (gas: -24 (-0.001\%))
 
testRetrieveSettlement() (gas: -172 (-0.001\%))
 
testCallOutWithDepositWrongCalldataForRootFallbackMode() (gas: -118 (-0.001\%))
 
testRedeemDepositAfterFallback() (gas: -118 (-0.001\%))
 
testRetrieveDeposit() (gas: -129 (-0.001\%))
 
testRedeemDepositAfterRetrieve() (gas: -129 (-0.001\%))
 
testAddLocalToken() (gas: -33 (-0.001\%))
testRemoveStrategyToken() (gas: -24 (-0.001\%))
 
testMulticallNoOutputNoDeposit() (gas: -1 (-0.001\%))
 
testCallOutWithDepositNotEnoughGasForRootFallbackMode() (gas: -123 (-0.001\%))
 
testCallOutWithDepositSuccess() (gas: -123 (-0.001\%))
 
testCallOutWithDepositWrongCalldataForRootRetryMode() (gas: -124 (-0.001\%))
 
testCallOutWithDepositNotEnoughGasForRootRetryMode() (gas: -123 (-0.001\%))
 
testFuzzExecuteWithSettlement(address,uint256,uint256,uint16) (gas: -22 (-0.001\%)) 
testAddGlobalTokenFork() (gas: -84 (-0.001\%))
 
testFuzzFallbackClearDepositRedeem(address,uint256,uint256,uint16) (gas: -22 (-0.001\%)) 
testCallOutWithDepositArbtirum() (gas: -130 (-0.001\%))
 
testAddLocalTokenArbitrum() (gas: -117 (-0.001\%))
 
testAddPortStrategyNotToken() (gas: -12 (-0.001\%))
 
testAddStrategyTokenInvalidMinReserve() (gas: -12 (-0.001\%))
 
testAddLocalToken() (gas: -33 (-0.001\%))
testCallOutWithDeposit() (gas: -46 (-0.002\%))
 
testCallOutWithDepositExecutionFailed() (gas: -40 (-0.002\%))
 
testMulticallSignedSingleOutputDepositMultiple() (gas: 34 (0.002\%))
testMulticallSignedSingleOutputDepositMultiple() (gas: 34 (0.002\%))
testFuzzExecuteWithSettlementMultiple(uint256,uint256,uint256,uint256,uint16) (gas: -76 (-0.002\%)) 
testMulticallSignedMultipleOutputDepositMultiple() (gas: 40 (0.002\%))
testCallOutWithDeposit() (gas: -142 (-0.002\%))
 
testMulticallSignedMultipleOutputDepositMultiple() (gas: 40 (0.002\%))
testRetrySettlement() (gas: -172 (-0.003\%))
 
testRedeemSettlement() (gas: -166 (-0.003\%))
 
testAddLocalTokenArbitrum() (gas: -33 (-0.003\%))
 
testAddGlobalToken() (gas: -96 (-0.003\%))
 
testMulticallSignedNoOutputDepositMultiple() (gas: 46 (0.003\%))
testAddLocalTokenArbitrum() (gas: -129 (-0.003\%))
 
testAddBridgeAgentFactory() (gas: -24 (-0.003\%))
 
testAddLocalToken() (gas: -33 (-0.003\%))
testMulticallSignedNoOutputDepositMultiple() (gas: 46 (0.003\%))
testAddLocalToken() (gas: -33 (-0.003\%))
testDepositToPort() (gas: -39 (-0.003\%))
 
testAddLocalTokenFromArbitrum() (gas: -33 (-0.003\%))
 
testAddLocalTokenNotEnoughGas() (gas: -33 (-0.003\%))
 
testMulticallSignedNoOutputDepositSingleNative() (gas: -64 (-0.003\%))
 
testMulticallSignedNoOutputDepositSingle() (gas: -64 (-0.003\%))
 
testMulticallSignedMultipleOutputDepositSingle() (gas: -64 (-0.003\%))
testMulticallSignedMultipleOutputNoDeposit() (gas: -64 (-0.003\%))
testRetryTwoSettlements() (gas: -215 (-0.003\%))
 
testMulticallMultipleOutputNoDeposit() (gas: -64 (-0.003\%))
testWithdrawFromPortUnknownGlobal() (gas: -42 (-0.003\%))
 
testRedeemTwoSettlements() (gas: -203 (-0.003\%))
 
testMulticallSignedMultipleOutputDepositSingle() (gas: -64 (-0.003\%))
testMulticallSignedNoOutputDepositSingle() (gas: -52 (-0.003\%))
testMulticallSignedMultipleOutputNoDeposit() (gas: -64 (-0.003\%))
testMulticallMultipleOutputNoDeposit() (gas: -64 (-0.003\%))
testFallbackClearDepositRedeemDouble() (gas: -18 (-0.003\%))
 
testMulticallSignedSingleOutputDepositSingle() (gas: -64 (-0.003\%))
testFallbackClearDepositRedeemAlreadyRedeemed() (gas: -18 (-0.003\%))
 
testFallbackClearDepositRedeemSuccess() (gas: -18 (-0.003\%))
 
testMulticallSignedNoOutputDepositSingle() (gas: -52 (-0.003\%))
testMulticallSignedSingleOutputNoDeposit() (gas: -64 (-0.003\%))
 
testMulticallSingleOutputNoDeposit() (gas: -64 (-0.003\%))
 
testWithdrawFromPort() (gas: -45 (-0.003\%))
 
testMulticallSignedSingleOutputDepositSingle() (gas: -64 (-0.003\%))
testMulticallMismatchTokens() (gas: -75 (-0.003\%))
 
testMulticallSignedSingleOutputNoDeposit() (gas: -64 (-0.004\%))
 
testMulticallSingleOutputNoDeposit() (gas: -64 (-0.004\%))
 
testMulticallNoOutputNoDeposit() (gas: -52 (-0.004\%))
testAddGlobalToken() (gas: -45 (-0.004\%))
 
testAddGlobalTokenNotEnoughGas() (gas: -45 (-0.004\%))
 
testRetryDepositFailCanAlwaysRetry() (gas: -22 (-0.004\%))
 
testMulticallNoCodeInTarget() (gas: -58 (-0.004\%))
testMulticallNoOutputNoDeposit() (gas: -52 (-0.004\%))
testRetryDeposit() (gas: -22 (-0.004\%))
 
testRetryDepositFailNotOwner() (gas: -22 (-0.004\%))
 
testSetLocalToken() (gas: -51 (-0.004\%))
 
testCallOutWithDeposit(address,uint256) (gas: -22 (-0.004\%)) 
testCallOutWithDeposit() (gas: -22 (-0.004\%))
 
testCallOutSignedAndBridge(address,uint256) (gas: -22 (-0.004\%)) 
testMulticallNoCodeInTarget() (gas: -58 (-0.004\%))
testRemoveBridgeAgent() (gas: -34 (-0.004\%))
 
testAddGlobalTokenAlreadyAdded() (gas: -57 (-0.004\%))
 
testAddGlobalToken() (gas: -57 (-0.005\%))
 
testSetLocalToken() (gas: -63 (-0.005\%))
 
testRemoveBridgeAgentFactory() (gas: -48 (-0.005\%))
 
testAddLocalTokenAlreadyAdded() (gas: -66 (-0.005\%))
 
testDepositToPortUnrecognized() (gas: -6 (-0.005\%))
 
testAddStrategyToken() (gas: -24 (-0.008\%))
 
testAddPortStrategyNotToken() (gas: -24 (-0.010\%))
 
testAddPortStrategy() (gas: -48 (-0.010\%))
 
testAddStrategyTokenInvalidMinReserve() (gas: -24 (-0.011\%))
 
testMulticallTwoTimesMessage() (gas: -13 (-0.011\%))
 
testRemoveStrategyToken() (gas: -48 (-0.013\%))
 
testRemoveBridgeAgent() (gas: -46 (-0.022\%))
 
testAddLocalTokenArbitrumFailedIsGlobal() (gas: -33 (-0.030\%))
 
testAddBridgeAgentWrongBranchFactory() (gas: -5224 (-0.071\%))
 
testRemoveBridgeAgentFactory() (gas: -4636 (-0.074\%))
 
testAddBridgeAgentWrongRootFactory() (gas: -5256 (-0.075\%))
 
testAddBridgeAgentWrongRootFactory() (gas: -9848 (-0.077\%))
 
testFuzzReadDepositData(uint32,uint256,uint256,uint256,uint256) (gas: -54 (-0.080\%)) 
testAddLocalTokenNewCore() (gas: -18512 (-0.083\%))
 
testAddBridgeAgentWrongBranchFactory() (gas: -5232 (-0.084\%))
 
testAddBridgeAgentFactory() (gas: -4624 (-0.084\%))
 
testAddTwoBridgeAgentFactories() (gas: -9248 (-0.085\%))
 
testAddBridgeAgentTwoTimes() (gas: -12237 (-0.086\%))
 
testAddBridgeAgentNotApproved() (gas: -5212 (-0.087\%))
 
testAddBridgeAgentNotManager() (gas: -5212 (-0.088\%))
 
testAddBridgeAgentNotApproved() (gas: -5208 (-0.088\%))
 
testAddBridgeAgentNotManager() (gas: -5208 (-0.088\%))
 
testAddBridgeAgentNewFactory() (gas: -15061 (-0.088\%))
 
testAddBridgeAgentAlreadyAdded() (gas: -12437 (-0.097\%))
 
testAddBridgeAgentSimple() (gas: -12437 (-0.097\%))
 
testAddBridgeAgentTwice() (gas: -12245 (-0.099\%))
 
testSyncNewCoreBranchRouter() (gas: -18479 (-0.102\%))
 
testSetCoreRootRouter() (gas: -18479 (-0.102\%))
 
testSetBranchRouter() (gas: -18479 (-0.102\%))
 
testAddBridgeAgentFactoryNotRootFactory() (gas: -4617 (-0.103\%))
 
testAddBridgeAgentAlreadyAdded() (gas: -12445 (-0.114\%))
 
testAddBridgeAgent() (gas: -12445 (-0.114\%))
 
testAddBridgeAgentArbitrum() (gas: -12430 (-0.114\%))
 
testAddBridgeAgentArbitrum() (gas: -12426 (-0.115\%))
 
Overall gas change: $\textcolor{green}{\textsf{-264017}}$ (-0.037\%)
</details>




### <a href="#gas-summary">[GAS&#x2011;3]</a><a name="GAS&#x2011;3"> Use do while loops instead of for loops

A do while loop will cost less gas since the condition is not being checked for the first iteration.

#### <ins>Proof Of Concept</ins>

Optimize to the following in order to decrease a total of $\textcolor{green}{\textsf{-6415}}$ gas usage (includes proof of forge test runs):

```solidity
File: BranchPort.sol

246: function bridgeInMultiple

do {
        // Check if hTokens are being bridged in
        if (_amounts[i] - _deposits[i] > 0) {
            unchecked {
                _bridgeIn(_recipient, _localAddresses[i], _amounts[i] - _deposits[i]);
            }
        }

        // Check if underlying tokens are being cleared
        if (_deposits[i] > 0) {
            withdraw(_recipient, _underlyingAddresses[i], _deposits[i]);
        }

        unchecked {
            ++i;
        }
    } while (i < length);
```

https://github.com/code-423n4/2023-09-maia/tree/main/src/BranchPort.sol#L247


#### <ins>Forge test</ins>

Running forge test comparison report:
        
<details>
Ran 9 test suites: 140 tests passed, 0 failed, 0 skipped (140 total tests)
        
testRetryDepositNotEnoughGasForSettlement() (gas: -23 (-0.000\%))

testRetrieveDeposit() (gas: -23 (-0.000\%))

testCallOutWithDepositWrongCalldataForRootFallbackMode() (gas: -23 (-0.000\%))

testCallOutWithDepositWrongCalldataForRootRetryMode() (gas: -23 (-0.000\%))

testCallOutWithDepositNotEnoughGasForRootFallbackMode() (gas: -23 (-0.000\%))

testCallOutWithDepositSuccess() (gas: -23 (-0.000\%))

testCallOutWithDepositNotEnoughGasForRootRetryMode() (gas: -23 (-0.000\%))

testAddBridgeAgentWrongBranchFactory() (gas: -16 (-0.000\%))

testAddBridgeAgentWrongBranchFactory() (gas: -16 (-0.000\%))

testRetrieveSettlement() (gas: -57 (-0.000\%))

testRetrySettlementTriggerFallback() (gas: -57 (-0.000\%))

testRedeemSettlement() (gas: -57 (-0.000\%))

testRetrySettlementNoFallback() (gas: -57 (-0.000\%))

testRetrySettlement() (gas: -57 (-0.000\%))

testRedeemDepositAfterRetrieve() (gas: -45 (-0.000\%))

testRetryDeposit() (gas: -57 (-0.000\%))

testRedeemDepositAfterFallback() (gas: -45 (-0.000\%))

testRedeemSettlement() (gas: -23 (-0.000\%))

testAddBridgeAgentArbitrum() (gas: -53 (-0.000\%))

testAddBridgeAgentArbitrum() (gas: -53 (-0.000\%))

testAddBridgeAgentTwoTimes() (gas: -73 (-0.001\%))

testAddLocalTokenNewCore() (gas: -115 (-0.001\%))

testFuzzExecuteWithSettlementMultiple(uint256,uint256,uint256,uint256,uint16) (gas: -23 (-0.001\%))

testAddBridgeAgentAlreadyAdded() (gas: -73 (-0.001\%))

testAddBridgeAgentSimple() (gas: -73 (-0.001\%))

testAddBridgeAgentTwice() (gas: -73 (-0.001\%))

testCallOutWithDepositArbtirum() (gas: -69 (-0.001\%))

testSyncNewCoreBranchRouter() (gas: -115 (-0.001\%))

testSetCoreRootRouter() (gas: -115 (-0.001\%))

testSetBranchRouter() (gas: -115 (-0.001\%))

testAddBridgeAgentNewFactory() (gas: -112 (-0.001\%))

testAddBridgeAgentAlreadyAdded() (gas: -73 (-0.001\%))

testAddBridgeAgent() (gas: -73 (-0.001\%))

testRedeemTwoSettlements() (gas: -46 (-0.001\%))

testAddBridgeAgentWrongRootFactory() (gas: -90 (-0.001\%))

testRetrySettlement() (gas: -57 (-0.001\%))

testCallOutWithDepositExecutionFailed() (gas: -23 (-0.001\%))

testAddBridgeAgentFactory() (gas: -55 (-0.001\%))

testCallOutWithDeposit() (gas: -69 (-0.001\%))

testAddTwoBridgeAgentFactories() (gas: -126 (-0.001\%))

testAddBridgeAgentWrongRootFactory() (gas: -90 (-0.001\%))

testRetryTwoSettlements() (gas: -114 (-0.002\%))

testRemoveBridgeAgentFactory() (gas: -139 (-0.002\%))

testCallOutWithDeposit() (gas: -69 (-0.002\%))

testAddStrategyToken() (gas: -60 (-0.003\%))

testFuzzExecuteWithSettlement(address,uint256,uint256,uint16) (gas: -79 (-0.004\%))

testRetryDepositFailCanAlwaysRetry() (gas: -23 (-0.004\%))

testRetryDeposit() (gas: -23 (-0.004\%))

testAddPortStrategy() (gas: -114 (-0.004\%))

testRetryDepositFailNotOwner() (gas: -23 (-0.004\%))

testCallOutWithDeposit(address,uint256) (gas: -23 (-0.004\%))

testCallOutWithDeposit() (gas: -23 (-0.004\%))

testCallOutSignedAndBridge(address,uint256) (gas: -23 (-0.004\%))

testFuzzFallbackClearDepositRedeem(address,uint256,uint256,uint16) (gas: -79 (-0.004\%))

testRemoveStrategyToken() (gas: -111 (-0.004\%))

testManageExceedsMinimumReserves() (gas: -132 (-0.004\%))

testAddPortStrategyNotToken() (gas: -46 (-0.005\%))

testManageExceedsDailyLimit() (gas: -160 (-0.005\%))

testAddStrategyTokenInvalidMinReserve() (gas: -52 (-0.006\%))

testRemoveBridgeAgent() (gas: -51 (-0.006\%))

testAddBridgeAgentFactory() (gas: -55 (-0.006\%))

testFallbackClearDepositRedeemDouble() (gas: -36 (-0.007\%))

testFallbackClearDepositRedeemAlreadyRedeemed() (gas: -36 (-0.007\%))

testFallbackClearDepositRedeemSuccess() (gas: -36 (-0.007\%))

testManage() (gas: -207 (-0.007\%))

testReplenishAsStrategyNotEnoughDebtToRepay() (gas: -233 (-0.008\%))

testDepositToPort() (gas: -107 (-0.008\%))

testManageTwoDayLimits() (gas: -300 (-0.010\%))

testReplenishAsStrategy() (gas: -307 (-0.010\%))

testReplenishAsUser() (gas: -322 (-0.010\%))

testWithdrawFromPortUnknownGlobal() (gas: -158 (-0.011\%))

testRemoveBridgeAgentFactory() (gas: -139 (-0.015\%))

testWithdrawFromPort() (gas: -223 (-0.017\%))

testAddPortStrategyNotToken() (gas: -46 (-0.019\%))

testAddStrategyToken() (gas: -60 (-0.020\%))

testAddStrategyTokenInvalidMinReserve() (gas: -52 (-0.024\%))

testAddPortStrategy() (gas: -114 (-0.024\%))

testRemoveBridgeAgent() (gas: -51 (-0.024\%))

testRemoveStrategyToken() (gas: -111 (-0.029\%))

testDepositToPortUnrecognized() (gas: -66 (-0.060\%))

Overall gas change: $\textcolor{green}{\textsf{-6415}}$ (-0.001\%)

</details>



### <a href="#gas-summary">[GAS&#x2011;4]</a><a name="GAS&#x2011;4"> `<array>.length` Should Not Be Looked Up In Every Loop Of A For-loop

The overheads outlined below are PER LOOP, excluding the first loop

storage arrays incur a Gwarmaccess (100 gas)
memory arrays use MLOAD (3 gas)
calldata arrays use CALLDATALOAD (3 gas)

Caching the length changes each of these to a DUP<N> (3 gas), and gets rid of the extra DUP<N> needed to store the stack offset

#### <ins>Proof Of Concept</ins>

Note: While the instances may be present in the automated findings, it is important to note that the automated findings also contain instances that will <b>increase</b> that gas cost. Hence, in order to provide actual valid instances, these are the only findings that will specifically <b>decrease</b> the gas usage

Optimize to the following in order to decrease a total of $\textcolor{green}{\textsf{-210040}}$ gas usage (includes proof of forge test runs):


```solidity
uint256 arrayLoopLength = outputTokens.length;
for (uint256 i = 0; i < arrayLoopLength;) {
```

https://github.com/code-423n4/2023-09-maia/tree/main/src/MulticallRootRouter.sol#L557

```solidity
uint256 arrayLoopLength = hTokens.length;
for (uint256 i = 0; i < arrayLoopLength;) {
```

https://github.com/code-423n4/2023-09-maia/tree/main/src/RootBridgeAgent.sol#L1070

</details>

#### <ins>Forge test</ins>

Running forge test comparison report:

<details>


Ran 9 test suites: 140 tests passed, 0 failed, 0 skipped (140 total tests)
testRetrieveDeposit() (gas: -18 (-0.000\%))
 
testRedeemDepositAfterRetrieve() (gas: -18 (-0.000\%))
 
testCallOutWithDepositWrongCalldataForRootFallbackMode() (gas: -18 (-0.000\%))
 
testRedeemDepositAfterFallback() (gas: -18 (-0.000\%))
 
testCallOutWithDepositWrongCalldataForRootRetryMode() (gas: -18 (-0.000\%))
 
testCallOutWithDepositNotEnoughGasForRootFallbackMode() (gas: -18 (-0.000\%))
 
testCallOutWithDepositSuccess() (gas: -18 (-0.000\%))
 
testCallOutWithDepositNotEnoughGasForRootRetryMode() (gas: -18 (-0.000\%))
 
testCallOutWithDepositArbtirum() (gas: -18 (-0.000\%))
 
testRetryDeposit() (gas: -36 (-0.000\%))
 
testRetryDepositNotEnoughGasForSettlement() (gas: -36 (-0.000\%))
 
testRetrieveSettlement() (gas: -54 (-0.000\%))
 
testRetrySettlementTriggerFallback() (gas: -54 (-0.000\%))
 
testRedeemSettlement() (gas: -54 (-0.000\%))
 
testRetrySettlementNoFallback() (gas: -54 (-0.000\%))
 
testRetrySettlement() (gas: -54 (-0.000\%))
 
testMulticallSignedMultipleOutputDepositSingle() (gas: -22 (-0.001\%))
testMulticallSignedMultipleOutputNoDeposit() (gas: -22 (-0.001\%))
testMulticallMultipleOutputNoDeposit() (gas: -22 (-0.001\%))
testMulticallSignedMultipleOutputDepositSingle() (gas: -22 (-0.001\%))
testMulticallSignedMultipleOutputNoDeposit() (gas: -22 (-0.001\%))
testMulticallMultipleOutputNoDeposit() (gas: -22 (-0.001\%))
testAddLocalTokenNewCore() (gas: -6040 (-0.027\%))
 
testSyncNewCoreBranchRouter() (gas: -6040 (-0.033\%))
 
testSetCoreRootRouter() (gas: -6040 (-0.033\%))
 
testSetBranchRouter() (gas: -6040 (-0.033\%))
 
testAddBridgeAgentAlreadyAdded() (gas: -7649 (-0.059\%))
 
testAddBridgeAgentSimple() (gas: -7649 (-0.059\%))
 
testAddBridgeAgentTwoTimes() (gas: -9266 (-0.065\%))
 
testAddBridgeAgentAlreadyAdded() (gas: -7653 (-0.070\%))
 
testAddBridgeAgent() (gas: -7653 (-0.070\%))
 
testAddBridgeAgentArbitrum() (gas: -7649 (-0.070\%))
 
testAddBridgeAgentArbitrum() (gas: -7653 (-0.071\%))
 
testAddBridgeAgentTwice() (gas: -9262 (-0.075\%))
 
testAddBridgeAgentNewFactory() (gas: -15069 (-0.088\%))
 
testAddBridgeAgentWrongBranchFactory() (gas: -7649 (-0.104\%))
 
testAddBridgeAgentWrongRootFactory() (gas: -7653 (-0.109\%))
 
testAddBridgeAgentWrongRootFactory() (gas: -15069 (-0.118\%))
 
testRemoveBridgeAgentFactory() (gas: -7420 (-0.119\%))
 
testAddBridgeAgentWrongBranchFactory() (gas: -7653 (-0.123\%))
 
testAddBridgeAgentNotApproved() (gas: -7653 (-0.128\%))
 
testAddBridgeAgentNotManager() (gas: -7653 (-0.129\%))
testAddBridgeAgentNotApproved() (gas: -7653 (-0.129\%))
 
testAddBridgeAgentNotManager() (gas: -7653 (-0.129\%))
testAddBridgeAgentFactory() (gas: -7420 (-0.135\%))
 
testAddTwoBridgeAgentFactories() (gas: -14840 (-0.137\%))
 
testAddBridgeAgentFactoryNotRootFactory() (gas: -7425 (-0.165\%))
 
Overall gas change: $\textcolor{green}{\textsf{-210040}}$ (-0.029\%)
</details>




### <a href="#gas-summary">[GAS&#x2011;5]</a><a name="GAS&#x2011;5"> Using XOR (^) and AND (&) bitwise equivalents

Given 4 variables a, b, c and d represented as such:

    0 0 0 0 0 1 1 0 <- a
    0 1 1 0 0 1 1 0 <- b
    0 0 0 0 0 0 0 0 <- c
    1 1 1 1 1 1 1 1 <- d

To have a == b means that every 0 and 1 match on both variables. Meaning that a XOR (operator ^) would evaluate to 0 ((a ^ b) == 0), as it excludes by definition any equalities.Now, if a != b, this means that there’s at least somewhere a 1 and a 0 not matching between a and b, making (a ^ b) != 0.Both formulas are logically equivalent and using the XOR bitwise operator costs actually the same amount of gas.However, it is much cheaper to use the bitwise OR operator (|) than comparing the truthy or falsy values.These are logically equivalent too, as the OR bitwise operator (|) would result in a 1 somewhere if any value is not 0 between the XOR (^) statements, meaning if any XOR (^) statement verifies that its arguments are different.

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: BranchBridgeAgent.sol

127: _lzEndpointAddress != address(0) || _rootChainId == _localChainId,

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/BranchBridgeAgent.sol#L127

```solidity
File: BranchBridgeAgent.sol

412: if (payload.length == 0) revert DepositRetryUnavailableUseCallout();

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/BranchBridgeAgent.sol#L412

```solidity
File: BranchBridgeAgent.sol

439: if (deposit.status == STATUS_SUCCESS) revert DepositRedeemUnavailable();

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/BranchBridgeAgent.sol#L439

```solidity
File: BranchBridgeAgent.sol

440: if (deposit.owner == address(0)) revert DepositRedeemUnavailable();

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/BranchBridgeAgent.sol#L440

```solidity
File: BranchBridgeAgent.sol

599: if (flag == 0x00) {

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/BranchBridgeAgent.sol#L599

```solidity
File: BranchBridgeAgent.sol

616: } else if (flag == 0x01) {

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/BranchBridgeAgent.sol#L616

```solidity
File: BranchBridgeAgent.sol

638: } else if (flag == 0x02) {

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/BranchBridgeAgent.sol#L638

```solidity
File: BranchBridgeAgent.sol

663: } else if (flag == 0x03) {

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/BranchBridgeAgent.sol#L663

```solidity
File: BranchBridgeAgent.sol

681: } else if (flag == 0x04) {

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/BranchBridgeAgent.sol#L681

```solidity
File: CoreRootRouter.sol

278: require(msg.sender == rootPortAddress, "Only root port can call");

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/CoreRootRouter.sol#L278

```solidity
File: CoreRootRouter.sol

307: if (funcId == 0x02) {

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/CoreRootRouter.sol#L307

```solidity
File: CoreRootRouter.sol

314: } else if (funcId == 0x03) {

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/CoreRootRouter.sol#L314

```solidity
File: CoreRootRouter.sol

320: } else if (funcId == 0x04) {

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/CoreRootRouter.sol#L320

```solidity
File: CoreRootRouter.sol

337: if (funcId == 0x01) {

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/CoreRootRouter.sol#L337

```solidity
File: CoreRootRouter.sol

412: if (_dstChainId == rootChainId) revert InvalidChainId();

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/CoreRootRouter.sol#L412

```solidity
File: CoreRootRouter.sol

470: newToken, (_srcChainId == rootChainId) ? newToken : _localAddress, _underlyingAddress, _srcChainId

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/CoreRootRouter.sol#L470

```solidity
File: MulticallRootRouter.sol

142: if (funcId == 0x01) {

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/MulticallRootRouter.sol#L142

```solidity
File: MulticallRootRouter.sol

150: } else if (funcId == 0x02) {

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/MulticallRootRouter.sol#L150

```solidity
File: MulticallRootRouter.sol

174: } else if (funcId == 0x03) {

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/MulticallRootRouter.sol#L174

```solidity
File: MulticallRootRouter.sol

234: if (funcId == 0x01) {

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/MulticallRootRouter.sol#L234

```solidity
File: MulticallRootRouter.sol

242: } else if (funcId == 0x02) {

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/MulticallRootRouter.sol#L242

```solidity
File: MulticallRootRouter.sol

265: } else if (funcId == 0x03) {

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/MulticallRootRouter.sol#L265

```solidity
File: MulticallRootRouter.sol

323: if (funcId == 0x01) {

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/MulticallRootRouter.sol#L323

```solidity
File: MulticallRootRouter.sol

331: } else if (funcId == 0x02) {

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/MulticallRootRouter.sol#L331

```solidity
File: MulticallRootRouter.sol

354: } else if (funcId == 0x03) {

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/MulticallRootRouter.sol#L354

```solidity
File: MulticallRootRouter.sol

411: if (funcId == 0x01) {

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/MulticallRootRouter.sol#L411

```solidity
File: MulticallRootRouter.sol

419: } else if (funcId == 0x02) {

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/MulticallRootRouter.sol#L419

```solidity
File: MulticallRootRouter.sol

442: } else if (funcId == 0x03) {

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/MulticallRootRouter.sol#L442

```solidity
File: BranchPort.sol

123: require(coreBranchRouterAddress == address(0), "Contract already initialized");

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/BranchPort.sol#L123

```solidity
File: BranchPort.sol

181: require(ERC20(_token).balanceOf(address(this)) - currBalance == _amount, "Port Strategy Withdraw Failed");

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/BranchPort.sol#L181

```solidity
File: BranchPort.sol

214: ERC20(_token).balanceOf(address(this)) - currBalance == amountToWithdraw, "Port Strategy Withdraw Failed"

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/BranchPort.sol#L214

```solidity
File: RootPort.sol

245: if (_globalAddress == address(0)) revert InvalidGlobalAddress();

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/RootPort.sol#L245

```solidity
File: RootPort.sol

246: if (_localAddress == address(0)) revert InvalidLocalAddress();

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/RootPort.sol#L246

```solidity
File: RootPort.sol

247: if (_underlyingAddress == address(0)) revert InvalidUnderlyingAddress();

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/RootPort.sol#L247

```solidity
File: RootPort.sol

264: if (_localAddress == address(0)) revert InvalidLocalAddress();

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/RootPort.sol#L264

```solidity
File: RootPort.sol

360: if (_user == address(0)) revert InvalidUserAddress();

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/RootPort.sol#L360

```solidity
File: RootPort.sol

510: if (_coreRootRouter == address(0)) revert InvalidCoreRootRouter();

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/RootPort.sol#L510

```solidity
File: RootPort.sol

511: if (_coreRootBridgeAgent == address(0)) revert InvalidCoreRootBridgeAgent();

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/RootPort.sol#L511

```solidity
File: RootPort.sol

528: if (_coreBranchRouter == address(0)) revert InvalidCoreBranchRouter();

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/RootPort.sol#L528

```solidity
File: RootPort.sol

529: if (_coreBranchBridgeAgent == address(0)) revert InvalidCoreBrancBridgeAgent();

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/RootPort.sol#L529

```solidity
File: RootPort.sol

544: if (_coreBranchRouter == address(0)) revert InvalidCoreBranchRouter();

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/RootPort.sol#L544

```solidity
File: RootPort.sol

545: if (_coreBranchBridgeAgent == address(0)) revert InvalidCoreBrancBridgeAgent();

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/RootPort.sol#L545

```solidity
File: RootBridgeAgent.sol

244: if (settlementReference.owner == address(0)) revert SettlementRetryUnavailable();

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/RootBridgeAgent.sol#L244

```solidity
File: RootBridgeAgent.sol

282: if (settlementOwner == address(0)) revert SettlementRetrieveUnavailable();

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/RootBridgeAgent.sol#L282

```solidity
File: RootBridgeAgent.sol

307: if (settlement.status == STATUS_SUCCESS) revert SettlementRedeemUnavailable();

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/RootBridgeAgent.sol#L307

```solidity
File: RootBridgeAgent.sol

308: if (settlementOwner == address(0)) revert SettlementRedeemUnavailable();

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/RootBridgeAgent.sol#L308

```solidity
File: RootBridgeAgent.sol

430: if (!success) if (msg.sender == getBranchBridgeAgent[localChainId]) revert ExecutionFailure();

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/RootBridgeAgent.sol#L430

```solidity
File: RootBridgeAgent.sol

577: } else if (_payload[0] & 0x7F == 0x05) {

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/RootBridgeAgent.sol#L577

```solidity
File: RootBridgeAgent.sol

617: } else if (_payload[0] & 0x7F == 0x06) {

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/RootBridgeAgent.sol#L617

```solidity
File: RootBridgeAgent.sol

657: } else if (_payload[0] & 0x7F == 0x07) {

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/RootBridgeAgent.sol#L657

```solidity
File: RootBridgeAgent.sol

670: if (settlementReference.owner == address(0)) revert SettlementRetryUnavailable();

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/RootBridgeAgent.sol#L670

```solidity
File: RootBridgeAgent.sol

818: if (callee == address(0)) revert UnrecognizedBridgeAgent();

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/RootBridgeAgent.sol#L818

```solidity
File: RootBridgeAgent.sol

871: if (_hTokens.length == 0) revert SettlementRetryUnavailableUseCallout();

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/RootBridgeAgent.sol#L871

```solidity
File: RootBridgeAgent.sol

877: if (_hTokens.length == 1) {

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/RootBridgeAgent.sol#L877

```solidity
File: RootBridgeAgent.sol

910: if (callee == address(0)) revert UnrecognizedBridgeAgent();

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/RootBridgeAgent.sol#L910

```solidity
File: RootBridgeAgent.sol

1141: if (_amount == 0 && _deposit == 0) revert InvalidInputParams();

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/RootBridgeAgent.sol#L1141

```solidity
File: RootBridgeAgent.sol

1145: if (_localAddress == address(0)) revert UnrecognizedLocalAddress();

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/RootBridgeAgent.sol#L1145

```solidity
File: RootBridgeAgent.sol

1146: if (_underlyingAddress == address(0)) if (_deposit > 0) revert UnrecognizedUnderlyingAddress();

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/RootBridgeAgent.sol#L1146

```solidity
File: ArbitrumBranchPort.sol

63: if (_globalToken == address(0)) revert UnknownGlobalToken();

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/ArbitrumBranchPort.sol#L63

```solidity
File: ArbitrumBranchPort.sol

90: if (_underlyingAddress == address(0)) revert UnknownUnderlyingToken();

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/ArbitrumBranchPort.sol#L90

```solidity
File: BranchBridgeAgentFactory.sol

121: msg.sender == localCoreBranchRouterAddress, "Only the Core Branch Router can create a new Bridge Agent."

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/factories/BranchBridgeAgentFactory.sol#L121

```solidity
File: BranchBridgeAgentFactory.sol

124: _rootBridgeAgentFactoryAddress == rootBridgeAgentFactoryAddress,

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/factories/BranchBridgeAgentFactory.sol#L124

```solidity
File: ArbitrumBranchBridgeAgentFactory.sol

85: msg.sender == localCoreBranchRouterAddress, "Only the Core Branch Router can create a new Bridge Agent."

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/factories/ArbitrumBranchBridgeAgentFactory.sol#L85

```solidity
File: ArbitrumBranchBridgeAgentFactory.sol

88: _rootBridgeAgentFactoryAddress == rootBridgeAgentFactoryAddress,

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/factories/ArbitrumBranchBridgeAgentFactory.sol#L88



</details>


#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized(1,2);
            c1.optimized(1,2);
        }
    }
    
    contract Contract0 {
        function not_optimized(uint8 a,uint8 b) public returns(bool){
            return ((a==1) || (b==1));
        }
    }
    
    contract Contract1 {
        function optimized(uint8 a,uint8 b) public returns(bool){
            return ((a ^ 1) & (b ^ 1)) == 0;
        }
    }

#### Gas Test Report

| Contract0 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 46099                                     | 261             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| not_optimized                             | 456             | 456 | 456    | 456 | 1       |


| Contract1 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 42493                                     | 243             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| optimized                                 | 430             | 430 | 430    | 430 | 1       |





### <a href="#gas-summary">[GAS&#x2011;6]</a><a name="GAS&#x2011;6"> `address(this)` can be stored in `state variable` that will ultimately cost less, than every time calculating this, specifically when it used multiple times in a `contract`

#### <ins>Proof Of Concept</ins>


<details>

```solidity
File: BranchBridgeAgent.sol

142: rootBridgeAgentPath = abi.encodePacked(_rootBridgeAgentAddress, address(this));


```

https://github.com/code-423n4/2023-09-maia/tree/main/src/BranchBridgeAgent.sol#L142

```solidity
File: BranchBridgeAgent.sol

168: address(this),


```

https://github.com/code-423n4/2023-09-maia/tree/main/src/BranchBridgeAgent.sol#L168

```solidity
File: BranchBridgeAgent.sol

579: address(this).excessivelySafeCall(


```

https://github.com/code-423n4/2023-09-maia/tree/main/src/BranchBridgeAgent.sol#L579

```solidity
File: BranchBridgeAgent.sol

737: (bool success,) = bridgeAgentExecutorAddress.call{value: address(this).balance}(_calldata);


```

https://github.com/code-423n4/2023-09-maia/tree/main/src/BranchBridgeAgent.sol#L737

```solidity
File: BranchBridgeAgent.sol

787: ILayerZeroEndpoint(lzEndpointAddress).send{value: address(this).balance}(


```

https://github.com/code-423n4/2023-09-maia/tree/main/src/BranchBridgeAgent.sol#L787

```solidity
File: BranchBridgeAgent.sol

938: if (msg.sender != address(this)) revert LayerZeroUnauthorizedEndpoint();


```

https://github.com/code-423n4/2023-09-maia/tree/main/src/BranchBridgeAgent.sol#L938

```solidity
File: ArbitrumBranchBridgeAgent.sol

126: if (msg.sender != address(this)) revert LayerZeroUnauthorizedEndpoint();


```

https://github.com/code-423n4/2023-09-maia/tree/main/src/ArbitrumBranchBridgeAgent.sol#L126

```solidity
File: VirtualAccount.sol

62: ERC721(_token).transferFrom(address(this), msg.sender, _tokenId);


```

https://github.com/code-423n4/2023-09-maia/tree/main/src/VirtualAccount.sol#L62

```solidity
File: BranchPort.sol

193: uint256 currBalance = ERC20(_token).balanceOf(address(this));

178: IPortStrategy(msg.sender).withdraw(address(this), _token, _amount);

181: require(ERC20(_token).balanceOf(address(this)) - currBalance == _amount, "Port Strategy Withdraw Failed");


```

https://github.com/code-423n4/2023-09-maia/tree/main/src/BranchPort.sol#L193

https://github.com/code-423n4/2023-09-maia/tree/main/src/BranchPort.sol#L178

https://github.com/code-423n4/2023-09-maia/tree/main/src/BranchPort.sol#L181



```solidity
File: BranchPort.sol

193: uint256 currBalance = ERC20(_token).balanceOf(address(this));

210: IPortStrategy(_strategy).withdraw(address(this), _token, amountToWithdraw);

214: ERC20(_token).balanceOf(address(this)) - currBalance == amountToWithdraw, "Port Strategy Withdraw Failed"


```

https://github.com/code-423n4/2023-09-maia/tree/main/src/BranchPort.sol#L193

https://github.com/code-423n4/2023-09-maia/tree/main/src/BranchPort.sol#L210

https://github.com/code-423n4/2023-09-maia/tree/main/src/BranchPort.sol#L214



```solidity
File: BranchPort.sol

436: uint256 currBalance = ERC20(_token).balanceOf(address(this));


```

https://github.com/code-423n4/2023-09-maia/tree/main/src/BranchPort.sol#L436

```solidity
File: BranchPort.sol

526: _localAddress.safeTransferFrom(_depositor, address(this), _hTokenAmount);

532: _underlyingAddress.safeTransferFrom(_depositor, address(this), _deposit);


```

https://github.com/code-423n4/2023-09-maia/tree/main/src/BranchPort.sol#L526

https://github.com/code-423n4/2023-09-maia/tree/main/src/BranchPort.sol#L532



```solidity
File: BaseBranchRouter.sol

167: _hToken.safeTransferFrom(msg.sender, address(this), _amount - _deposit);

174: _token.safeTransferFrom(msg.sender, address(this), _deposit);


```

https://github.com/code-423n4/2023-09-maia/tree/main/src/BaseBranchRouter.sol#L167

https://github.com/code-423n4/2023-09-maia/tree/main/src/BaseBranchRouter.sol#L174



```solidity
File: BranchBridgeAgentExecutor.sol

92: _recipient.safeTransferETH(address(this).balance);


```

https://github.com/code-423n4/2023-09-maia/tree/main/src/BranchBridgeAgentExecutor.sol#L92

```solidity
File: BranchBridgeAgentExecutor.sol

126: _recipient.safeTransferETH(address(this).balance);


```

https://github.com/code-423n4/2023-09-maia/tree/main/src/BranchBridgeAgentExecutor.sol#L126

```solidity
File: RootPort.sol

301: _hToken.safeTransferFrom(_from, address(this), _amount);


```

https://github.com/code-423n4/2023-09-maia/tree/main/src/RootPort.sol#L301

```solidity
File: RootPort.sol

362: newAccount = new VirtualAccount{salt: keccak256(abi.encode(_user))}(_user, address(this));


```

https://github.com/code-423n4/2023-09-maia/tree/main/src/RootPort.sol#L362

```solidity
File: RootBridgeAgent.sol

148: address(this),


```

https://github.com/code-423n4/2023-09-maia/tree/main/src/RootBridgeAgent.sol#L148

```solidity
File: RootBridgeAgent.sol

424: (bool success,) = address(this).excessivelySafeCall(


```

https://github.com/code-423n4/2023-09-maia/tree/main/src/RootBridgeAgent.sol#L424

```solidity
File: RootBridgeAgent.sol

695: address(this).balance


```

https://github.com/code-423n4/2023-09-maia/tree/main/src/RootBridgeAgent.sol#L695

```solidity
File: RootBridgeAgent.sol

779: (bool success,) = bridgeAgentExecutorAddress.call{value: address(this).balance}(_calldata);


```

https://github.com/code-423n4/2023-09-maia/tree/main/src/RootBridgeAgent.sol#L779

```solidity
File: RootBridgeAgent.sol

940: ILayerZeroEndpoint(lzEndpointAddress).send{value: address(this).balance}(


```

https://github.com/code-423n4/2023-09-maia/tree/main/src/RootBridgeAgent.sol#L940

```solidity
File: RootBridgeAgent.sol

1182: getBranchBridgeAgentPath[_branchChainId] = abi.encodePacked(_newBranchBridgeAgent, address(this));


```

https://github.com/code-423n4/2023-09-maia/tree/main/src/RootBridgeAgent.sol#L1182

```solidity
File: ArbitrumBranchPort.sol

66: _underlyingAddress.safeTransferFrom(_depositor, address(this), _deposit);


```

https://github.com/code-423n4/2023-09-maia/tree/main/src/ArbitrumBranchPort.sol#L66

```solidity
File: ArbitrumBranchPort.sol

128: _underlyingAddress.safeTransferFrom(_depositor, address(this), _deposit);


```

https://github.com/code-423n4/2023-09-maia/tree/main/src/ArbitrumBranchPort.sol#L128

```solidity
File: ERC20hTokenRootFactory.sol

83: address(this),


```

https://github.com/code-423n4/2023-09-maia/tree/main/src/factories/ERC20hTokenRootFactory.sol#L83



</details>




### <a href="#gas-summary">[GAS&#x2011;7]</a><a name="GAS&#x2011;7"> Gas savings can be achieved by changing the model for assigning value to the structure

Here's an example to illustrate this:
```solidity
    struct MyStruct {
        uint256 a;
        uint256 b;
        uint256 c;
    }

    function assignValuesToStruct(uint256 _a, uint256 _b, uint256 _c) public {
        MyStruct memory myStruct = MyStruct(_a, _b, _c);
        // Do something with myStruct
    }
```
In this example, we have a simple MyStruct data structure with three uint256 fields. The assignValuesToStruct function takes three input parameters _a, _b, and _c, and assigns them to the corresponding fields in a new myStruct variable. This is done using the struct constructor syntax, which creates a new instance of the MyStruct struct with the specified field values.

The following approach can be more efficient than assigning values to the struct fields:
```solidity
    function assignValuesToStruct(uint256 _a, uint256 _b, uint256 _c) public {
        MyStruct memory myStruct;
        myStruct.a = _a;
        myStruct.b = _b;
        myStruct.c = _c;
        // Do something with myStruct
    }
```
By changing the pattern of assigning value to the structure, gas savings of `~130` per instance are achieved. In addition, this use will provide significant savings in distribution costs.

#### <ins>Proof Of Concept</ins>


<details>

```solidity
File: RootBridgeAgentExecutor.sol

88: DepositParams memory dParams = DepositParams({
            depositNonce: uint32(bytes4(_payload[PARAMS_START:PARAMS_TKN_START])),
            hToken: address(uint160(bytes20(_payload[PARAMS_TKN_START:PARAMS_TKN_START_SIGNED]))),
            token: address(uint160(bytes20(_payload[PARAMS_TKN_START_SIGNED:45]))),
            amount: uint256(bytes32(_payload[45:77])),
            deposit: uint256(bytes32(_payload[77:PARAMS_TKN_SET_SIZE]))
        });


```

https://github.com/code-423n4/2023-09-maia/tree/main/src/RootBridgeAgentExecutor.sol#L88

```solidity
File: RootBridgeAgentExecutor.sol

173: DepositParams memory dParams = DepositParams({
            depositNonce: uint32(bytes4(_payload[PARAMS_START_SIGNED:PARAMS_TKN_START_SIGNED])),
            hToken: address(uint160(bytes20(_payload[PARAMS_TKN_START_SIGNED:45]))),
            token: address(uint160(bytes20(_payload[45:65]))),
            amount: uint256(bytes32(_payload[65:97])),
            deposit: uint256(bytes32(_payload[97:PARAMS_SETTLEMENT_OFFSET]))
        });


```

https://github.com/code-423n4/2023-09-maia/tree/main/src/RootBridgeAgentExecutor.sol#L173

```solidity
File: RootBridgeAgentExecutor.sol

338: dParams = DepositMultipleParams({
            numberOfAssets: numOfAssets,
            depositNonce: nonce,
            hTokens: hTokens,
            tokens: tokens,
            amounts: amounts,
            deposits: deposits
        });


```

https://github.com/code-423n4/2023-09-maia/tree/main/src/RootBridgeAgentExecutor.sol#L338

```solidity
File: BranchBridgeAgentExecutor.sol

72: SettlementParams memory sParams = SettlementParams({
            settlementNonce: uint32(bytes4(_payload[PARAMS_START_SIGNED:PARAMS_TKN_START_SIGNED])),
            recipient: _recipient,
            hToken: address(uint160(bytes20(_payload[PARAMS_TKN_START_SIGNED:45]))),
            token: address(uint160(bytes20(_payload[45:65]))),
            amount: uint256(bytes32(_payload[65:97])),
            deposit: uint256(bytes32(_payload[97:PARAMS_SETTLEMENT_OFFSET]))
        });


```

https://github.com/code-423n4/2023-09-maia/tree/main/src/BranchBridgeAgentExecutor.sol#L72

```solidity
File: RootBridgeAgent.sol

402: DepositParams({
                    hToken: _dParams.hTokens[i],
                    token: _dParams.tokens[i],
                    amount: _dParams.amounts[i],
                    deposit: _dParams.deposits[i],
                    depositNonce: 0
                }),
                _srcChainId
            );


```

https://github.com/code-423n4/2023-09-maia/tree/main/src/RootBridgeAgent.sol#L402



</details>





### <a href="#gas-summary">[GAS&#x2011;8]</a><a name="GAS&#x2011;8"> Using `unchecked` blocks to save gas

Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isn’t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an `unchecked` block

#### <ins>Proof Of Concept</ins>

Optimize to the following in order to decrease a total of $\textcolor{green}{\textsf{-1159}}$ gas usage (includes proof of forge test runs):

```solidity
unchecked {
    getPortStrategyTokenDebt[_strategy][_token] = portStrategyTokenDebt - amountToWithdraw;
}
```

https://github.com/code-423n4/2023-09-maia/tree/main/src/BranchPort.sol#L205

```solidity
unchecked {
    getStrategyTokenDebt[_token] = strategyTokenDebt - amountToWithdraw;
}
```

https://github.com/code-423n4/2023-09-maia/tree/main/src/BranchPort.sol#L207

</details>

#### <ins>Forge test</ins>

Running forge test comparison report:

<details>


Ran 9 test suites: 140 tests passed, 0 failed, 0 skipped (140 total tests)
testRetrieveSettlement() (gas: -5 (-0.000\%))
 
testRetrySettlementTriggerFallback() (gas: -5 (-0.000\%))
 
testRedeemSettlement() (gas: -5 (-0.000\%))
 
testRetrySettlementNoFallback() (gas: -5 (-0.000\%))
 
testRetrySettlement() (gas: -5 (-0.000\%))
testRetryDeposit() (gas: -5 (-0.000\%))
 
testRedeemDepositAfterRetrieve() (gas: -5 (-0.000\%))
 
testRedeemDepositAfterFallback() (gas: -5 (-0.000\%))
 
testAddBridgeAgentWrongBranchFactory() (gas: -5 (-0.000\%))
testRetrySettlement() (gas: -5 (-0.000\%))
testAddBridgeAgentWrongBranchFactory() (gas: -5 (-0.000\%))
testAddLocalTokenNewCore() (gas: -25 (-0.000\%))
 
testSyncNewCoreBranchRouter() (gas: -25 (-0.000\%))
 
testSetCoreRootRouter() (gas: -25 (-0.000\%))
 
testSetBranchRouter() (gas: -25 (-0.000\%))
 
testRetryTwoSettlements() (gas: -10 (-0.000\%))
 
testAddBridgeAgentTwoTimes() (gas: -20 (-0.000\%))
 
testAddBridgeAgentNewFactory() (gas: -25 (-0.000\%))
 
testAddBridgeAgentAlreadyAdded() (gas: -20 (-0.000\%))
testAddBridgeAgentSimple() (gas: -20 (-0.000\%))
 
testAddBridgeAgentWrongRootFactory() (gas: -20 (-0.000\%))
testAddBridgeAgentTwice() (gas: -20 (-0.000\%))
 
testAddBridgeAgentFactory() (gas: -10 (-0.000\%))
 
testAddBridgeAgentAlreadyAdded() (gas: -20 (-0.000\%))
testAddBridgeAgent() (gas: -20 (-0.000\%))
 
testAddTwoBridgeAgentFactories() (gas: -25 (-0.000\%))
 
testAddBridgeAgentWrongRootFactory() (gas: -20 (-0.000\%))
testFuzzExecuteWithSettlement(address,uint256,uint256,uint16) (gas: -10 (-0.000\%)) 
testRemoveBridgeAgentFactory() (gas: -30 (-0.000\%))
 
testFuzzFallbackClearDepositRedeem(address,uint256,uint256,uint16) (gas: -10 (-0.001\%)) 
testFallbackClearDepositRedeemDouble() (gas: -4 (-0.001\%))
 
testFallbackClearDepositRedeemAlreadyRedeemed() (gas: -4 (-0.001\%))
 
testFallbackClearDepositRedeemSuccess() (gas: -4 (-0.001\%))
 
testAddStrategyToken() (gas: -15 (-0.001\%))
 
testManageExceedsDailyLimit() (gas: -30 (-0.001\%))
 
testManageExceedsMinimumReserves() (gas: -30 (-0.001\%))
 
testAddPortStrategy() (gas: -30 (-0.001\%))
 
testManage() (gas: -35 (-0.001\%))
 
testAddBridgeAgentFactory() (gas: -10 (-0.001\%))
 
testRemoveStrategyToken() (gas: -30 (-0.001\%))
 
testManageTwoDayLimits() (gas: -40 (-0.001\%))
 
testReplenishAsStrategyNotEnoughDebtToRepay() (gas: -40 (-0.001\%))
 
testReplenishAsStrategy() (gas: -45 (-0.001\%))
 
testAddPortStrategyNotToken() (gas: -15 (-0.002\%))
 
testAddStrategyTokenInvalidMinReserve() (gas: -15 (-0.002\%))
 
testRemoveBridgeAgent() (gas: -15 (-0.002\%))
 
testRemoveBridgeAgentFactory() (gas: -30 (-0.003\%))
 
testAddStrategyToken() (gas: -15 (-0.005\%))
 
testAddPortStrategyNotToken() (gas: -15 (-0.006\%))
 
testAddPortStrategy() (gas: -30 (-0.006\%))
 
testReplenishAsUser() (gas: -212 (-0.007\%))
 
testAddStrategyTokenInvalidMinReserve() (gas: -15 (-0.007\%))
 
testRemoveBridgeAgent() (gas: -15 (-0.007\%))
 
testRemoveStrategyToken() (gas: -30 (-0.008\%))
 
Overall gas change: $\textcolor{green}{\textsf{-1159}}$ (-0.000\%)
</details>



### <a href="#gas-summary">[GAS&#x2011;9]</a><a name="GAS&#x2011;9"> `require()`/`revert()` Strings Longer Than 32 Bytes Cost Extra Gas

#### <ins>Proof Of Concept</ins>

Note: While the instances may be present in the automated findings, it is important to note that the automated findings also contain instances that will <b>increase</b> that gas cost. Hence, in order to provide actual valid instances, these are the only findings that will specifically <b>decrease</b> the gas usage

Optimize to the following in order to decrease a total of $\textcolor{green}{\textsf{-207456}}$ gas usage (includes proof of forge test runs):

<details><summary>Contains 8 gas optimization instances</summary>

```solidity
require(_rootBridgeAgentAddress != address(0), "shortString");
```

https://github.com/code-423n4/2023-09-maia/tree/main/src/BranchBridgeAgent.sol#L125

```solidity
require(_setup, "shortString");
```

https://github.com/code-423n4/2023-09-maia/tree/main/src/CoreRootRouter.sol#L84

```solidity
require(_bridgeAgentAddress != address(0), "shortString");
```

https://github.com/code-423n4/2023-09-maia/tree/main/src/MulticallRootRouter.sol#L110

```solidity
require(ERC20(_token).balanceOf(address(this)) - currBalance == _amount, "shortString");
```

https://github.com/code-423n4/2023-09-maia/tree/main/src/BranchPort.sol#L181

```solidity
require(_localBridgeAgentAddress != address(0), "shortString");
```

https://github.com/code-423n4/2023-09-maia/tree/main/src/BaseBranchRouter.sol#L61

```solidity
require(_coreRootBridgeAgent != address(0), "shortString");
```

https://github.com/code-423n4/2023-09-maia/tree/main/src/factories/BranchBridgeAgentFactory.sol#L88

```solidity
require(_coreRouter != address(0), "shortString");
```

https://github.com/code-423n4/2023-09-maia/tree/main/src/factories/ERC20hTokenBranchFactory.sol#L61

```solidity
require(_coreRouter != address(0), "shortString");
```

https://github.com/code-423n4/2023-09-maia/tree/main/src/factories/ERC20hTokenRootFactory.sol#L50

</details>

#### <ins>Forge test</ins>

Running forge test comparison report:

<details>


Ran 9 test suites: 140 tests passed, 0 failed, 0 skipped (140 total tests)
testRetrieveSettlement() (gas: -5 (-0.000\%))
 
testRetrySettlementTriggerFallback() (gas: -5 (-0.000\%))
 
testRedeemSettlement() (gas: -5 (-0.000\%))
 
testRetrySettlementNoFallback() (gas: -5 (-0.000\%))
 
testRetrySettlement() (gas: -5 (-0.000\%))
testRetryDeposit() (gas: -5 (-0.000\%))
 
testRedeemDepositAfterRetrieve() (gas: -5 (-0.000\%))
 
testRedeemDepositAfterFallback() (gas: -5 (-0.000\%))
 
testRetrySettlement() (gas: -5 (-0.000\%))
testRetryTwoSettlements() (gas: -10 (-0.000\%))
 
testReplenishAsUser() (gas: 7 (0.000\%))
 
testFuzzExecuteWithSettlement(address,uint256,uint256,uint16) (gas: -10 (-0.000\%)) 
testFuzzFallbackClearDepositRedeem(address,uint256,uint256,uint16) (gas: -10 (-0.001\%)) 
testFallbackClearDepositRedeemDouble() (gas: -4 (-0.001\%))
 
testFallbackClearDepositRedeemAlreadyRedeemed() (gas: -4 (-0.001\%))
 
testFallbackClearDepositRedeemSuccess() (gas: -4 (-0.001\%))
 
testAddStrategyToken() (gas: -15 (-0.001\%))
 
testManageExceedsDailyLimit() (gas: -30 (-0.001\%))
 
testManageExceedsMinimumReserves() (gas: -30 (-0.001\%))
 
testAddPortStrategy() (gas: -30 (-0.001\%))
 
testManage() (gas: -35 (-0.001\%))
 
testRemoveStrategyToken() (gas: -30 (-0.001\%))
 
testManageTwoDayLimits() (gas: -40 (-0.001\%))
 
testReplenishAsStrategyNotEnoughDebtToRepay() (gas: -40 (-0.001\%))
 
testAddPortStrategyNotToken() (gas: -15 (-0.002\%))
 
testAddStrategyTokenInvalidMinReserve() (gas: -15 (-0.002\%))
 
testRemoveBridgeAgent() (gas: -15 (-0.002\%))
 
testReplenishAsStrategy() (gas: -55 (-0.002\%))
 
testAddStrategyToken() (gas: -15 (-0.005\%))
 
testAddPortStrategyNotToken() (gas: -15 (-0.006\%))
 
testAddPortStrategy() (gas: -30 (-0.006\%))
 
testAddStrategyTokenInvalidMinReserve() (gas: -15 (-0.007\%))
 
testRemoveBridgeAgent() (gas: -15 (-0.007\%))
 
testRemoveStrategyToken() (gas: -30 (-0.008\%))
 
testAddBridgeAgentWrongBranchFactory() (gas: -3414 (-0.046\%))
 
testAddBridgeAgentAlreadyAdded() (gas: -6797 (-0.053\%))
 
testAddBridgeAgentSimple() (gas: -6797 (-0.053\%))
 
testAddBridgeAgentNewFactory() (gas: -9204 (-0.054\%))
 
testAddBridgeAgentWrongBranchFactory() (gas: -3418 (-0.055\%))
 
testAddBridgeAgentNotApproved() (gas: -3413 (-0.057\%))
 
testAddBridgeAgentNotManager() (gas: -3413 (-0.057\%))
 
testAddBridgeAgentNotApproved() (gas: -3413 (-0.058\%))
 
testAddBridgeAgentNotManager() (gas: -3413 (-0.058\%))
 
testAddBridgeAgentAlreadyAdded() (gas: -6801 (-0.062\%))
 
testAddBridgeAgent() (gas: -6801 (-0.062\%))
 
testAddBridgeAgentArbitrum() (gas: -6803 (-0.062\%))
 
testAddBridgeAgentArbitrum() (gas: -6807 (-0.063\%))
 
testAddLocalTokenNewCore() (gas: -14183 (-0.064\%))
 
testAddBridgeAgentTwoTimes() (gas: -10214 (-0.072\%))
 
testAddBridgeAgentWrongRootFactory() (gas: -9238 (-0.072\%))
 
testSyncNewCoreBranchRouter() (gas: -14183 (-0.078\%))
 
testSetCoreRootRouter() (gas: -14183 (-0.078\%))
 
testSetBranchRouter() (gas: -14183 (-0.079\%))
 
testAddBridgeAgentTwice() (gas: -10210 (-0.083\%))
 
testRemoveBridgeAgentFactory() (gas: -5839 (-0.093\%))
 
testAddBridgeAgentFactory() (gas: -5819 (-0.106\%))
 
testAddTwoBridgeAgentFactories() (gas: -11643 (-0.107\%))
 
testAddBridgeAgentFactoryNotRootFactory() (gas: -5809 (-0.129\%))
 
testAddBridgeAgentWrongRootFactory() (gas: -9242 (-0.132\%))
 
testRemoveBridgeAgentFactory() (gas: -5843 (-0.614\%))
 
testAddBridgeAgentFactory() (gas: -5823 (-0.669\%))
 
Overall gas change: $\textcolor{green}{\textsf{-207456}}$ (-0.029\%)
</details>



### <a href="#gas-summary">[GAS&#x2011;10]</a><a name="GAS&#x2011;10"> Counting down in `for` statements is more gas efficient

Counting down is more gas efficient than counting up because neither we are making zero variable to non-zero variable and also we will get gas refund in the last transaction when making non-zero to zero variable.

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: BranchBridgeAgent.sol

447: for (uint256 i = 0; i < deposit.tokens.length;) {

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/BranchBridgeAgent.sol#L447

```solidity
File: BranchBridgeAgent.sol

513: for (uint256 i = 0; i < numOfAssets;) {

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/BranchBridgeAgent.sol#L513

```solidity
File: RootBridgeAgentExecutor.sol

281: for (uint256 i = 0; i < uint256(uint8(numOfAssets));) {

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/RootBridgeAgentExecutor.sol#L281

```solidity
File: VirtualAccount.sol

70: for (uint256 i = 0; i < length;) {

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/VirtualAccount.sol#L70

```solidity
File: VirtualAccount.sol

90: for (uint256 i = 0; i < length;) {

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/VirtualAccount.sol#L90

```solidity
File: MulticallRootRouter.sol

278: for (uint256 i = 0; i < outputParams.outputTokens.length;) {

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/MulticallRootRouter.sol#L278

```solidity
File: MulticallRootRouter.sol

367: for (uint256 i = 0; i < outputParams.outputTokens.length;) {

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/MulticallRootRouter.sol#L367

```solidity
File: MulticallRootRouter.sol

455: for (uint256 i = 0; i < outputParams.outputTokens.length;) {

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/MulticallRootRouter.sol#L455

```solidity
File: MulticallRootRouter.sol

557: for (uint256 i = 0; i < outputTokens.length;) {

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/MulticallRootRouter.sol#L557

```solidity
File: BranchPort.sol

257: for (uint256 i = 0; i < length;) {

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/BranchPort.sol#L257

```solidity
File: BranchPort.sol

305: for (uint256 i = 0; i < length;) {

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/BranchPort.sol#L305

```solidity
File: BaseBranchRouter.sol

192: for (uint256 i = 0; i < _hTokens.length;) {

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/BaseBranchRouter.sol#L192

```solidity
File: RootBridgeAgent.sol

318: for (uint256 i = 0; i < settlement.hTokens.length;) {

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/RootBridgeAgent.sol#L318

```solidity
File: RootBridgeAgent.sol

399: for (uint256 i = 0; i < length;) {

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/RootBridgeAgent.sol#L399

```solidity
File: RootBridgeAgent.sol

1070: for (uint256 i = 0; i < hTokens.length;) {

```

https://github.com/code-423n4/2023-09-maia/tree/main/src/RootBridgeAgent.sol#L1070



</details>


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

#### Gas Test Report

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




### <a href="#gas-summary">[GAS&#x2011;11]</a><a name="GAS&#x2011;11"> Hash using `assembly` instead of solidity

Saves about ~80 gas

```solidity
function solidityHash(uint256 a, uint256 b) public view {
    //unoptimized
    keccak256(abi.encodePacked(a, b));
}
```
Gas Test: 313

```solidity
function assemblyHash(uint256 a, uint256 b) public view {
    //optimized
    assembly {
        mstore(0x00, a)
        mstore(0x20, b)
        let hashedVal := keccak256(0x00, 0x40)
    }
} 
```
Gas Test: 231

#### <ins>Proof Of Concept</ins>


```solidity
File: RootPort.sol

362: newAccount = new VirtualAccount{salt: keccak256(abi.encode(_user))}(_user, address(this));


```

https://github.com/code-423n4/2023-09-maia/tree/main/src/RootPort.sol#L362

