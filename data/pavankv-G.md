## 1. Refactor for-loop to save some runtime gas :-

In below code snippet we can see for each loop iteration numOfAssets which already declared in line [273](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L273) as uint8 , but in loop again try to convert to uint8 and again to uint256 bits . So for-loop requires only constant value. So instead of conversion in every loop iteration we can declare variable outside loop and try to call. Which save some conversion operation.

**Before**
```solidity
for (uint256 i = 0; i < uint256(uint8(numOfAssets));) //@audit
```

**After**
```solidity
// 1 Either this way

uint256 numOfAssets1 = uint256(numOfAssets);//@audit changed here
for (uint256 i = 0; i < numOfAssets1) { //@audit changed here


// 2. Or this way 

//Here we directly call uint8 numOfAssets in for-loop and conversion to uint256 .

for (uint256 i = 0; i < uint256(numOfAssets1)) { //@audit changed here


```
1 way is gas-efficient in nature .

Look into `//@audit changed here` comment

code snippet:-
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L273C1-L281C64


## 2. Use do-while loop for simple operation :-
Scenarion 1 :-

**Before**
```solidity
        for (uint256 i = 0; i < deposit.tokens.length;) {
            _clearToken(msg.sender, deposit.hTokens[i], deposit.tokens[i], deposit.amounts[i], deposit.deposits[i]);

            unchecked {
                ++i;
            }
        }
```
**after**
```solidity
        uint256 i = 0;
        do{
            _clearToken(msg.sender, deposit.hTokens[i], deposit.tokens[i], deposit.amounts[i], deposit.deposits[i]);
            unchecked {
                ++i;
            }
        }while(i < deposit.tokens.length);
```


**Foundry Gas BenchMark**
Before 
|Function Name   | Min | avg   | median   | max   | calls  |
|---|---|---|---|---|---|
| redeemDeposit  | 3641  | 26245  | 28605  | 64228  | 6  |

After

|Function Name   | Min | avg   | median   | max   | calls  |
|---|---|---|---|---|---|
| redeemDeposit  | 3641  | 26171  | 28495  | 64118  | 6  |


code snippet:-
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L447C1-L453C10


Scenario 2 :-


**Before**
```solidity
for (uint256 i = 0; i < outputParams.outputTokens.length;) {
                IVirtualAccount(userAccount).withdrawERC20(outputParams.outputTokens[i], outputParams.amountsOut[i]);

                unchecked {
                    ++i;
                }
            }
```

**After**
```solidity
uint i;
do{  
        
IVirtualAccount(userAccount).withdrawERC20(outputParams.outputTokens[i], outputParams.amountsOut[i]);

++i;

}while(i < outputParams.outputTokens.length);
```
code snippet:-
https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L278C1-L284C14
https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L367C11-L373C14

## 3. Use calldata instead of memory which arguments used for read-only(ex. only if-else , for-loop) :-

Below function [_transferAndApproveMultipleTokens()](https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L186C1-L199C6) arguments of function declared as memory location which is used for for-loop calling another internal function [_transferAndApproveMultipleToken()](https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L160C4-L177C6) inside this function arguments used in if-else statements and transfer of tokens . Here we can declare arguments as calldata location to save some runtime gas. 

```solidity
    function _transferAndApproveMultipleTokens(
        address[] memory _hTokens,//@audit
        address[] memory _tokens,//@audit
        uint256[] memory _amounts,//@audit
        uint256[] memory _deposits//@audit
    ) internal {
        for (uint256 i = 0; i < _hTokens.length;) {
            _transferAndApproveToken(_hTokens[i], _tokens[i], _amounts[i], _deposits[i]);

            unchecked {
                ++i;
            }//gas-opt
        }
    }
```

After

```solidity
    function _transferAndApproveMultipleTokens(
        address[] calldata _hTokens, //@audit changed here
        address[] calldata _tokens,//@audit changed here
        uint256[] calldata _amounts,//@audit changed here
        uint256[] calldata _deposits//@audit changed here
    ) internal {
        for (uint256 i = 0; i < _hTokens.length;) {
            _transferAndApproveToken(_hTokens[i], _tokens[i], _amounts[i], _deposits[i]);

            unchecked {
                ++i;
            }//gas-opt
        }
    }
```

Please look into `//@audit changed here` in above function

Function [_transferAndApproveMultipleTokens()](https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L186C1-L199C6) called by [callOutAndBridgeMultiple()](https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L104) external function only in [BaseBranchRouter.sol](https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol) contract 

**Foundry Gas Benchmark**

Before
|Function Name   | Min | avg   | median   | max   | calls  |
|---|---|---|---|---|---|
|  callOutAndBridgeMultiple | 540103  | 540103  | 540103  | 540103  | 6  |



After
|Function Name   | Min | avg   | median   | max   | calls  |
|---|---|---|---|---|---|
|  callOutAndBridgeMultiple | 462810  | 462810  | 462810  | 462810  | 6  |

