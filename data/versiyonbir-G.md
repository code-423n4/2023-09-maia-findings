1
---
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumCoreBranchRouter.sol#L53-L53
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumCoreBranchRouter.sol#L112-L112
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L48-L48
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L72-L72
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L178-L178
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L226-L226
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L126-L126
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L168-L168
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L193-L193
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L220-L220
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L251-L251
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L281-L281
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L424-L424
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L362-L362
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L471-L471

---
ABI ENCODE IS LESS EFFICIENT THAN ABI ENCODEPACKED
The contract uses abi.encode() in many functions. In abi.encode(), all base types are padded up to 32 bytes and dynamic arrays include their length, whereas abi.encodePacked() uses only the minimum memory needed to encode the data.

Unless explicitly needed , it is recommended to use abi.encodePacked() instead of abi.encode()

2
---
https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L85-L112

---
FUNCTION SHOULD BE EXTERNAL
A function with public visibility modifier was detected that is not called internally.
public and external differs in terms of gas usage. The former use more than the latter when used with large arrays of data. This is due to the fact that Solidity copies arguments to memory on a public function while external read from calldata which a cheaper than memory allocation.
If you know the function you create only allows for external calls, use the external visibility modifier instead of public. It provides performance benefits and you will save on gas.

3
---
https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouterLibZip.sol#L29-L31
https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ArbitrumBranchBridgeAgentFactory.sol#L30-L46
https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenBranchFactory.sol#L42-L49
https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/BranchBridgeAgentFactory.sol#L52-L77
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumCoreBranchRouter.sol#L44-L44
https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/RootBridgeAgentFactory.sol#L31-L37
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L35-L37
https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenRootFactory.sol#L34-L39
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol#L38-L43
https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L92-L100
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L30-L32
https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L48-L50
https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L35-L38
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L108-L111
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgentExecutor.sol#L39-L41
https://github.com/code-423n4/2023-09-maia/blob/main/src/token/ERC20hTokenRoot.sol#L31-L45
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L71-L77
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L111-L118
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L117-L143
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L105-L122
https://github.com/code-423n4/2023-09-maia/blob/main/src/token/ERC20hTokenBranch.sol#L13-L22
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchBridgeAgent.sol#L43-L57

---
DEFINE CONSTRUCTOR AS PAYABLE
Developers can save around 10 opcodes and some gas if the constructors are defined as payable.
However, it should be noted that it comes with risks because payable constructors can accept ETH during deployment.

It is suggested to mark the constructors as payable to save some gas. Make sure it does not lead to any adverse effects in case an upgrade pattern is involved.

4
---
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumCoreBranchRouter.sol#L94-L94
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol#L57-L57
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol#L80-L80
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L208-L208
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L246-L246
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L265-L265
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L286-L286
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L314-L314
https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L162-L162

---
UNNECESSARY MEMORY OPERATIONS WITH AN IMMUTABLE VARIABLE
Immutable variables are different from storage variables. Their instances get replaced in the code with their value. Hence caching operation is unnecessary and costs extra gas.

Immutable variables can be used directly. There is no need for caching them.

5
---
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L100-L100
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L134-L134
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L185-L185
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L220-L222
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol#L127-L127
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol#L132-L132
https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L165-L165
https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L173-L173
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L149-L149
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L299-L299
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L363-L363
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L525-L525
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L531-L531
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgentExecutor.sol#L87-L87
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgentExecutor.sol#L119-L119
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L284-L284
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L290-L290
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L869-L869
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L906-L906
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L912-L912
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L357-L357
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L363-L363
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L370-L370
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L396-L396
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1057-L1057
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1142-L1142
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1149-L1149
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1156-L1156

---
CHEAPER INEQUALITIES IN IF()
The contract was found to be doing comparisons using inequalities inside the if statement.
When inside the if statements, non-strict inequalities (>=, <=) are usually cheaper than the strict equalities (>, <).

It is recommended to go through the code logic, and, if possible, modify the strict inequalities with the non-strict ones to save ~3 gas as long as the logic of the code is not affected.

6
---
https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L557-L563
https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L192-L198
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L447-L453
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L318-L340
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1070-L1086

---
ARRAY LENGTH CACHING
During each iteration of the loop, reading the length of the array uses more gas than is necessary. In the most favorable scenario, in which the length is read from a memory variable, storing the array length in the stack can save about 3 gas per iteration. In the least favorable scenario, in which external calls are made during each iteration, the amount of gas wasted can be significant.

Consider storing the array length of the variable before the loop and use the stored length instead of fetching it in each iteration.

7
---
https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ArbitrumBranchBridgeAgentFactory.sol#L57-L57
https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ArbitrumBranchBridgeAgentFactory.sol#L84-L86
https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ArbitrumBranchBridgeAgentFactory.sol#L87-L90
https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/BranchBridgeAgentFactory.sol#L61-L61
https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/BranchBridgeAgentFactory.sol#L62-L65
https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/BranchBridgeAgentFactory.sol#L66-L66
https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/BranchBridgeAgentFactory.sol#L88-L88
https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/BranchBridgeAgentFactory.sol#L120-L122
https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/BranchBridgeAgentFactory.sol#L123-L126
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L127-L127
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L130-L130
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L131-L131
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L152-L152
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L153-L153
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L154-L154
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L125-L125
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L126-L129
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L130-L130
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L131-L131
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L111-L111
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L112-L112
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L113-L113

---
LONG REQUIRE/REVERT STRINGS
The require() and revert() functions take an input string to show errors if the validation fails.
This strings inside these functions that are longer than 32 bytes require at least one additional MSTORE, along with additional overhead for computing memory offset, and other parameters.

It is recommended to short the strings passed inside require() and revert() to fit under 32 bytes. This will decrease the gas usage at the time of deployment and at runtime when the validation condition is met.

8
---
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol#L127-L127
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol#L132-L132
https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L165-L165
https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L173-L173
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L525-L525
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L531-L531
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L284-L284
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L290-L290
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L906-L906
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L912-L912
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L363-L363
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L370-L370
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1146-L1146
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1149-L1149
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1156-L1156

---
CHEAPER CONDITIONAL OPERATORS
During compilation, x != 0 is cheaper than x > 0 for unsigned integers in solidity inside conditional statements.

Consider using x != 0 in place of x > 0 in uint wherever possible.

9
---
https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L124-L124
https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L204-L204
https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L210-L210
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L136-L136
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L356-L356
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L366-L366
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L371-L371
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L381-L381
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L391-L391
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L167-L167

---
CUSTOM ERRORS TO SAVE GAS
The contract was found to be using revert() statements. Since Solidity v0.8.4, custom errors have been introduced which are a better alternative to the revert.
This allows the developers to pass custom errors with dynamic data while reverting the transaction and also making the whole implementation a bit cheaper than using revert.

It is recommended to replace all the instances of revert() statements with error() to save gas.


10
---
https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ArbitrumBranchBridgeAgentFactory.sol#L6-L6
https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/RootBridgeAgentFactory.sol#L7-L7
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L7-L7
https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenRootFactory.sol#L6-L6
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L9-L9
https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L10-L10
https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L11-L11

---
UNUSED IMPORTS
Solidity is a Gas-constrained language. Having unused code or import statements incurs extra gas usage when deploying the contract.

It is recommended to remove the import statement if it’s not supposed to be used.

11
---
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L32-L32
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L42-L42
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L52-L52
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L58-L58
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L61-L61
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L68-L68
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L74-L74
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L77-L77
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L80-L80
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L83-L84
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L54-L54
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L64-L64
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L77-L77
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L87-L87

---
OPTIMIZING ADDRESS ID MAPPING
Combining multiple address/ID mappings into a single mapping using a struct enhances storage efficiency, simplifies code, and reduces gas costs, resulting in a more streamlined and cost-effective smart contract design.
It saves storage slot for the mapping and depending on the circumstances and sizes of types, it can avoid a Gsset (20000 gas) per mapping combined. Reads and subsequent writes can also be cheaper when a function requires both values and they fit in the same storage slot.

It is suggested to modify the code so that multiple mappings using the address->id parameter are combined into a struct.

12
---
https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenBranchFactory.sol#L17-L17
https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/BranchBridgeAgentFactory.sol#L33-L33
https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/RootBridgeAgentFactory.sol#L16-L16
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol#L22-L22
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L25-L25
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L42-L42
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L58-L58
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L74-L74
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L32-L32
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L25-L25
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L42-L42
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L32-L32
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L52-L52
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L68-L68
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L77-L77
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L83-L84
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L44-L44
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L51-L51
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L24-L24
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L27-L27
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L54-L54
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L64-L64
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L77-L77
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L40-L40
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L61-L61
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L34-L34
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L87-L87
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L87-L87
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L70-L70
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L94-L94
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L74-L74
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L49-L49
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L78-L78
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L46-L46
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L85-L85

---
STORAGE VARIABLE CACHING IN MEMORY
SLOADs are expensive (100 gas after the 1st one) compared to MLOAD/MSTORE (3 gas each).
Storage variables read multiple times inside a function should instead be cached in the memory the first time (costing 1 SLOAD) and then read from this cache to avoid multiple SLOADs.
