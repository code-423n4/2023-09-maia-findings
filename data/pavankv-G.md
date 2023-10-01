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



## 10. Use do-while loop for simple operation :-

It will save 5 gas per iteration . 

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
