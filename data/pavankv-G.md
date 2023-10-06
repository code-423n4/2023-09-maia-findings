## 1. Refactor for-loop to save some runtime gas :-

In below code snippet we can see for each loop iteration numOfAssets which already declared in line [273](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L273) as uint8 , but in loop again try to type cast to uint8 and again to uint256 bits . So for-loop requires only constant value. So instead of conversion in every loop iteration we can declare variable outside loop and try to call. Which save some type cast operation.

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

## 4. TypeCast can be done in if-else instead of memory variable :-
In below function calldata argument can type cast directly in if-else to save some gas .


**Before**
```solidity
  function execute(bytes calldata _encodedData, uint16) external payable override requiresExecutor {
        // Parse funcId
        bytes1 funcId = _encodedData[0]; //@audit

        /// FUNC ID: 1 (_addGlobalToken)
        if (funcId == 0x01) {//@audit
            (address refundee, address globalAddress, uint16 dstChainId, GasParams[2] memory gasParams) =
                abi.decode(_encodedData[1:], (address, address, uint16, GasParams[2]));

            _addGlobalToken(refundee, globalAddress, dstChainId, gasParams);

            /// Unrecognized Function Selector
        } else {
            revert UnrecognizedFunctionId();
        }
    }
```
Please look into `//@audit` comment in above function

**After**
```solidity
function execute(bytes calldata _encodedData, uint16) external payable override requiresExecutor {
               
        /// FUNC ID: 1 (_addGlobalToken)
        if (bytes1(_encodedData[0]) == 0x01) { //@audit changed here
            (address refundee, address globalAddress, uint16 dstChainId, GasParams[2] memory gasParams) =
                abi.decode(_encodedData[1:], (address, address, uint16, GasParams[2]));

            _addGlobalToken(refundee, globalAddress, dstChainId, gasParams);

            /// Unrecognized Function Selector
        } else {
            revert UnrecognizedFunctionId();
        }
    }
```
Please look into `//@audit changed here` comment in above function.

**Foundry Gas BenchMark**
Before
|Function Name   | Min | avg   | median   | max   | calls  |
|---|---|---|---|---|---|
|  execute | 1025  | 1025  | 1025  | 1025  | 5  |


After
|Function Name   | Min | avg   | median   | max   | calls  |
|---|---|---|---|---|---|
|  execute | 1014  | 1014  | 1014 | 1014  | 5  |

This is exact gas benchmark.

code snippet :-
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L297C4-L307C30


Scenario 2 :-
In below function we can type cast to bytes1() in if-else only to save some runtime gas . 

**Before**
```solidity
 function executeResponse(bytes calldata _encodedData, uint16 _srcChainId)
        external
        payable
        override
        requiresExecutor
    {
        // Parse funcId
        bytes1 funcId = _encodedData[0];//@audit

        ///  FUNC ID: 2 (_addLocalToken)
        if (funcId == 0x02) {//@audit
            (address underlyingAddress, address localAddress, string memory name, string memory symbol, uint8 decimals)
            = abi.decode(_encodedData[1:], (address, address, string, string, uint8));

            _addLocalToken(underlyingAddress, localAddress, name, symbol, decimals, _srcChainId);

            /// FUNC ID: 3 (_setLocalToken)
        } else if (funcId == 0x03) {//@audit
            (address globalAddress, address localAddress) = abi.decode(_encodedData[1:], (address, address));

            _setLocalToken(globalAddress, localAddress, _srcChainId);

            /// FUNC ID: 4 (_syncBranchBridgeAgent)
        } else if (funcId == 0x04) {//@audit
            (address newBranchBridgeAgent, address rootBridgeAgent) = abi.decode(_encodedData[1:], (address, address));

            _syncBranchBridgeAgent(newBranchBridgeAgent, rootBridgeAgent, _srcChainId);

            /// Unrecognized Function Selector
        } else {
            revert UnrecognizedFunctionId();
        }
    }
```
Please look into `//@audit` comment in above function

**After**
```solidity
    function executeResponse(bytes calldata _encodedData, uint16 _srcChainId)
        external
        payable
        override
        requiresExecutor
    {
        // Parse funcId
        ///  FUNC ID: 2 (_addLocalToken)
        if (bytes1(_encodedData[0]) == 0x02) {//@audit changed here
            (address underlyingAddress, address localAddress, string memory name, string memory symbol, uint8 decimals)
            = abi.decode(_encodedData[1:], (address, address, string, string, uint8));

            _addLocalToken(underlyingAddress, localAddress, name, symbol, decimals, _srcChainId);

            /// FUNC ID: 3 (_setLocalToken)
        } else if (bytes1(_encodedData[0]) == 0x03) {//@audit changed here
            (address globalAddress, address localAddress) = abi.decode(_encodedData[1:], (address, address));

            _setLocalToken(globalAddress, localAddress, _srcChainId);

            /// FUNC ID: 4 (_syncBranchBridgeAgent)
        } else if (bytes1(_encodedData[0]) == 0x04) {//@audit changed here
            (address newBranchBridgeAgent, address rootBridgeAgent) = abi.decode(_encodedData[1:], (address, address));

            _syncBranchBridgeAgent(newBranchBridgeAgent, rootBridgeAgent, _srcChainId);

            /// Unrecognized Function Selector
        } else {
            revert UnrecognizedFunctionId();
        }
    }
```
Please look into the `//@audit changed here` comment in above function .

**Foundry Gas BenchMark**
Before
|Function Name   | Min | avg   | median   | max   | calls  |
|---|---|---|---|---|---|
|  executeResponse | 5909  | 887221  | 1102009 | 1102009  | 10  |


After
|Function Name   | Min | avg   | median   | max   | calls  |
|---|---|---|---|---|---|
|  executeResponse | 5898  | 887202  | 1101984 | 1101984  | 10  |

code snippet:-
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L297C3-L329C6

Scenario 3 :-
In below we can directly type cast the `encodedData` argument into bytes1() in if-else check statement in order to save some run time gas like scenario 2 situtions.

code snippet:-
https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L137C5-L200C6
