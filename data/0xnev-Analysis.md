## Analysis Summary
In this analysis, I solely focused on all the possible execution pathways in the Ulysses Omnichain protocol. The general summary flow for interaction involves the following, (with some exceptions):

1. Origin Branch Chain Router 
2. Origin Branch Chain Bridge Agent 
3. Root Chain Bridge Agent
4. Root Chain Bridge Agent Executor 
5. Root Chain Router
6. Destination Branch Chain Bridge Agent
7. Destination Branch Chain Bridge Agent Executor
8. Destination Branch Chain Router

## 1. Bridge Agents `lzReceiveNonBlocking()`

- Called by `lzReceive()` by enpoint

## 1.1 `BranchBridgeAgent.lzReceiveNonBlocking()`

| Contract | Function| Flag | Action | 
|:--:|:--:|:--:|:--:|
| `BranchBridgeAgent.sol` | `lzReceiveNonBlocking()` | `0x00` | `BranchBridgeAgentExecutor.executeNoSettlement()` |
| `BranchBridgeAgent.sol` | `lzReceiveNonBlocking()` | `0x01` | `BranchBridgeAgentExecutor.executeWithSettlement()` |
| `BranchBridgeAgent.sol` | `lzReceiveNonBlocking()` | `0x02` | `BranchBridgeAgentExecutor.executeWithSettlementMultiple()` |
| `BranchBridgeAgent.sol` | `lzReceiveNonBlocking()` | `0x03` | `_performFallback()`, `0x09` flag to root chain RootBridgeAgent |
| `BranchBridgeAgent.sol` | `lzReceiveNonBlocking()` | `0x04` | Fallback, stops cross chain calls to make settlements non-retryable |

## 1.2 `RootBridgeAgent.lzReceiveNonBlocking`
| Contract | Function| Flag | Action | 
|:--:|:--:|:--:|:--:|
| `RootBridgeAgent.sol` | `lzReceiveNonBlocking()` | `0x00` | `RootBridgeAgentExecutor.executeSystemRequest()` |
| `RootBridgeAgent.sol` | `lzReceiveNonBlocking()` | `0x01` | `RootBridgeAgentExecutor.executeNoDeposit()` |
| `RootBridgeAgent.sol` | `lzReceiveNonBlocking()` | `0x02` | `RootBridgeAgentExecutor.executeWithDeposit()` |
| `RootBridgeAgent.sol` | `lzReceiveNonBlocking()` | `0x03` | `RootBridgeAgentExecutor.executeWithDepositMultiple()` |
| `RootBridgeAgent.sol` | `lzReceiveNonBlocking()` | `0x04` | `RootBridgeAgentExecutor.executeSignedNoDeposit()` |
| `RootBridgeAgent.sol` | `lzReceiveNonBlocking()` | `0x05` | `RootBridgeAgentExecutor.executeSignedWithDeposit()` |
| `RootBridgeAgent.sol` | `lzReceiveNonBlocking()` | `0x06` | `RootBridgeAgentExecutor.executeSignedWithDepositMultiple()` |
| `RootBridgeAgent.sol` | `lzReceiveNonBlocking()` | `0x07` | `_performRetrySettlementCall()` |
| `RootBridgeAgent.sol` | `lzReceiveNonBlocking()` | `0x08` | `_performFallback()` `0x04` flag to branch chain BranchBridgeAgent|
| `RootBridgeAgent.sol` | `lzReceiveNonBlocking()` | `0x09` | Fallback, stops cross chain calls to reopen settlements for redemption via `redeemSettlement()` |

## 2. Bridge Agent Executors

## 2.1 `BranchBridgeAgentExecutor.sol`

| Contract | Function| Action | Notes |
|:--:|:--:|:--:|:--:|
| `BranchBridgeAgentExecutor.sol` | `executeNoSettlement()` | `CoreBranchRouter.executeNoSettlement()` | 7 Flags, `0x01 - 0x07`|
| `BranchBridgeAgentExecutor.sol` | `executeWithSettlement()` | `CoreBranchRouter.executeWithSettlement()` | Custom Router Logic |
| `BranchBridgeAgentExecutor.sol` | `executeWithSettlementMultiple()` | `CoreBranchRouter.executeWithSettlementMultiple()` | Custom Router Logic|

## 2.2 `RootBridgeAgentExecutor.sol`
| Contract | Function| Action | Notes |
|:--:|:--:|:--:|:--:|
| `RootBridgeAgentExecutor.sol` | `executeSystemRequest()` | `CoreRootRouter.executeResponse()` | 3 Flags, `0x02 - 0x04`|
| `RootBridgeAgentExecutor.sol` | `executeNoDeposit()` | `CoreRootRouter.execute()` | 1 Flag, `0x01` |
| `RootBridgeAgentExecutor.sol` | `executeWithDeposit()` | `CoreRootRouter.executeDepositSingle()` | Custom Router Logic|
| `RootBridgeAgentExecutor.sol` | `executeWithDepositMultiple()` | `CoreRootRouter.executeWithDepositMultiple()` | Custom Router Logic|
| `RootBridgeAgentExecutor.sol` | `executeSignedNoDeposit()` | `CoreRootRouter.executeSigned()` | Custom Router Logic|
| `RootBridgeAgentExecutor.sol` | `executeSignedWithDeposit()` | `CoreRootRouter.executeSignedDepositSingle()` | Custom Router Logic |
| `RootBridgeAgentExecutor.sol` | `executeSignedWithDepositMultiple()` | `CoreRootRouter.executeSignedDepositMultiple()` | Custom Router Logic |


## 3. Router Executions
## 3.1 `CoreBranchRouter.sol`

### `CoreBranchRouter.executeNoSettlement()`
| Contract | Function| Flag | Action | 
|:--:|:--:|:--:|:--:|
| `CoreBranchRouter.sol` | `executeNoSettlement()` | `0x01` | `_receiveAddGlobalToken()` |
| `CoreBranchRouter.sol` | `executeNoSettlement()` | `0x02` | `_receiveAddBridgeAgent()` |
| `CoreBranchRouter.sol` | `executeNoSettlement()` | `0x03` | `_toggleBranchBridgeAgentFactory()` |
| `CoreBranchRouter.sol` | `executeNoSettlement()` | `0x04` | `_removeBranchBridgeAgent()` |
| `CoreBranchRouter.sol` | `executeNoSettlement()` | `0x05` | `_manageStrategyToken()` |
| `CoreBranchRouter.sol` | `executeNoSettlement()` | `0x06` | `_managePortStrategy()` |
| `CoreBranchRouter.sol` | `executeNoSettlement()` | `0x07` | `RootPort.setCoreBranchRouter()` |

## 3.2 `BaseBranchRouter.sol`

### `BaseBranchRouter.sol`: BranchBridgeAgentExecutor Settlements Execution
| Contract | Function| Flag | Action | 
|:--:|:--:|:--:|:--:|
| `CoreBranchRouter.sol` | `executeSettlement()` | - | Custom Router Logic |
| `CoreBranchRouter.sol` | `executeSettlementMultiple()` | - | Custom Router Logic |

## 3.3 `CoreRootRouter.sol`
### `CoreRootRouter.executeResponse()`
| Contract | Function| Flag | Action | 
|:--:|:--:|:--:|:--:|
| `CoreRootRouter.sol` | `executeResponse()` | `0x02` | `_addLocalToken()` |
| `CoreRootRouter.sol` | `executeResponse()` | `0x03` | `_setLocalToken()` |
| `CoreRootRouter.sol` | `executeResponse()` | `0x04` | `_syncBranchBridgeAgent()` |

### `CoreRootRouter.execute()`
| Contract | Function| Flag | Action | 
|:--:|:--:|:--:|:--:|
| `CoreRootRouter.sol` | `execute()` | `0x01` | `_addGlobalToken()` |

### `CoreRootRouter.sol`: RootBridgeAgentExecutor Deposits Execution
| Contract | Function| Flag | Action | 
|:--:|:--:|:--:|:--:|
| `CoreRootRouter.sol` | `executeDepositSingle()` | - | Custom Router Logic |
| `CoreRootRouter.sol` | `executeDepositMultiple()` | - | Custom Router Logic |
| `CoreRootRouter.sol` | `executeSigned()` | - | Custom Router Logic |
| `CoreRootRouter.sol` | `executeSignedDepositSingle()` | - | Custom Router Logic |
| `CoreRootRouter.sol` | `executeSignedDepositMultiple()` | - | Custom Router Logic |

## 4. User Flow

## 4.1. `CoreBranchRouter.sol`
### 4.1.1 `CoreBranchRouter.addGlobalToken()`
| Steps | `params` | `payload` | Internal Function Calls | External Function Calls | Flag |  Checks |
|:--:|:--:|:--:|:--:|:--:|:--:|:--:|
| 1. `CoreBranchRouter.addGlobalToken()` | ` abi.encode(msg.sender, _globalAddress, _dstChainId, [_gParams[1], _gParams[2]]);`  | `abi.encodePacked(bytes1(0x01), params);` | - | `BranchBridgeAgent.callOut()`| `0x01`| - |
| 2. `BranchBridgeAgent.callOut()` | - | `abi.encodePacked(bytes1(0x01), depositNonce++, _params);` | `_performCall()` | `ILayerZeroEndpoint(lzEndpointAddress).send()`| `0x01`| - |
| 3. `RootBridgeAgent.lzReceive() --> RootBridgeAgent.lzReceiveNonBlocking()` | - | `abi.encodePacked(bytes1(0x01), depositNonce++, _params);` | `_execute()` | `RootBridgeAgentExecutor.executeNoDeposit()`| `0x01`| `if (executionState[_srcChainId][nonce] != STATUS_READY) revert AlreadyExecutedTransaction();` |
| 4. `RootBridgeAgentExecutor.executeNoDeposit()` | `abi.encodePacked(bytes1(0x01), params);` | - | - | `CoreRootRouter.execute()`| `0x01`| - |
| 5. `CoreRootRouter.execute()` | `bytes memory params = abi.encode(_globalAddress, ERC20(_globalAddress).name(), ERC20(_globalAddress).symbol(), ERC20(_globalAddress).decimals(), _refundee, _gParams[1]);`| `bytes memory payload = abi.encodePacked(bytes1(0x01), params);` | `_addGlobalToken()`  | `RootBridgeAgent.callOut()` | `0x01`  |`!RootPort.isGlobalAddress()`, `RootPort.isGlobalToken()` |
| 6. `RootBridgeAgent.callOut()` | - | `abi.encodePacked(bytes1(0x00), _recipient, settlementNonce++, _params);` | `_performCall()` | - | `0x00`| `if (callee == address(0)) revert UnrecognizedBridgeAgent();`, `if (_dstChainId != localChainId)`: perform cross chain messaging |
| 7. `BranchBridgeAgent.lzReceive --> BranchBridgeAgent.lzReceiveNonBlocking()` | - | `abi.encodePacked(bytes1(0x00), _recipient, settlementNonce++, _params);` | `_execute()` | `BranchBridgeAgentExecutor.executeNoSettlement()` | `0x00`| `if (executionState[nonce] != STATUS_READY) revert AlreadyExecutedTransaction();` |
| 8. `BranchBridgeAgentExecutor.executeNoSettlement()` | - | `abi.encodePacked(bytes1(0x00), _recipient, settlementNonce++, _params);` | `_execute()` | `CoreBranchRouter.executeNoSettlement()` | `0x00`| - |
| 9. `CoreBranchRouter.executeNoSettlement()` | `bytes memory payload = abi.encodePacked(bytes1(0x01), params);` | - | `_receiveAddGlobalToken()` | `BranchBridgeAgent.callOutSystem()` | `0x01`| - |
| 10. `BranchBridgeAgent.callOutSystem()` | `params = abi.encode(_globalAddress, newToken);` | `abi.encodePacked(bytes1(0x03), params);` |` _performCall()` | `ILayerZeroEndpoint(lzEndpointAddress).send()` | `0x03`| - |
| 11. `BranchBridgeAgent.callOutSystem()` | - | `abi.encodePacked(bytes1(0x00), depositNonce, params);` |` _performCall()` | `ILayerZeroEndpoint(lzEndpointAddress).send()` | `0x00`| - |
| 12. `RootBridgeAgent.lzReceiveNonBlocking()` | - | `abi.encodePacked(bytes1(0x00), depositNonce, params);` | - | `RootBridgeAgentExcecutor.executeSystemRequest()` | `0x00`| - |
| 13. `RootBridgeAgentExecutor.executeSystemRequest()` | `abi.encodePacked(bytes1(0x03), params);` | - | - | `CoreRootRouter.executeResponse()` | `0x03`| - |
| 14. `CoreRootRouter.executeResponse()` | `abi.encodePacked(bytes1(0x03), params);` | - | `_setLocalToken()` | `RootPort.setLocalAddress()` | `0x03`| `RootPort.isLocalToken()` |

### 4.1.2 `CoreBranchRouter.addLocalToken()`
| Steps | `params` | `payload` | Internal Function Calls | External Function Calls | Flag |  Checks |
|:--:|:--:|:--:|:--:|:--:|:--:|:--:|
| 1. `CoreBranchRouter.addLocalToken()` | `abi.encode(_underlyingAddress, newToken, newToken.name(), newToken.symbol(), decimals);`  | `abi.encodePacked(bytes1(0x02), params);` | - | `BranchBridgeAgent.callOutSystem()`| `0x02`| - |
| 2. `BranchBridgeAgent.callOutSystem()` | - | `abi.encodePacked(bytes1(0x00), depositNonce++, _params);` | `_performCall()` | `ILayerZeroEndpoint(lzEndpointAddress).send()`| `0x00`| - |
| 3. `RootBridgeAgent.lzReceive() --> RootBridgeAgent.lzReceiveNonBlocking()` | - | `abi.encodePacked(bytes1(0x00), depositNonce++, _params);` | `_execute()` | `RootBridgeAgentExecutor.executeSystemRequest()`| `0x00`| `if (executionState[_srcChainId][nonce] != STATUS_READY) revert AlreadyExecutedTransaction();` |
| 4. `RootBridgeAgentExecutor.executeSystemRequest()` | `abi.encodePacked(bytes1(0x02), params);` | - | - | `CoreRootRouter.executeResponse()`| `0x02`| - |
| 5. `CoreRootRouter.executeResponse()` | `address underlyingAddress, address localAddress, string memory name, string memory symbol, uint8 decimals, _srcChainId` | - | `_addLocalToken()`  | `ERChTokenRootFactory.createToken()`, `RootPort.setAddresses()` | -  |`RootPort.isGlobalAddress()`, `RootPort.isLocalToken()`, `RootPort.isUnderlyingToken()` |

### 4.1.3 `CoreBranchRouter.callOut`
- The only way for this call to not revert is when calling `addGlobalToken()` since it will revert in `CoreRootRouter.execute()`

### 4.1.4 `CoreBranchRouter.callOutAndBridge()`
| Steps | `params` | `payload` | Internal Function Calls | External Function Calls | Flag |  Checks |
|:--:|:--:|:--:|:--:|:--:|:--:|:--:|
| 1. `CoreBranchRouter.callOutAndBridge()` | - | - | `_transferAndApproveToken()` | `BranchBridgeAgent.callOutAndBridge()`| - | `_amount - deposit > 0`, `_deposit > 0` |
| 2. `BranchBridgeAgent.callOutAndBridge()` | - | `abi.encodePacked(bytes1(0x02), _depositNonce, _dParams.hToken, _dParams.token, _dParams.amount, _dParams.deposit, _params);` | `_createDeposit()`, `_performCall()` | `BranchPort.bridgeOut()`, `ILayerZeroEndpoint(lzEndpointAddress).send()`| `0x02`| - |
| 3. `RootBridgeAgent.lzReceive() --> RootBridgeAgent.lzReceiveNonBlocking()` | - |  `abi.encodePacked(bytes1(0x02), _depositNonce, _dParams.hToken, _dParams.token, _dParams.amount, _dParams.deposit, _params);` | `_execute()` | `RootBridgeAgentExecutor.executeWithDeposit()`| `0x02`| `if (executionState[_srcChainId][nonce] != STATUS_READY) revert AlreadyExecutedTransaction();` |
| 4. `RootBridgeAgentExecutor.executeWithDeposit()` | `abi.encodePacked(bytes1(0x02), params);` | - | `_bridgeIn` | `RootBridgeAgent.bridgeIn()`, `CoreRootRouter.executeDepositSingle()`| -| - |
| 5. `RootBridgeAgent.bridgeIn()` | - | - | `_bridgeIn` | `RootPort.bridgeToRoot()`| -| `!RootPort.isLocalToken()`, `RootPort.getLocalTokenFromUnderlying()` |
| 6. `RootPort.bridgeToRoot()` | - | - | - | `hToken.safeTransfer()`, `ERC20hTokenRoot.mint()`| - | `if (!isGlobalAddress[_hToken]) revert UnrecognizedToken();` |
| 7. `CoreRootRouter.executeDepositSingle()` | - | `abi.encodePacked(bytes1(0x02), params);` | Custom Router Logic | Custom Router Logic | Custom Router Logic  | Custom Router Logic |
| 8. `RootBridgeAgent.callOutAndBridge()` | - | `abi.encodePacked( _hasFallbackToggled ? bytes1(0x81) : bytes1(0x01), _recipient, _settlementNonce, localAddress, underlyingAddress, _amount, _deposit, _params);` | `_createSettlement()`, `_updateStateOnBridgeOut()`, `_performCall()` | `globalAddress.safeTransferFrom()`, `RootPort.burn()`, `ILayerZeroEndpoint(lzEndpointAddress).send()` | `0x01`  | - |
|9. `BranchBridgeAgent.lzReceive() --> BranchBridgeAgent.lzReceiveNonBlocking()`| - | `abi.encodePacked( _hasFallbackToggled ? bytes1(0x81) : bytes1(0x01), _recipient, _settlementNonce, localAddress, underlyingAddress, _amount, _deposit, _params);` | `_execute()` | `BranchBridgeAgentExecutor.executeWithSettlement()` | `0x01` | `if (executionState[nonce] != STATUS_READY) revert AlreadyExecutedTransaction();` |
|10. `BranchBridgeAgentExecutor.executeWithSettlement()`| `sParams`|  `abi.encodePacked( _hasFallbackToggled ? bytes1(0x81) : bytes1(0x01), _recipient, _settlementNonce, localAddress, underlyingAddress, _amount, _deposit, _params);`| - | `BranchBridgeAgent.clearToken()`, `CoreBranchRouter.executeSettlement()`, `_recipient.safeTransferETH`, `BranchPort.bridgeIn()`, `BranchPort.withdraw()`, `ERC20hTokenBranch.mint()`, `underlyingAddress.safeTransfer()`| `0x01`| `if (_amount - _deposit > 0)`, `if (_deposit > 0)`|
|11. `CoreBranchRouter.executeSettlement()`| `payload[129:]`| - | Custom Router Logic | Custom Router Logic | Custom Router Logic | Custom Router Logic| 


### 4.1.5 `CoreBranchRouter.callOutAndBridgeMultiple()`
| Steps | `params` | `payload` | Internal Function Calls | External Function Calls | Flag |  Checks |
|:--:|:--:|:--:|:--:|:--:|:--:|:--:|
| 1. `CoreBranchRouter.callOutAndBridgeMultiple()` | - | - | `_transferAndApproveMultipleTokens()` | `BranchBridgeAgent.callOutAndBridge()`| - | `_amount - deposit > 0`, `_deposit > 0` |
| 2. `BranchBridgeAgent.callOutAndBridgeMultiple()` | - | `abi.encodePacked(bytes1(0x03), _depositNonce, _dParams.hTokens, _dParams.tokens, _dParams.amounts, _dParams.deposits, _params);` | `_createDepositMultiple()`, `_bridgeOut()`, `_performCall()` | `RootPort.bridgeOutMultiple()`, `ILayerZeroEndpoint(lzEndpointAddress).send()`| `0x03`| `if (length > 255) revert InvalidInputArrays();`, `if (length != _underlyingAddresses.length) revert InvalidInputArrays();`,  `if (_underlyingAddresses.length != _amounts.length) revert InvalidInputArrays();`, `if (_amounts.length != _deposits.length) revert InvalidInputArrays();`|
| 3. `RootBridgeAgent.lzReceive() --> RootBridgeAgent.lzReceiveNonBlocking()` | - |  `abi.encodePacked(bytes1(0x03), _depositNonce, _dParams.hTokens, _dParams.tokens, _dParams.amounts, _dParams.deposits, _params);` | `_execute()` | `RootBridgeAgentExecutor.executeWithDepositMultiple()`| `0x03`| `if (executionState[_srcChainId][nonce] != STATUS_READY) revert AlreadyExecutedTransaction();` |
| 4. `RootBridgeAgentExecutor.executeWithDepositMultiple()` | `dParams`, `_payload[6 + hToken.length * 128 : ]` | - | `_bridgeInMultiple()` | `RootBridgeAgent.bridgeInMultiple()`, `CoreRootRouter.executeDepositMultiple()`| - | - |
| 5. `RootBridgeAgent.bridgeInMultiple()` | - | - | `bridgeIn()` | `RootPort.bridgeToRoot()`| - | `if (length > MAX_TOKENS_LENGTH) revert InvalidInputParams();`, `!RootPort.isLocalToken()`, `RootPort.getLocalTokenFromUnderlying()` |
| 6. `RootPort.bridgeToRoot()` | - | - | - | `hToken.safeTransfer()`, `ERC20hTokenRoot.mint()`| - | `if (!isGlobalAddress[_hToken]) revert UnrecognizedToken();` |
| 7. `CoreRootRouter.executeDepositSingle()` | `_payload[6 + hToken.length * 128 : ]` | - | Custom Router Logic | Custom Router Logic | Custom Router Logic  | Custom Router Logic |
| 8. `RootBridgeAgent.callOutAndBridgeMultiple()` | - | `abi.encodePacked( _hasFallbackToggled ? bytes1(0x02) & 0x0F : bytes1(0x02), _recipient, _settlementNonce, localAddress, underlyingAddress, _amount, _deposit, _params);` | `_createSettlement()`, `_updateStateOnBridgeOut()`, `_performCall()` | `globalAddress.safeTransferFrom()`, `RootPort.burn()`, `ILayerZeroEndpoint(lzEndpointAddress).send()` | `0x02`  | `if (_globalAddresses.length > MAX_TOKENS_LENGTH) revert InvalidInputParamsLength();`, `if (_globalAddresses.length != _amounts.length) revert InvalidInputParamsLength();`, `if (_amounts.length != _deposits.length) revert InvalidInputParamsLength();` |
|9. `BranchBridgeAgent.lzReceive() --> BranchBridgeAgent.lzReceiveNonBlocking()`| - | `abi.encodePacked( _hasFallbackToggled ? bytes1(0x02) & 0x0F : bytes1(0x01), _recipient, _settlementNonce, localAddress, underlyingAddress, _amount, _deposit, _params);` | `_execute()` | `BranchBridgeAgentExecutor.executeWithSettlementMultiple()` | `0x02` | `if (executionState[nonce] != STATUS_READY) revert AlreadyExecutedTransaction();` |
|10. `BranchBridgeAgentExecutor.executeWithSettlementMultiple()`| `sParams`|  `abi.encodePacked( _hasFallbackToggled ? bytes1(0x02) & 0x0F : bytes1(0x02), _recipient, _settlementNonce, localAddress, underlyingAddress, _amount, _deposit, _params);`| `BranchPort._bridgeIn()` | `BranchBridgeAgent.clearToken()`, `CoreBranchRouter.executeSettlement()`, `_recipient.safeTransferETH`, `BranchPort.bridgeInMultiple()`, `BranchPort.withdraw()`, `ERC20hTokenBranch.mint()`, `underlyingAddress.safeTransfer()`| `0x02`| `if (_amount[i] - _deposit[i] > 0)`, `if (_deposit[i] > 0)`|
|11. `CoreBranchRouter.executeSettlementMultiple()`| `payload[26 + assestsOffset * 128:]`| - | Custom Router Logic | Custom Router Logic | Custom Router Logic | Custom Router Logic| 

## 4.2. `BranchBridgeAgent.sol`
### 4.2.1 `BranchBridgeAgent.callOutSigned()`
- Same Route as `BranchBridgeAgent.callOut()`, just with a `0x04` deposit flag to communciate with `RootBridgeAgent` and initiated from a virtual account

### 4.2.2 `BranchBridgeAgent.callOutSignedAndBridge()`
- Same route as `BranchBridgeAgent.callOutAndBridge()`, just with a `0x05` deposit flag to communciate with `RootBridgeAgent` and initiated from a virtual account

### 4.2.3 `BranchBridgeAgent.callOutSignedAndBridgeMultiple()`
- Same route as `BranchBridgeAgent.callOutAndBridgeMultiple()`, just with a `0x06` deposit flag to communciate with `RootBridgeAgent` and initiated from a virtual account


### 4.2.4 `BranchBridgeAgent.retryDeposit()`
- This is retrying a deposit towards the root chain in the event the cross-chain messanging fails and fallback is not toggled yet. The following is the flags to retry:

Non-Signed Requests:
| Flag | `payload` |  Flow |
|:--:|:--:|:--:|
| `0x02`| `payload = abi.encodePacked(bytes1(0x02), _depositNonce, deposit.hTokens[0],deposit.tokens[0], deposit.amounts[0], deposit.deposits[0],_params);` | [1.4](#14-corebranchroutercalloutandbridge) | 
| `0x03`| `payload = abi.encodePacked(bytes1(0x03), uint8(deposit.hTokens.length), _depositNonce, deposit.hTokens[0],deposit.tokens[0], deposit.amounts[0], deposit.deposits[0],_params);` | [1.5](#15-corebranchroutercalloutandbridgemultiple) | 

Signed Requests:
| Flag | `payload` |  Flow |
|:--:|:--:|:--:|
| `0x05`| `payload = abi.encodePacked(_hasFallbackToggled ? bytes1(0x85) : bytes1(0x05), msg.sender, _depositNonce, deposit.hTokens[0],deposit.tokens[0], deposit.amounts[0], deposit.deposits[0],_params);` | [2.2](#22-branchbridgeagentcalloutsignedandbridge)| 
| `0x06`| `payload = abi.encodePacked(_hasFallbackToggled ? bytes1(0x86) : bytes1(0x06), msg.sender, uint8(deposit.hTokens.length), _depositNonce, deposit.hTokens,deposit.tokens, deposit.amounts, deposit.deposits,_params);`| [2.3](#23-branchbridgeagentcalloutsignedandbridgemultiple) | 


### 4.2.5 `BranchBridgeAgent.retrieveDeposit()`
- Request tokens back to branch chain from branch port after failing omnichain environment interaction

| Steps | `params` | `payload` | Internal Function Calls | External Function Calls | Flag |  Checks |
|:--:|:--:|:--:|:--:|:--:|:--:|:--:|
| 1. `BranchBridgeAgent.retrieveDeposit()` | - | `abi.encodePacked(bytes1(0x08), msg.sender, _depositNonce);` | `_performCall()` | `ILayerZeroEndpoint(lzEndpointAddress).send()`| `0x08` | `if (getDeposit[_depositNonce].owner != msg.sender) revert NotDepositOwner();` |
| 2. `RootBridgeAgent.lzReceive() --> RootBridgeAgent.lzReceiveNonBlocking()` | - | `abi.encodePacked(bytes1(0x08), msg.sender, _depositNonce);`| `_performFallbackCall()` | `ILayerZeroEndpoint(lzEndpointAddress).send()`| `0x08`| `if (executionState[_srcChainId][nonce] == STATUS_DONE)`, `if (executionState[_srcChainId][nonce] == STATUS_READY)` |
| 3. `ILayerZeroEndpoint(lzEndpointAddress).send()` | - | `abi.encodePacked(bytes1(0x04), _depositNonce),` | - | - | `0x04`| - |
| 4. `BranchBridgeAgent.lzReceiveNonBlocking()` | - | `abi.encodePacked(bytes1(0x04), _depositNonce),` | State change of `getDeposit[nonce] = STATUS_FAILED` from `STATUS_SUCCESS` | - | `0x04`| - |
| 5. `BranchBridgeAgent.redeemDeposit()` | `_depositNonce` | `_clearToken()` |  `delete getDeposit[nonce]` | - | - | - | - |



### 4.2.6 `BranchBridgeAgent.redeemDeposit()`
- Users can redeem failed deposits from branch chain to root chain by manually calling `retrieveDeposit()` or have fallback toggled where incase a deposit fails, it will automatically make a deposit non retryable and available for retrieval and subsequent redemption.

Use Cases:
1. If fallback is toggled off, user will have to manually call `BranchBridgeAgent.retrieveDeposit` from the origin branch chain to set deposit status to retrievable, and then call `BranchBridgeAgent.redeemDeposit()` to get back deposits

<br/>

2. If `hasFallbackToggled` is true, then failed deposits will automatically be made non-retryable and only available for retrieval and subsequent redemption.

### 4.2.7 `BranchBridgeAgent.retrySettlement()`
- Retry failed settlement (deposit from root chain to destination branch chain) from origin branch chain

| Steps | `params` | `payload` | Internal Function Calls | External Function Calls | Flag |  Checks |
|:--:|:--:|:--:|:--:|:--:|:--:|:--:|
| 1. `BranchBridgeAgent.retrySettlement()` | `bytes memory params = abi.encode(_settlementNonce, msg.sender, _params, _gParams[1]);` | `abi.encodePacked(_hasFallbackToggled ? bytes1(0x87) : bytes1(0x07), params);` | `_performCall()` | `ILayerZeroEndpoint(lzEndpointAddress).send()`| `0x07` | - |
| 2. `RootBridgeAgent.lzReceive() --> RootBridgeAgent.lzReceiveNonBlocking()` | - | `abi.encodePacked(_hasFallbackToggled ? bytes1(0x87) : bytes1(0x07), params);`| `_performRetrySettlementCall()` | `ILayerZeroEndpoint(lzEndpointAddress).send()`/ `IBranchBridgeAgent(callee).lzReceive()` | `0x07`| `if (owner != settlementReference.owner)`, `if (owner != address(IPort(localPortAddress).getUserAccount(settlementReference.owner)))` |
| 3. `ILayerZeroEndpoint(lzEndpointAddress).send()` | - | `payload = abi.encodePacked(_hasFallbackToggled ? bytes1(0x81) : bytes1(0x01), _recipient, _settlementNonce, _hTokens[0], _tokens[0], _amounts[0], _deposits[0], _params);` OR `payload = abi.encodePacked(_hasFallbackToggled ? bytes1(0x82) : bytes1(0x02), _recipient, uint8(_hTokens.length), _settlementNonce, _hTokens, _tokens, _amounts, _deposits, _params);`| - | - | `0x01/0x02`| `if (callee == address(0))` |


From step 4 onwards, depending on the settlement involving a single asset or multiple asset, the flow is the same from step 9 onwards of [1.4](#14-corebranchroutercalloutandbridge) and [1.5](#15-corebranchroutercalloutandbridgemultiple) respectively

## 4.3. `CoreRootRouter.sol`
- This are all admin functions that utilizes the `0x00` settlement flag to execute sensitive admin actions with no funds involved
- Deployer of CoreRootRouter could always grief users due to this sensitive actions.
- All calls originate from `CoreRootRouter.sol` which will exist in Arbitrum, all step 2 to 4 is the same for all function calls, the only difference are the checks and parameters.

### General Steps

1. Function call in `CoreRootRouter.sol`
2. `RootBridgeAgent.callOut()`
3. `BranchBridgeAgent.lzReceiveNonBlocking()`
4. `BranchBridgeAgent.executeNoSettlement()`
5. `CoreBranchRouter.executeNoSettlement()`

| Entry (Step 1) | Function Flag | Settlement Flag | `params` for Action | `payload` | Action (Step 5) | 
|:--:|:--:|:--:|:--:|:--:|:--:|
| `addBranchToBridgeAgent()` | `0x02` | `0x00` |` abi.encode(_newBranchRouter, _branchBridgeAgentFactory, _rootBridgeAgent, IBridgeAgent(_rootBridgeAgent).factoryAddress(), _refundee, _gParams[1]);` | `abi.encodePacked(bytes1(0x02), params);` |  `_receiveAddGlobalToken()` |
| `toggleBranchBridgeAgentFactory()` | `0x03` | `0x00` | `abi.encode(_branchBridgeAgentFactory);` |  `abi.encodePacked(bytes1(0x03), params);`| `_toggleBranchBridgeAgentFactory()`  |
| `addBranchToBridgeAgent()` | `0x04` | `0x00` | `abi.encode(_branchBridgeAgent);` | `abi.encodePacked(bytes1(0x04), params);` | `_receiveAddGlobalToken()` |
| `manageStrategyToken()` | `0x05` | `0x00` | `abi.encode(_underlyingToken, _minimumReservesRatio);` | `abi.encodePacked(bytes1(0x05), params);` | `_manageStrategyToken()` |
| `managePortStrategy()` | `0x06` | `0x00` | `abi.encode(_portStrategy, _underlyingToken, _dailyManagementLimit, _isUpdateDailyLimit);` | `abi.encodePacked(bytes1(0x06), params);` | `_managePortStrategy()` |
| `setCoreBranch()` | `0x07` | `0x00` | `abi.encode(_coreBranchRouter, _coreBranchBridgeAgent);` | `abi.encodePacked(bytes1(0x07), params);` | `BranchPort.setCoreBranchRouter()` |


## 4.4. `RootBridgeAgent.sol`
### 4.4.1 `RootBridgeAgent.retrySettlement()`
Same flow as [2.7](#27-branchbridgeagentretrysettlement), just that the call is initiated from the root chain root bridge agent instead of the origin branch chain branch bridge agent.

### 4.4.2 `RootBridgeAgent.retrieveSettlement()`
- Request settlement tokens back to root chain from root port after failing omnichain environment interaction
- In step 3, if destination chain is root chain itself, the local branch bridge agent (root chain) is called

| Steps | `params` | `payload` | Internal Function Calls | External Function Calls | Flag |  Checks |
|:--:|:--:|:--:|:--:|:--:|:--:|:--:|
| 1. `RootBridgeAgent.retrieveSettlement()` | - | `abi.encodePacked(bytes1(0x03), settlementOwner, _settlementNonce);` | `_performCall()` | `ILayerZeroEndpoint(lzEndpointAddress).send()`| `0x03` | `if (settlementOwner == address(0)) revert SettlementRetrieveUnavailable();`, `if (msg.sender != settlementOwner)`, `if (msg.sender != address(IPort(localPortAddress).getUserAccount(settlementOwner)))` |
| 2. `BranchBridgeAgent.lzReceive() --> BranchBridgeAgent.lzReceiveNonBlocking()` | - | `abi.encodePacked(bytes1(0x03), settlementOwner, _settlementNonce);` | `_performFallbackCall()` | `ILayerZeroEndpoint(lzEndpointAddress).send()`| `0x08`| `if (executionState[nonce] == STATUS_DONE)`, `if (executionState[nonce] == STATUS_READY)` | 
| 3. `ILayerZeroEndpoint(lzEndpointAddress).send()` | - | `abi.encodePacked(bytes1(0x09), _settlementNonce)` | - | - | `0x09`| - | - |
| 4. `RootBridgeAgent.lzReceiveNonBlocking()` | - | `abi.encodePacked(bytes1(0x04), _depositNonce),` | State change of `getSettlement[nonce] = STATUS_FAILED` from `STATUS_SUCCESS` | - | `0x09`| - | 
| 5. `RootBridgeAgent.redeemSettlement()` | `_settlementNonce` | `_clearToken()` | `RootPort.bridgeToRoot()` | - | - |  `if (settlement.status == STATUS_SUCCESS) revert SettlementRedeemUnavailable();`, `if (msg.sender != settlementOwner)`, `if (msg.sender != address(IPort(localPortAddress).getUserAccount(settlementOwner)))` |

### 4.4.3 `RootBridgeAgent.redeemSettlement()`
Similar to `BranchBridgeAgent.redeemDeposit()`, except users can redeem failed settlements to branch chain from root chain by manually calling `retrieveSettlement()` or have fallback toggled where incase a deposit fails, it will automatically make a deposit non retryable and available for retrieval and subsequent redemption.

Use Cases:
1. If fallback is toggled off, user will have to manually call `BranchBridgeAgent.retrieveSettlement` from the root chain to set settlement status to retrievable, and then call `RootBridgeAgent.redeemSettlement()` to get back settlements

<br/>

2. If `hasFallbackToggled` is true, then failed settlements will automatically be made non-retryable and only available for subsequent redemption via `RootBridgeAgent.redeemSettlement()`

## 4.5. `MultiCallRootRouter.sol`
This is essentially a root router where user wants to interact directly with third party dApps on root omnichain (arbitrum) environtment from branch chains and/or root chain itself.

Users can access this functions via when they call a BranchBridgeAgent that is connected to a RootBridgeAgent that has a `MultiCallRootRouter.sol` as it's router

Action depends on the logic of the third-party dApp to interact with.

The difference between this and `CoreRootRouter.sol` is the access to virtual accounts to directly interact with third-party dApps and the Router excecutions below:

## 4.5.1 `MultiCallRootRouter` execution function flags
| Contract | Flag | Action | 
|:--:|:--:|:--:|
| `MultiCallRootRouter.sol` | `0x01` | `_multicallNoOutput()` |
| `MultiCallRootRouter.sol` | `0x02` | `_multicallSingleOutput()` |
| `MultiCallRootRouter.sol` | `0x03` | `_multicallMultipleOutput()` |






### Time spent:
50 hours