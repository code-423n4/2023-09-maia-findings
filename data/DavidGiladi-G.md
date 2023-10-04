
### Gas Optimization Issues
|Title|Issue|Instances|Total Gas Saved|
|-|:-|:-:|:-:|
|[G-1] Use of selfbalance() over address(this).balance | [Use of selfbalance() over address(this).balance](#use-of-selfbalance-over-addressthisbalance) | 14 | 1400 |
|[G-2] Redundant state variable getters | [Redundant state variable getters](#redundant-state-variable-getters) | 2 | - |
|[G-3] Cache External Calls in Loops | [Cache External Calls in Loops](#cache-external-calls-in-loops) | 1 | 100 |
|[N-4] Unnecessary Casting of Variables | [Unnecessary Casting of Variables](#unnecessary-casting-of-variables) | 21 |
|[G-5] Greater or Equal Comparison Costs Less Gas than Greater Comparison | [Greater or Equal Comparison Costs Less Gas than Greater Comparison](#greater-or-equal-comparison-costs-less-gas-than-greater-comparison) | 1 | 3 |
|[G-6] Optimal State Variable Order | [Optimal State Variable Order](#optimal-state-variable-order) | 1 | 5000 |
|[G-7] Redundant Contract Existence Check in Consecutive External Calls | [Redundant Contract Existence Check in Consecutive External Calls](#redundant-contract-existence-check-in-consecutive-external-calls) | 1 | 100 |

Total: 7 issues

#


## Use of selfbalance() over address(this).balance
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 1400

### Description
Detects when address(this).balance is used instead of selfbalance(), which is cheaper in terms of gas usage.

<details>

<summary>
There are 14 instances of this issue:

</summary>

###
#

```
File: src/BranchBridgeAgent.sol

717    (bool success,) = bridgeAgentExecutorAddress.call{value: address(this).balance}(_calldata)
```

[https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L717](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L717)


#

```
File: src/BranchBridgeAgent.sol

737    (bool success,) = bridgeAgentExecutorAddress.call{value: address(this).balance}(_calldata)
```

[https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L737](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L737)


#

```
File: src/BranchBridgeAgent.sol

787    ILayerZeroEndpoint(lzEndpointAddress).send{value: address(this).balance}(

788                rootChainId,

789                rootBridgeAgentPath,

790                abi.encodePacked(bytes1(0x09), _settlementNonce),

791                _refundee,

792                address(0),

793                ""

794            )
```

[https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L787-L794](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L787-L794)


#

```
File: src/BranchBridgeAgentExecutor.sol

92    _recipient.safeTransferETH(address(this).balance)
```

[https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgentExecutor.sol#L92](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgentExecutor.sol#L92)


#

```
File: src/BranchBridgeAgentExecutor.sol

126    _recipient.safeTransferETH(address(this).balance)
```

[https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgentExecutor.sol#L126](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgentExecutor.sol#L126)


#

```
File: src/BranchBridgeAgent.sol

717    (bool success,) = bridgeAgentExecutorAddress.call{value: address(this).balance}(_calldata)
```

[https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L717](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L717)


#

```
File: src/BranchBridgeAgent.sol

737    (bool success,) = bridgeAgentExecutorAddress.call{value: address(this).balance}(_calldata)
```

[https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L737](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L737)


#

```
File: src/BranchBridgeAgent.sol

787    ILayerZeroEndpoint(lzEndpointAddress).send{value: address(this).balance}(

788                rootChainId,

789                rootBridgeAgentPath,

790                abi.encodePacked(bytes1(0x09), _settlementNonce),

791                _refundee,

792                address(0),

793                ""

794            )
```

[https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L787-L794](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L787-L794)


#

```
File: src/BranchBridgeAgentExecutor.sol

92    _recipient.safeTransferETH(address(this).balance)
```

[https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgentExecutor.sol#L92](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgentExecutor.sol#L92)


#

```
File: src/BranchBridgeAgentExecutor.sol

126    _recipient.safeTransferETH(address(this).balance)
```

[https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgentExecutor.sol#L126](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgentExecutor.sol#L126)


#

```
File: src/RootBridgeAgent.sol

683    _performRetrySettlementCall(

684                    _payload[0] == 0x87,

685                    settlementReference.hTokens,

686                    settlementReference.tokens,

687                    settlementReference.amounts,

688                    settlementReference.deposits,

689                    params,

690                    nonce,

691                    payable(settlementReference.owner),

692                    settlementReference.recipient,

693                    settlementReference.dstChainId,

694                    gParams,

695                    address(this).balance

696                )
```

[https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L683-L696](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L683-L696)


#

```
File: src/RootBridgeAgent.sol

754    (bool success,) = bridgeAgentExecutorAddress.call{value: address(this).balance}(_calldata)
```

[https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L754](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L754)


#

```
File: src/RootBridgeAgent.sol

779    (bool success,) = bridgeAgentExecutorAddress.call{value: address(this).balance}(_calldata)
```

[https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L779](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L779)


#

```
File: src/RootBridgeAgent.sol

940    ILayerZeroEndpoint(lzEndpointAddress).send{value: address(this).balance}(

941                _dstChainId,

942                getBranchBridgeAgentPath[_dstChainId],

943                abi.encodePacked(bytes1(0x04), _depositNonce),

944                payable(_refundee),

945                address(0),

946                ""

947            )
```

[https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L940-L947](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L940-L947)


</details>

# 



## Redundant state variable getters
- Severity: Gas Optimization
- Confidence: High

### Description
Detects and reports state variables that have manually coded getters. Getters for public state variables are automatically generated in Solidity. Coding them manually is redundant and a waste of gas.

<details>

<summary>
There are 2 instances of this issue:

</summary>

###
#

```
File: src/factories/ERC20hTokenBranchFactory.sol

87    function getHTokens() external view returns (ERC20hTokenBranch[] memory) 
```

[https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenBranchFactory.sol#L87-L89](https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenBranchFactory.sol#L87-L89)


#

```
File: src/factories/ERC20hTokenRootFactory.sol

63    function getHTokens() external view returns (ERC20hTokenRoot[] memory) 
```

[https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenRootFactory.sol#L63-L65](https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenRootFactory.sol#L63-L65)


</details>

# 

## Unnecessary Casting of Variables
- Severity: Gas Optimization
- Confidence: High

### Description
This detector scans for instances where a variable is casted to its own type. This is unnecessary and can be safely removed to improve code readability.

<details>

<summary>
There are 21 instances of this issue:

</summary>

###
#

```
File: src/BranchBridgeAgent.sol

770    ILayerZeroEndpoint(lzEndpointAddress).send{value: msg.value}(

771                rootChainId,

772                rootBridgeAgentPath,

773                _payload,

774                payable(_refundee),

775                address(0),

776                abi.encodePacked(uint16(2), _gParams.gasLimit, _gParams.remoteBranchExecutionGas, rootBridgeAgentAddress)

777            )
```
Unnecessary cast: `payable(_refundee)` it cast to the same type.<br>
[https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L770-L777](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L770-L777)


#

```
File: src/BranchBridgeAgent.sol

665    address payable recipient = payable(address(uint160(bytes20(_payload[PARAMS_START:PARAMS_START_SIGNED]))))
```
Unnecessary cast: `payable(address(uint160(bytes20(_payload[PARAMS_START:PARAMS_START_SIGNED]))))` it cast to the same type.<br>
[https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L665](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L665)


#

```
File: src/BranchBridgeAgent.sol

640    address payable recipient = payable(address(uint160(bytes20(_payload[PARAMS_START:PARAMS_START_SIGNED]))))
```
Unnecessary cast: `payable(address(uint160(bytes20(_payload[PARAMS_START:PARAMS_START_SIGNED]))))` it cast to the same type.<br>
[https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L640](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L640)


#

```
File: src/BranchBridgeAgent.sol

618    address payable recipient = payable(address(uint160(bytes20(_payload[PARAMS_START:PARAMS_START_SIGNED]))))
```
Unnecessary cast: `payable(address(uint160(bytes20(_payload[PARAMS_START:PARAMS_START_SIGNED]))))` it cast to the same type.<br>
[https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L618](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L618)


#

```
File: src/BranchBridgeAgent.sol

747    _performFallbackCall(payable(_refundee), _settlementNonce)
```
Unnecessary cast: `payable(_refundee)` it cast to the same type.<br>
[https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L747](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L747)


#

```
File: src/BranchBridgeAgent.sol

770    ILayerZeroEndpoint(lzEndpointAddress).send{value: msg.value}(

771                rootChainId,

772                rootBridgeAgentPath,

773                _payload,

774                payable(_refundee),

775                address(0),

776                abi.encodePacked(uint16(2), _gParams.gasLimit, _gParams.remoteBranchExecutionGas, rootBridgeAgentAddress)

777            )
```
Unnecessary cast: `payable(_refundee)` it cast to the same type.<br>
[https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L770-L777](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L770-L777)


#

```
File: src/BranchBridgeAgent.sol

618    address payable recipient = payable(address(uint160(bytes20(_payload[PARAMS_START:PARAMS_START_SIGNED]))))
```
Unnecessary cast: `payable(address(uint160(bytes20(_payload[PARAMS_START:PARAMS_START_SIGNED]))))` it cast to the same type.<br>
[https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L618](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L618)


#

```
File: src/BranchBridgeAgent.sol

640    address payable recipient = payable(address(uint160(bytes20(_payload[PARAMS_START:PARAMS_START_SIGNED]))))
```
Unnecessary cast: `payable(address(uint160(bytes20(_payload[PARAMS_START:PARAMS_START_SIGNED]))))` it cast to the same type.<br>
[https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L640](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L640)


#

```
File: src/BranchBridgeAgent.sol

747    _performFallbackCall(payable(_refundee), _settlementNonce)
```
Unnecessary cast: `payable(_refundee)` it cast to the same type.<br>
[https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L747](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L747)


 


#

```
File: src/RootBridgeAgent.sol

915    ILayerZeroEndpoint(lzEndpointAddress).send{value: _value}(

916                    _dstChainId,

917                    getBranchBridgeAgentPath[_dstChainId],

918                    payload,

919                    payable(_refundee),

920                    address(0),

921                    abi.encodePacked(uint16(2), _gParams.gasLimit, _gParams.remoteBranchExecutionGas, callee)

922                )
```
Unnecessary cast: `payable(_refundee)` it cast to the same type.<br>
[https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L915-L922](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L915-L922)


#

```
File: src/RootBridgeAgent.sol

257    _performRetrySettlementCall(

258                _hasFallbackToggled,

259                settlementReference.hTokens,

260                settlementReference.tokens,

261                settlementReference.amounts,

262                settlementReference.deposits,

263                _params,

264                _settlementNonce,

265                payable(settlementReference.owner),

266                _recipient,

267                settlementReference.dstChainId,

268                _gParams,

269                msg.value

270            )
```
Unnecessary cast: `payable(settlementReference.owner)` it cast to the same type.<br>
[https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L257-L270](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L257-L270)


#

```
File: src/RootBridgeAgent.sol

295    _performCall(settlementReference.dstChainId, payable(settlementOwner), payload, _gParams)
```
Unnecessary cast: `payable(settlementOwner)` it cast to the same type.<br>
[https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L295](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L295)


#

```
File: src/RootBridgeAgent.sol

683    _performRetrySettlementCall(

684                    _payload[0] == 0x87,

685                    settlementReference.hTokens,

686                    settlementReference.tokens,

687                    settlementReference.amounts,

688                    settlementReference.deposits,

689                    params,

690                    nonce,

691                    payable(settlementReference.owner),

692                    settlementReference.recipient,

693                    settlementReference.dstChainId,

694                    gParams,

695                    address(this).balance

696                )
```
Unnecessary cast: `payable(settlementReference.owner)` it cast to the same type.<br>
[https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L683-L696](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L683-L696)


#

```
File: src/RootBridgeAgent.sol

712    _performFallbackCall(

713                        payable(address(uint160(bytes20(_payload[PARAMS_START:PARAMS_START_SIGNED])))), nonce, _srcChainId

714                    )
```
Unnecessary cast: `payable(address(uint160(bytes20(_payload[PARAMS_START:PARAMS_START_SIGNED]))))` it cast to the same type.<br>
[https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L712-L714](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L712-L714)


#

```
File: src/RootBridgeAgent.sol

788    _performFallbackCall(payable(_refundee), _depositNonce, _srcChainId)
```
Unnecessary cast: `payable(_refundee)` it cast to the same type.<br>
[https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L788](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L788)


#

```
File: src/RootBridgeAgent.sol

940    ILayerZeroEndpoint(lzEndpointAddress).send{value: address(this).balance}(

941                _dstChainId,

942                getBranchBridgeAgentPath[_dstChainId],

943                abi.encodePacked(bytes1(0x04), _depositNonce),

944                payable(_refundee),

945                address(0),

946                ""

947            )
```
Unnecessary cast: `payable(_refundee)` it cast to the same type.<br>
[https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L940-L947](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L940-L947)


#

```
File: src/RootBridgeAgentExecutor.sol

281    i < uint256(uint8(numOfAssets))
```
Unnecessary cast: `uint8(numOfAssets)` it cast to the same type.<br>
[https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L281](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L281)


#

```
File: src/token/ERC20hTokenBranch.sol

20    ERC20(string(string.concat(chainName, _name)), string(string.concat(chainSymbol, _symbol)), _decimals)
```
Unnecessary cast: `string(string.concat(chainName, _name))` it cast to the same type.<br>
[https://github.com/code-423n4/2023-09-maia/blob/main/src/token/ERC20hTokenBranch.sol#L20](https://github.com/code-423n4/2023-09-maia/blob/main/src/token/ERC20hTokenBranch.sol#L20)


#

```
File: src/token/ERC20hTokenBranch.sol

20    ERC20(string(string.concat(chainName, _name)), string(string.concat(chainSymbol, _symbol)), _decimals)
```
Unnecessary cast: `string(string.concat(chainSymbol, _symbol))` it cast to the same type.<br>
[https://github.com/code-423n4/2023-09-maia/blob/main/src/token/ERC20hTokenBranch.sol#L20](https://github.com/code-423n4/2023-09-maia/blob/main/src/token/ERC20hTokenBranch.sol#L20)


#

```
File: src/token/ERC20hTokenRoot.sol

38    ERC20(string(string.concat(_name)), string(string.concat(_symbol)), _decimals)
```
Unnecessary cast: `string(string.concat(_name))` it cast to the same type.<br>
[https://github.com/code-423n4/2023-09-maia/blob/main/src/token/ERC20hTokenRoot.sol#L38](https://github.com/code-423n4/2023-09-maia/blob/main/src/token/ERC20hTokenRoot.sol#L38)


#

```
File: src/token/ERC20hTokenRoot.sol

38    ERC20(string(string.concat(_name)), string(string.concat(_symbol)), _decimals)
```
Unnecessary cast: `string(string.concat(_symbol))` it cast to the same type.<br>
[https://github.com/code-423n4/2023-09-maia/blob/main/src/token/ERC20hTokenRoot.sol#L38](https://github.com/code-423n4/2023-09-maia/blob/main/src/token/ERC20hTokenRoot.sol#L38)


 

#



</details>

# 

## Cache External Calls in Loops
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 100

### Description
This detector identifies instances of repeated external calls within loops. By moving the first external call outside of the loop and using low-level calls inside the loop, it is possible to save gas by avoiding repeated EXTCODESIZE opcodes, as there is no need to check again if the contract exists. 

Note: This detector only triggers if the function call does not return any value.

Prior to 0.8.10, the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent Solidity versions the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low-level calls never check for contract existence. 

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
#

```
File: src/RootBridgeAgent.sol

328    IPort(localPortAddress).bridgeToRoot(

329                        msg.sender,

330                        IPort(localPortAddress).getGlobalTokenFromLocal(_hToken, _dstChainId),

331                        settlement.amounts[i],

332                        settlement.deposits[i],

333                        _dstChainId

334                    )
```

[https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L328-L334](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L328-L334)


</details>

# 



## Greater or Equal Comparison Costs Less Gas than Greater Comparison
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 3

### Description
In Solidity, the >= operator costs less gas than the > operator. This is because the Solidity compiler uses the LT opcode for >= comparisons, which requires less gas than the combination of GT and ISZERO opcodes used for > comparisons. Therefore, replacing > with >= when possible can reduce the gas cost of your contract.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
#

```
File: src/RootBridgeAgent.sol

1146    _deposit > 0
```
 using GREATER_EQUAL/LESS_EQUAL in this case is identical therefore should be use.

[https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1146](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1146)


</details>

# 



## Optimal State Variable Order
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 5000

### Description
Detects optimal variable order in contract storage layouts to decrease the number of storage slots used. Each storage slot used can cost at least 5,000 gas for each write operation, and potentially up to 20,000 gas if you're turning a zero value into a non-zero value. Hence, optimizing storage usage can result in significant gas savings. The real-world savings could vary depending on your contract's specific logic and the state of the Ethereum network.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
#
Optimization opportunity in storage variable layout of contract 
```
File: src/RootBridgeAgent.sol

32    contract RootBridgeAgent is IRootBridgeAgent, BridgeAgentConstants 
```

- original variable order (count: 12 slots)
    - uint16 localChainId
    - address factoryAddress
    - address localRouterAddress
    - address localPortAddress
    - address lzEndpointAddress
    - address bridgeAgentExecutorAddress
    - mapping(uint256 => address) getBranchBridgeAgent
    - mapping(uint256 => bytes) getBranchBridgeAgentPath
    - mapping(uint256 => bool) isBranchBridgeAgentAllowed
    - uint32 settlementNonce
    - mapping(uint256 => Settlement) getSettlement
    - mapping(uint256 => mapping(uint256 => uint256)) executionState
    - uint256 _unlocked


 - optimized variable order (count: 11 slots)
    - address factoryAddress
    - uint32 settlementNonce
    - uint16 localChainId
    - address localRouterAddress
    - address localPortAddress
    - address lzEndpointAddress
    - address bridgeAgentExecutorAddress
    - mapping(uint256 => address) getBranchBridgeAgent
    - mapping(uint256 => bytes) getBranchBridgeAgentPath
    - mapping(uint256 => bool) isBranchBridgeAgentAllowed
    - mapping(uint256 => Settlement) getSettlement
    - mapping(uint256 => mapping(uint256 => uint256)) executionState
    - uint256 _unlocked



Recomended optimizations will use 1 less slots.

[https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L32-L1238](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L32-L1238)


</details>

# 



## Redundant Contract Existence Check in Consecutive External Calls
- Severity: Gas Optimization
- Confidence: Medium
- Total Gas Saved: 100

### Description
This detector identifies instances where there are consecutive external calls to the same contract, where the subsequent calls could use a low-level call to save gas.

Note: This detector only triggers if the function call does not return any value. 

Prior to 0.8.10, the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent Solidity versions the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low-level calls never check for contract existence. 

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
#

```
File: src/RootBridgeAgent.sol

574    IPort(localPortAddress).toggleVirtualAccountApproved(userAccount, localRouterAddress)
```
This call could be replaced with a low-level call because the contract `IRootPort` has already been checked in <br>`Line: 554    IPort(localPortAddress).toggleVirtualAccountApproved(userAccount, localRouterAddress)`<br>
[https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L574](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L574)


</details>

# 
