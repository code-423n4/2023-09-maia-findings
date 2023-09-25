## Summary
## Low Issues


| |Issue|Instances|
|-|:-|:-:|
| [L-1](#L-1-1693313903) | Consider disabling `renounceOwnership()` | 12 |
| [L-2](#L-2-1693313958) | Double type casts create complexity within the code | 5 |
| [L-3](#L-3-1694956284) | Some function should not be marked as payable | 43 |
| [L-4](#L-4-1693314112) | increase/decrease or forceApprove allowance should be used instead of approve | 2 |
| [L-5](#L-5-1693314276) | `onlyOwner` functions not accessible if `owner` renounces ownership | 24 |
| [L-6](#L-6-1693314386) | Consider using OpenZeppelin’s SafeCast library to prevent unexpected overflows when casting from various type int/uint values | 2 |
| [L-7](#L-7-1693315091) | Gas grief possible on unsafe external calls | 18 |

Total: 88 instances over 6 issues


## Non Critical Issues


| |Issue|Instances|
|-|:-|:-:|
| [NC-1](#NC-1-1693315336) | Consider using modifiers for address control | 3 |
| [NC-2](#NC-2-1693315622) | Variable names don't follow the Solidity style guide | 39 |
| [NC-3](#NC-3-1693315664) | Contracts containing only utility functions should be made into libraries | 1 |
| [NC-4](#NC-4-1693315680) | Control structures do not follow the Solidity Style Guide | 78 |
| [NC-5](#NC-5-1693315718) | Consider using `delete` rather than assigning `false` to clear values | 3 |
| [NC-6](#NC-6-1693315838) | Empty bytes check is missing | 87 |
| [NC-7](#NC-7-1694440518) | High cyclomatic complexity | 6 |
| [NC-8](#NC-8-1693316104) | Some if-statement can be converted to a ternary | 4 |
| [NC-9](#NC-9-1693316148) | Contract implements interface without extending the interface | 11 |
| [NC-10](#NC-10-1693316207) | Inconsistent usage of `require`/`error` | 54 |
| [NC-11](#NC-11-1693316644) | Some error strings are not descriptive | 1 |
| [NC-12](#NC-12-1693317147) | State variables should include comments | 22 |
| [NC-13](#NC-13-1693317214) | Top level pragma declarations should be separated by two blank lines | 37 |
| [NC-14](#NC-14-1693317342) | Uncommented fields in a struct | 7 |
| [NC-15](#NC-15-1693317551) | Missing upgradability functionality | 1 |
| [NC-16](#NC-16-1693317704) | Use a single file for system wide constants | 2 |

Total: 356 instances over 16 issues



## Low Issues


| |Issue|Instances|
|-|:-|:-:|
| [L-1](#L-1-1693313903) | Consider disabling `renounceOwnership()` | 12 |
| [L-2](#L-2-1693313958) | Double type casts create complexity within the code | 5 |
| [L-3](#L-3-1694956284) | Some function should not be marked as payable | 43 |
| [L-4](#L-4-1693314112) | increase/decrease or forceApprove allowance should be used instead of approve | 2 |
| [L-5](#L-5-1693314276) | `onlyOwner` functions not accessible if `owner` renounces ownership | 24 |
| [L-6](#L-6-1693314386) | Consider using OpenZeppelin’s SafeCast library to prevent unexpected overflows when casting from various type int/uint values | 2 |
| [L-7](#L-7-1693315091) | Gas grief possible on unsafe external calls | 18 |
### <a name="L-1-1693313903"></a>[L-1] Consider disabling `renounceOwnership()`
If the plan for your project does not include eventually giving up all ownership control, consider overwriting OpenZeppelin's `Ownable`'s `renounceOwnership()` function in order to disable it.

*Instances (12)*:
<details><summary> see instances </summary>

```solidity
File: src/BaseBranchRouter.sol

25: contract BaseBranchRouter is IBranchRouter, Ownable {

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L25)

```solidity
File: src/BranchBridgeAgentExecutor.sol

29: contract BranchBridgeAgentExecutor is Ownable, BridgeAgentConstants {

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgentExecutor.sol#L29)

```solidity
File: src/BranchPort.sol

17: contract BranchPort is Ownable, IBranchPort {

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L17)

```solidity
File: src/CoreRootRouter.sol

38: contract CoreRootRouter is IRootRouter, Ownable {

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L38)

```solidity
File: src/MulticallRootRouter.sol

57: contract MulticallRootRouter is IRootRouter, Ownable {

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L57)

```solidity
File: src/RootBridgeAgentExecutor.sol

27: contract RootBridgeAgentExecutor is Ownable, BridgeAgentConstants {

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L27)

```solidity
File: src/RootPort.sol

16: contract RootPort is Ownable, IRootPort {

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L16)

```solidity
File: src/factories/BranchBridgeAgentFactory.sol

19: contract BranchBridgeAgentFactory is Ownable, IBranchBridgeAgentFactory {

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/BranchBridgeAgentFactory.sol#L19)

```solidity
File: src/factories/ERC20hTokenBranchFactory.sol

12: contract ERC20hTokenBranchFactory is Ownable, IERC20hTokenBranchFactory {

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenBranchFactory.sol#L12)

```solidity
File: src/factories/ERC20hTokenRootFactory.sol

12: contract ERC20hTokenRootFactory is Ownable, IERC20hTokenRootFactory {

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenRootFactory.sol#L12)

```solidity
File: src/token/ERC20hTokenBranch.sol

12: contract ERC20hTokenBranch is ERC20, Ownable, IERC20hTokenBranch {

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/token/ERC20hTokenBranch.sol#L12)

```solidity
File: src/token/ERC20hTokenRoot.sol

12: contract ERC20hTokenRoot is ERC20, Ownable, IERC20hTokenRoot {

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/token/ERC20hTokenRoot.sol#L12)

</details>

### <a name="L-2-1693313958"></a>[L-2] Double type casts create complexity within the code
Double type casting should be avoided in Solidity contracts to prevent unintended consequences and ensure accurate data representation. Performing multiple type casts in succession can lead to unexpected truncation, rounding errors, or loss of precision, potentially compromising the contract's functionality and reliability. Furthermore, double type casting can make the code less readable and harder to maintain, increasing the likelihood of errors and misunderstandings during development and debugging. To ensure precise and consistent data handling, developers should use appropriate data types and avoid unnecessary or excessive type casting, promoting a more robust and dependable contract execution. 

*Instances (5)*:
<details><summary> see instances </summary>

```solidity
File: src/RootBridgeAgentExecutor.sol

125:                     PARAMS_END_OFFSET + uint256(uint8(bytes1(_payload[PARAMS_START]))) * PARAMS_TKN_SET_SIZE_MULTIPLE

213:                         + uint256(uint8(bytes1(_payload[PARAMS_START_SIGNED]))) * PARAMS_TKN_SET_SIZE_MULTIPLE

222:                     + uint256(uint8(bytes1(_payload[PARAMS_START_SIGNED]))) * PARAMS_TKN_SET_SIZE_MULTIPLE

228:                         + uint256(uint8(bytes1(_payload[PARAMS_START_SIGNED]))) * PARAMS_TKN_SET_SIZE_MULTIPLE:

281:         for (uint256 i = 0; i < uint256(uint8(numOfAssets));) {

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L281)

</details>

### <a name="L-3-1694956284"></a>[L-3] Some function should not be marked as payable
Some function should not be marked as payable, otherwise the ETH that mistakenly sent along with the function call is locked in the contract

*Instances (43)*:
<details><summary> see instances </summary>

```solidity
File: src/ArbitrumBranchBridgeAgent.sol

69:     function depositToPort(address underlyingAddress, uint256 amount) external payable lock {
            IArbPort(localPortAddress).depositToPort(msg.sender, msg.sender, underlyingAddress, amount);
        }

79:     function withdrawFromPort(address localAddress, uint256 amount) external payable lock {
            IArbPort(localPortAddress).withdrawFromPort(msg.sender, msg.sender, localAddress, amount);
        }

89:     function retrySettlement(uint32, bytes calldata, GasParams[2] calldata, bool) external payable override lock {}

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchBridgeAgent.sol#L89-L89)

```solidity
File: src/ArbitrumCoreBranchRouter.sol

51:     function addLocalToken(address _underlyingAddress, GasParams calldata) external payable override {
            //Encode Data, no need to create local token since we are already in the global environment
            bytes memory params = abi.encode(
                _underlyingAddress,
                address(0),
                string.concat("Arbitrum Ulysses ", ERC20(_underlyingAddress).name()),
                string.concat("arb-u", ERC20(_underlyingAddress).symbol()),
                ERC20(_underlyingAddress).decimals()
            );
    
            // Pack FuncId
            bytes memory payload = abi.encodePacked(bytes1(0x02), params);
    
            //Send Cross-Chain request (System Response/Request)
            IBridgeAgent(localBridgeAgentAddress).callOutSystem(payable(msg.sender), payload, GasParams(0, 0));
        }

126:     function executeNoSettlement(bytes calldata _data) external payable override requiresAgentExecutor {
             if (_data[0] == 0x02) {
                 (
                     address newBranchRouter,
                     address branchBridgeAgentFactory,
                     address rootBridgeAgent,
                     address rootBridgeAgentFactory,
                     address refundee,
                 ) = abi.decode(_data[1:], (address, address, address, address, address, GasParams));
     
                 _receiveAddBridgeAgent(
                     newBranchRouter,
                     branchBridgeAgentFactory,
                     rootBridgeAgent,
                     rootBridgeAgentFactory,
                     refundee,
                     GasParams(0, 0)
                 );
     
                 /// _toggleBranchBridgeAgentFactory
             } else if (_data[0] == 0x03) {
                 (address bridgeAgentFactoryAddress) = abi.decode(_data[1:], (address));
     
                 _toggleBranchBridgeAgentFactory(bridgeAgentFactoryAddress);
     
                 /// _removeBranchBridgeAgent
             } else if (_data[0] == 0x04) {
                 (address branchBridgeAgent) = abi.decode(_data[1:], (address));
                 _removeBranchBridgeAgent(branchBridgeAgent);
     
                 /// _manageStrategyToken
             } else if (_data[0] == 0x05) {
                 (address underlyingToken, uint256 minimumReservesRatio) = abi.decode(_data[1:], (address, uint256));
                 _manageStrategyToken(underlyingToken, minimumReservesRatio);
     
                 /// _managePortStrategy
             } else if (_data[0] == 0x06) {
                 (address portStrategy, address underlyingToken, uint256 dailyManagementLimit, bool isUpdateDailyLimit) =
                     abi.decode(_data[1:], (address, address, uint256, bool));
                 _managePortStrategy(portStrategy, underlyingToken, dailyManagementLimit, isUpdateDailyLimit);
     
                 /// Unrecognized Function Selector
             } else {
                 revert UnrecognizedFunctionId();
             }
         }

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumCoreBranchRouter.sol#L126-L171)

```solidity
File: src/BaseBranchRouter.sol

123:     function executeNoSettlement(bytes calldata) external payable virtual override requiresAgentExecutor {
             revert UnrecognizedFunctionId();
         }

128:     function executeSettlement(bytes calldata, SettlementParams memory)
             external
             payable
             virtual
             override
             requiresAgentExecutor
         {
             revert UnrecognizedFunctionId();
         }

139:     function executeSettlementMultiple(bytes calldata, SettlementMultipleParams memory)
             external
             payable
             virtual
             override
             requiresAgentExecutor
         {
             revert UnrecognizedFunctionId();
         }

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L139-L147)

```solidity
File: src/BranchBridgeAgent.sol

149:     receive() external payable {}

180:     function callOutSystem(address payable _refundee, bytes calldata _params, GasParams calldata _gParams)
             external
             payable
             override
             lock
             requiresRouter
         {
             //Encode Data for cross-chain call.
             bytes memory payload = abi.encodePacked(bytes1(0x00), depositNonce++, _params);
     
             //Perform Call
             _performCall(_refundee, payload, _gParams);
         }

195:     function callOut(address payable _refundee, bytes calldata _params, GasParams calldata _gParams)
             external
             payable
             override
             lock
         {
             //Encode Data for cross-chain call.
             bytes memory payload = abi.encodePacked(bytes1(0x01), depositNonce++, _params);
     
             //Perform Call
             _performCall(_refundee, payload, _gParams);
         }

209:     function callOutAndBridge(
             address payable _refundee,
             bytes calldata _params,
             DepositInput memory _dParams,
             GasParams calldata _gParams
         ) external payable override lock {
             //Cache Deposit Nonce
             uint32 _depositNonce = depositNonce;
     
             //Encode Data for cross-chain call.
             bytes memory payload = abi.encodePacked(
                 bytes1(0x02), _depositNonce, _dParams.hToken, _dParams.token, _dParams.amount, _dParams.deposit, _params
             );
     
             //Create Deposit and Send Cross-Chain request
             _createDeposit(_depositNonce, _refundee, _dParams.hToken, _dParams.token, _dParams.amount, _dParams.deposit);
     
             //Perform Call
             _performCall(_refundee, payload, _gParams);
         }

231:     function callOutAndBridgeMultiple(
             address payable _refundee,
             bytes calldata _params,
             DepositMultipleInput memory _dParams,
             GasParams calldata _gParams
         ) external payable override lock {
             //Cache Deposit Nonce
             uint32 _depositNonce = depositNonce;
     
             //Encode Data for cross-chain call.
             bytes memory payload = abi.encodePacked(
                 bytes1(0x03),
                 uint8(_dParams.hTokens.length),
                 _depositNonce,
                 _dParams.hTokens,
                 _dParams.tokens,
                 _dParams.amounts,
                 _dParams.deposits,
                 _params
             );
     
             //Create Deposit and Send Cross-Chain request
             _createDepositMultiple(
                 _depositNonce, _refundee, _dParams.hTokens, _dParams.tokens, _dParams.amounts, _dParams.deposits
             );
     
             //Perform Call
             _performCall(_refundee, payload, _gParams);
         }

262:     function callOutSigned(address payable _refundee, bytes calldata _params, GasParams calldata _gParams)
             external
             payable
             override
             lock
         {
             //Encode Data for cross-chain call.
             bytes memory payload = abi.encodePacked(bytes1(0x04), msg.sender, depositNonce++, _params);
     
             //Perform Signed Call without deposit
             _performCall(_refundee, payload, _gParams);
         }

276:     function callOutSignedAndBridge(
             address payable _refundee,
             bytes calldata _params,
             DepositInput memory _dParams,
             GasParams calldata _gParams,
             bool _hasFallbackToggled
         ) external payable override lock {
             //Cache Deposit Nonce
             uint32 _depositNonce = depositNonce;
     
             //Encode Data for cross-chain call.
             bytes memory payload = abi.encodePacked(
                 _hasFallbackToggled ? bytes1(0x85) : bytes1(0x05),
                 msg.sender,
                 _depositNonce,
                 _dParams.hToken,
                 _dParams.token,
                 _dParams.amount,
                 _dParams.deposit,
                 _params
             );
     
             //Create Deposit and Send Cross-Chain request
             _createDeposit(_depositNonce, _refundee, _dParams.hToken, _dParams.token, _dParams.amount, _dParams.deposit);
     
             //Perform Call
             _performCall(_refundee, payload, _gParams);
         }

306:     function callOutSignedAndBridgeMultiple(
             address payable _refundee,
             bytes calldata _params,
             DepositMultipleInput memory _dParams,
             GasParams calldata _gParams,
             bool _hasFallbackToggled
         ) external payable override lock {
             // Cache Deposit Nonce
             uint32 _depositNonce = depositNonce;
     
             // Encode Data for cross-chain call.
             bytes memory payload = abi.encodePacked(
                 _hasFallbackToggled ? bytes1(0x86) : bytes1(0x06),
                 msg.sender,
                 uint8(_dParams.hTokens.length),
                 _depositNonce,
                 _dParams.hTokens,
                 _dParams.tokens,
                 _dParams.amounts,
                 _dParams.deposits,
                 _params
             );
     
             // Create a Deposit and Send Cross-Chain request
             _createDepositMultiple(
                 _depositNonce, _refundee, _dParams.hTokens, _dParams.tokens, _dParams.amounts, _dParams.deposits
             );
     
             //Perform Call
             _performCall(_refundee, payload, _gParams);
         }

343:     function retryDeposit(
             bool _isSigned,
             uint32 _depositNonce,
             bytes calldata _params,
             GasParams calldata _gParams,
             bool _hasFallbackToggled
         ) external payable override lock {
             // Get Settlement Reference
             Deposit storage deposit = getDeposit[_depositNonce];
     
             //Check if deposit belongs to message sender
             if (deposit.owner != msg.sender) revert NotDepositOwner();
     
             //Encode Data for cross-chain call.
             bytes memory payload;
     
             if (uint8(deposit.hTokens.length) == 1) {
                 if (_isSigned) {
                     //Pack new Data
                     payload = abi.encodePacked(
                         _hasFallbackToggled ? bytes1(0x85) : bytes1(0x05),
                         msg.sender,
                         _depositNonce,
                         deposit.hTokens[0],
                         deposit.tokens[0],
                         deposit.amounts[0],
                         deposit.deposits[0],
                         _params
                     );
                 } else {
                     payload = abi.encodePacked(
                         bytes1(0x02),
                         _depositNonce,
                         deposit.hTokens[0],
                         deposit.tokens[0],
                         deposit.amounts[0],
                         deposit.deposits[0],
                         _params
                     );
                 }
             } else if (uint8(deposit.hTokens.length) > 1) {
                 if (_isSigned) {
                     //Pack new Data
                     payload = abi.encodePacked(
                         _hasFallbackToggled ? bytes1(0x86) : bytes1(0x06),
                         msg.sender,
                         uint8(deposit.hTokens.length),
                         _depositNonce,
                         deposit.hTokens,
                         deposit.tokens,
                         deposit.amounts,
                         deposit.deposits,
                         _params
                     );
                 } else {
                     payload = abi.encodePacked(
                         bytes1(0x03),
                         uint8(deposit.hTokens.length),
                         _depositNonce,
                         deposit.hTokens,
                         deposit.tokens,
                         deposit.amounts,
                         deposit.deposits,
                         _params
                     );
                 }
             }
     
             // Check if payload is empty
             if (payload.length == 0) revert DepositRetryUnavailableUseCallout();
     
             // Ensure success Status
             deposit.status = STATUS_SUCCESS;
     
             // Perform Call
             _performCall(payable(msg.sender), payload, _gParams);
         }

422:     function retrieveDeposit(uint32 _depositNonce, GasParams calldata _gParams) external payable override lock {
             // Check if the deposit belongs to the message sender
             if (getDeposit[_depositNonce].owner != msg.sender) revert NotDepositOwner();
     
             //Encode Data for cross-chain call.
             bytes memory payload = abi.encodePacked(bytes1(0x08), msg.sender, _depositNonce);
     
             //Update State and Perform Call
             _performCall(payable(msg.sender), payload, _gParams);
         }

464:     function retrySettlement(
             uint32 _settlementNonce,
             bytes calldata _params,
             GasParams[2] calldata _gParams,
             bool _hasFallbackToggled
         ) external payable virtual override lock {
             // Encode Retry Settlement Params
             bytes memory params = abi.encode(_settlementNonce, msg.sender, _params, _gParams[1]);
     
             // Prepare payload for cross-chain call.
             bytes memory payload = abi.encodePacked(_hasFallbackToggled ? bytes1(0x87) : bytes1(0x07), params);
     
             // Perform Call
             _performCall(payable(msg.sender), payload, _gParams[0]);
         }

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L464-L478)

```solidity
File: src/BranchPort.sol

135:     function renounceOwnership() public payable override onlyOwner {
             revert("Cannot renounce ownership");
         }

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L135-L137)

```solidity
File: src/CoreBranchRouter.sol

86:     function executeNoSettlement(bytes calldata _params) external payable virtual override requiresAgentExecutor {
            /// _receiveAddGlobalToken
            if (_params[0] == 0x01) {
                (
                    address globalAddress,
                    string memory name,
                    string memory symbol,
                    uint8 decimals,
                    address refundee,
                    GasParams memory gParams
                ) = abi.decode(_params[1:], (address, string, string, uint8, address, GasParams));
    
                _receiveAddGlobalToken(globalAddress, name, symbol, decimals, refundee, gParams);
                /// _receiveAddBridgeAgent
            } else if (_params[0] == 0x02) {
                (
                    address newBranchRouter,
                    address branchBridgeAgentFactory,
                    address rootBridgeAgent,
                    address rootBridgeAgentFactory,
                    address refundee,
                    GasParams memory gParams
                ) = abi.decode(_params[1:], (address, address, address, address, address, GasParams));
    
                _receiveAddBridgeAgent(
                    newBranchRouter, branchBridgeAgentFactory, rootBridgeAgent, rootBridgeAgentFactory, refundee, gParams
                );
    
                /// _toggleBranchBridgeAgentFactory
            } else if (_params[0] == 0x03) {
                (address bridgeAgentFactoryAddress) = abi.decode(_params[1:], (address));
    
                _toggleBranchBridgeAgentFactory(bridgeAgentFactoryAddress);
    
                /// _removeBranchBridgeAgent
            } else if (_params[0] == 0x04) {
                (address branchBridgeAgent) = abi.decode(_params[1:], (address));
    
                _removeBranchBridgeAgent(branchBridgeAgent);
    
                /// _manageStrategyToken
            } else if (_params[0] == 0x05) {
                (address underlyingToken, uint256 minimumReservesRatio) = abi.decode(_params[1:], (address, uint256));
    
                _manageStrategyToken(underlyingToken, minimumReservesRatio);
    
                /// _managePortStrategy
            } else if (_params[0] == 0x06) {
                (address portStrategy, address underlyingToken, uint256 dailyManagementLimit, bool isUpdateDailyLimit) =
                    abi.decode(_params[1:], (address, address, uint256, bool));
    
                _managePortStrategy(portStrategy, underlyingToken, dailyManagementLimit, isUpdateDailyLimit);
    
                /// _setCoreBranchRouter
            } else if (_params[0] == 0x07) {
                (address coreBranchRouter, address coreBranchBridgeAgent) = abi.decode(_params[1:], (address, address));
    
                IPort(localPortAddress).setCoreBranchRouter(coreBranchRouter, coreBranchBridgeAgent);
    
                /// Unrecognized Function Selector
            } else {
                revert UnrecognizedFunctionId();
            }
        }

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L86-L149)

```solidity
File: src/CoreRootRouter.sol

297:     function executeResponse(bytes calldata _encodedData, uint16 _srcChainId)
             external
             payable
             override
             requiresExecutor
         {
             // Parse funcId
             bytes1 funcId = _encodedData[0];
     
             ///  FUNC ID: 2 (_addLocalToken)
             if (funcId == 0x02) {
                 (address underlyingAddress, address localAddress, string memory name, string memory symbol, uint8 decimals)
                 = abi.decode(_encodedData[1:], (address, address, string, string, uint8));
     
                 _addLocalToken(underlyingAddress, localAddress, name, symbol, decimals, _srcChainId);
     
                 /// FUNC ID: 3 (_setLocalToken)
             } else if (funcId == 0x03) {
                 (address globalAddress, address localAddress) = abi.decode(_encodedData[1:], (address, address));
     
                 _setLocalToken(globalAddress, localAddress, _srcChainId);
     
                 /// FUNC ID: 4 (_syncBranchBridgeAgent)
             } else if (funcId == 0x04) {
                 (address newBranchBridgeAgent, address rootBridgeAgent) = abi.decode(_encodedData[1:], (address, address));
     
                 _syncBranchBridgeAgent(newBranchBridgeAgent, rootBridgeAgent, _srcChainId);
     
                 /// Unrecognized Function Selector
             } else {
                 revert UnrecognizedFunctionId();
             }
         }

332:     function execute(bytes calldata _encodedData, uint16) external payable override requiresExecutor {
             // Parse funcId
             bytes1 funcId = _encodedData[0];
     
             /// FUNC ID: 1 (_addGlobalToken)
             if (funcId == 0x01) {
                 (address refundee, address globalAddress, uint16 dstChainId, GasParams[2] memory gasParams) =
                     abi.decode(_encodedData[1:], (address, address, uint16, GasParams[2]));
     
                 _addGlobalToken(refundee, globalAddress, dstChainId, gasParams);
     
                 /// Unrecognized Function Selector
             } else {
                 revert UnrecognizedFunctionId();
             }
         }

350:     function executeDepositSingle(bytes memory, DepositParams memory, uint16)
             external
             payable
             override
             requiresExecutor
         {
             revert();
         }

360:     function executeDepositMultiple(bytes calldata, DepositMultipleParams memory, uint16)
             external
             payable
             override
             requiresExecutor
         {
             revert();
         }

370:     function executeSigned(bytes memory, address, uint16) external payable override requiresExecutor {
             revert();
         }

375:     function executeSignedDepositSingle(bytes memory, DepositParams memory, address, uint16)
             external
             payable
             override
             requiresExecutor
         {
             revert();
         }

385:     function executeSignedDepositMultiple(bytes memory, DepositMultipleParams memory, address, uint16)
             external
             payable
             override
             requiresExecutor
         {
             revert();
         }

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L385-L392)

```solidity
File: src/MulticallRootRouter.sol

123:     function executeResponse(bytes memory, uint16) external payable override {
             revert();
         }

137:     function execute(bytes calldata encodedData, uint16) external payable override lock requiresExecutor {
             // Parse funcId
             bytes1 funcId = encodedData[0];
     
             /// FUNC ID: 1 (multicallNoOutput)
             if (funcId == 0x01) {
                 // Decode Params
                 (IMulticall.Call[] memory callData) = abi.decode(_decode(encodedData[1:]), (IMulticall.Call[]));
     
                 // Perform Calls
                 _multicall(callData);
     
                 /// FUNC ID: 2 (multicallSingleOutput)
             } else if (funcId == 0x02) {
                 // Decode Params
                 (
                     IMulticall.Call[] memory callData,
                     OutputParams memory outputParams,
                     uint16 dstChainId,
                     GasParams memory gasParams
                 ) = abi.decode(_decode(encodedData[1:]), (IMulticall.Call[], OutputParams, uint16, GasParams));
     
                 // Perform Calls
                 _multicall(callData);
     
                 // Bridge Out assets
                 _approveAndCallOut(
                     outputParams.recipient,
                     outputParams.recipient,
                     outputParams.outputToken,
                     outputParams.amountOut,
                     outputParams.depositOut,
                     dstChainId,
                     gasParams
                 );
     
                 /// FUNC ID: 3 (multicallMultipleOutput)
             } else if (funcId == 0x03) {
                 // Decode Params
                 (
                     IMulticall.Call[] memory callData,
                     OutputMultipleParams memory outputParams,
                     uint16 dstChainId,
                     GasParams memory gasParams
                 ) = abi.decode(_decode(encodedData[1:]), (IMulticall.Call[], OutputMultipleParams, uint16, GasParams));
     
                 // Perform Calls
                 _multicall(callData);
     
                 // Bridge Out assets
                 _approveMultipleAndCallOut(
                     outputParams.recipient,
                     outputParams.recipient,
                     outputParams.outputTokens,
                     outputParams.amountsOut,
                     outputParams.depositsOut,
                     dstChainId,
                     gasParams
                 );
                 /// UNRECOGNIZED FUNC ID
             } else {
                 revert UnrecognizedFunctionId();
             }
         }

203:     function executeDepositSingle(bytes calldata, DepositParams calldata, uint16) external payable override {
             revert();
         }

209:     function executeDepositMultiple(bytes calldata, DepositMultipleParams calldata, uint16) external payable {
             revert();
         }

223:     function executeSigned(bytes calldata encodedData, address userAccount, uint16)
             external
             payable
             override
             lock
             requiresExecutor
         {
             // Parse funcId
             bytes1 funcId = encodedData[0];
     
             /// FUNC ID: 1 (multicallNoOutput)
             if (funcId == 0x01) {
                 // Decode Params
                 Call[] memory calls = abi.decode(_decode(encodedData[1:]), (Call[]));
     
                 // Make requested calls
                 IVirtualAccount(userAccount).call(calls);
     
                 /// FUNC ID: 2 (multicallSingleOutput)
             } else if (funcId == 0x02) {
                 // Decode Params
                 (Call[] memory calls, OutputParams memory outputParams, uint16 dstChainId, GasParams memory gasParams) =
                     abi.decode(_decode(encodedData[1:]), (Call[], OutputParams, uint16, GasParams));
     
                 // Make requested calls
                 IVirtualAccount(userAccount).call(calls);
     
                 // Withdraw assets from Virtual Account
                 IVirtualAccount(userAccount).withdrawERC20(outputParams.outputToken, outputParams.amountOut);
     
                 // Bridge Out assets
                 _approveAndCallOut(
                     IVirtualAccount(userAccount).userAddress(),
                     outputParams.recipient,
                     outputParams.outputToken,
                     outputParams.amountOut,
                     outputParams.depositOut,
                     dstChainId,
                     gasParams
                 );
     
                 /// FUNC ID: 3 (multicallMultipleOutput)
             } else if (funcId == 0x03) {
                 // Decode Params
                 (
                     Call[] memory calls,
                     OutputMultipleParams memory outputParams,
                     uint16 dstChainId,
                     GasParams memory gasParams
                 ) = abi.decode(_decode(encodedData[1:]), (Call[], OutputMultipleParams, uint16, GasParams));
     
                 // Make requested calls
                 IVirtualAccount(userAccount).call(calls);
     
                 // Withdraw assets from Virtual Account
                 for (uint256 i = 0; i < outputParams.outputTokens.length;) {
                     IVirtualAccount(userAccount).withdrawERC20(outputParams.outputTokens[i], outputParams.amountsOut[i]);
     
                     unchecked {
                         ++i;
                     }
                 }
     
                 // Bridge Out assets
                 _approveMultipleAndCallOut(
                     IVirtualAccount(userAccount).userAddress(),
                     outputParams.recipient,
                     outputParams.outputTokens,
                     outputParams.amountsOut,
                     outputParams.depositsOut,
                     dstChainId,
                     gasParams
                 );
                 /// UNRECOGNIZED FUNC ID
             } else {
                 revert UnrecognizedFunctionId();
             }
         }

312:     function executeSignedDepositSingle(bytes calldata encodedData, DepositParams calldata, address userAccount, uint16)
             external
             payable
             override
             requiresExecutor
             lock
         {
             // Parse funcId
             bytes1 funcId = encodedData[0];
     
             /// FUNC ID: 1 (multicallNoOutput)
             if (funcId == 0x01) {
                 // Decode Params
                 Call[] memory calls = abi.decode(_decode(encodedData[1:]), (Call[]));
     
                 // Make requested calls
                 IVirtualAccount(userAccount).call(calls);
     
                 /// FUNC ID: 2 (multicallSingleOutput)
             } else if (funcId == 0x02) {
                 // Decode Params
                 (Call[] memory calls, OutputParams memory outputParams, uint16 dstChainId, GasParams memory gasParams) =
                     abi.decode(_decode(encodedData[1:]), (Call[], OutputParams, uint16, GasParams));
     
                 // Make requested calls
                 IVirtualAccount(userAccount).call(calls);
     
                 // Withdraw assets from Virtual Account
                 IVirtualAccount(userAccount).withdrawERC20(outputParams.outputToken, outputParams.amountOut);
     
                 // Bridge Out assets
                 _approveAndCallOut(
                     IVirtualAccount(userAccount).userAddress(),
                     outputParams.recipient,
                     outputParams.outputToken,
                     outputParams.amountOut,
                     outputParams.depositOut,
                     dstChainId,
                     gasParams
                 );
     
                 /// FUNC ID: 3 (multicallMultipleOutput)
             } else if (funcId == 0x03) {
                 // Decode Params
                 (
                     Call[] memory calls,
                     OutputMultipleParams memory outputParams,
                     uint16 dstChainId,
                     GasParams memory gasParams
                 ) = abi.decode(_decode(encodedData[1:]), (Call[], OutputMultipleParams, uint16, GasParams));
     
                 // Make requested calls
                 IVirtualAccount(userAccount).call(calls);
     
                 // Withdraw assets from Virtual Account
                 for (uint256 i = 0; i < outputParams.outputTokens.length;) {
                     IVirtualAccount(userAccount).withdrawERC20(outputParams.outputTokens[i], outputParams.amountsOut[i]);
     
                     unchecked {
                         ++i;
                     }
                 }
     
                 // Bridge Out assets
                 _approveMultipleAndCallOut(
                     IVirtualAccount(userAccount).userAddress(),
                     outputParams.recipient,
                     outputParams.outputTokens,
                     outputParams.amountsOut,
                     outputParams.depositsOut,
                     dstChainId,
                     gasParams
                 );
                 /// UNRECOGNIZED FUNC ID
             } else {
                 revert UnrecognizedFunctionId();
             }
         }

401:     function executeSignedDepositMultiple(
             bytes calldata encodedData,
             DepositMultipleParams calldata,
             address userAccount,
             uint16
         ) external payable override requiresExecutor lock {
             // Parse funcId
             bytes1 funcId = encodedData[0];
     
             /// FUNC ID: 1 (multicallNoOutput)
             if (funcId == 0x01) {
                 // Decode Params
                 Call[] memory calls = abi.decode(_decode(encodedData[1:]), (Call[]));
     
                 // Make requested calls
                 IVirtualAccount(userAccount).call(calls);
     
                 /// FUNC ID: 2 (multicallSingleOutput)
             } else if (funcId == 0x02) {
                 // Decode Params
                 (Call[] memory calls, OutputParams memory outputParams, uint16 dstChainId, GasParams memory gasParams) =
                     abi.decode(_decode(encodedData[1:]), (Call[], OutputParams, uint16, GasParams));
     
                 // Make requested calls
                 IVirtualAccount(userAccount).call(calls);
     
                 // Withdraw assets from Virtual Account
                 IVirtualAccount(userAccount).withdrawERC20(outputParams.outputToken, outputParams.amountOut);
     
                 // Bridge Out assets
                 _approveAndCallOut(
                     IVirtualAccount(userAccount).userAddress(),
                     outputParams.recipient,
                     outputParams.outputToken,
                     outputParams.amountOut,
                     outputParams.depositOut,
                     dstChainId,
                     gasParams
                 );
     
                 /// FUNC ID: 3 (multicallMultipleOutput)
             } else if (funcId == 0x03) {
                 // Decode Params
                 (
                     Call[] memory calls,
                     OutputMultipleParams memory outputParams,
                     uint16 dstChainId,
                     GasParams memory gasParams
                 ) = abi.decode(_decode(encodedData[1:]), (Call[], OutputMultipleParams, uint16, GasParams));
     
                 // Make requested calls
                 IVirtualAccount(userAccount).call(calls);
     
                 // Withdraw assets from Virtual Account
                 for (uint256 i = 0; i < outputParams.outputTokens.length;) {
                     IVirtualAccount(userAccount).withdrawERC20(outputParams.outputTokens[i], outputParams.amountsOut[i]);
     
                     unchecked {
                         ++i;
                     }
                 }
     
                 // Bridge Out assets
                 _approveMultipleAndCallOut(
                     IVirtualAccount(userAccount).userAddress(),
                     outputParams.recipient,
                     outputParams.outputTokens,
                     outputParams.amountsOut,
                     outputParams.depositsOut,
                     dstChainId,
                     gasParams
                 );
                 /// UNRECOGNIZED FUNC ID
             } else {
                 revert UnrecognizedFunctionId();
             }
         }

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L401-L477)

```solidity
File: src/RootBridgeAgent.sol

128:     receive() external payable {}

160:     function callOut(
             address payable _refundee,
             address _recipient,
             uint16 _dstChainId,
             bytes calldata _params,
             GasParams calldata _gParams
         ) external payable override lock requiresRouter {
             //Encode Data for call.
             bytes memory payload = abi.encodePacked(bytes1(0x00), _recipient, settlementNonce++, _params);
     
             //Perform Call to clear hToken balance on destination branch chain.
             _performCall(_dstChainId, _refundee, payload, _gParams);
         }

175:     function callOutAndBridge(
             address payable _refundee,
             address _recipient,
             uint16 _dstChainId,
             bytes calldata _params,
             SettlementInput calldata _sParams,
             GasParams calldata _gParams,
             bool _hasFallbackToggled
         ) external payable override lock requiresRouter {
             // Create Settlement and Perform call
             bytes memory payload = _createSettlement(
                 settlementNonce,
                 _refundee,
                 _recipient,
                 _dstChainId,
                 _params,
                 _sParams.globalAddress,
                 _sParams.amount,
                 _sParams.deposit,
                 _hasFallbackToggled
             );
     
             //Perform Call.
             _performCall(_dstChainId, _refundee, payload, _gParams);
         }

202:     function callOutAndBridgeMultiple(
             address payable _refundee,
             address _recipient,
             uint16 _dstChainId,
             bytes calldata _params,
             SettlementMultipleInput calldata _sParams,
             GasParams calldata _gParams,
             bool _hasFallbackToggled
         ) external payable override lock requiresRouter {
             // Create Settlement and Perform call
             bytes memory payload = _createSettlementMultiple(
                 settlementNonce,
                 _refundee,
                 _recipient,
                 _dstChainId,
                 _sParams.globalAddresses,
                 _sParams.amounts,
                 _sParams.deposits,
                 _params,
                 _hasFallbackToggled
             );
     
             // Perform Call to destination Branch Chain.
             _performCall(_dstChainId, _refundee, payload, _gParams);
         }

274:     function retrieveSettlement(uint32 _settlementNonce, GasParams calldata _gParams) external payable lock {
             //Get settlement storage reference
             Settlement storage settlementReference = getSettlement[_settlementNonce];
     
             // Get Settlement owner.
             address settlementOwner = settlementReference.owner;
     
             // Check if Settlement is Retrieve.
             if (settlementOwner == address(0)) revert SettlementRetrieveUnavailable();
     
             // Check Settlement Owner
             if (msg.sender != settlementOwner) {
                 if (msg.sender != address(IPort(localPortAddress).getUserAccount(settlementOwner))) {
                     revert NotSettlementOwner();
                 }
             }
     
             //Encode Data for cross-chain call.
             bytes memory payload = abi.encodePacked(bytes1(0x03), settlementOwner, _settlementNonce);
     
             //Retrieve Deposit
             _performCall(settlementReference.dstChainId, payable(settlementOwner), payload, _gParams);
         }

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L274-L296)

```solidity
File: src/RootBridgeAgentExecutor.sol

50:     function executeSystemRequest(address _router, bytes calldata _payload, uint16 _srcChainId)
            external
            payable
            onlyOwner
        {
            //Try to execute remote request
            IRouter(_router).executeResponse(_payload[PARAMS_TKN_START:], _srcChainId);
        }

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L50-L57)

```solidity
File: src/RootPort.sol

166:     function renounceOwnership() public payable override onlyOwner {
             revert("Cannot renounce ownership");
         }

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L166-L168)

```solidity
File: src/VirtualAccount.sol

44:     receive() external payable {}

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L44-L44)

</details>

### <a name="L-4-1693314112"></a>[L-4] increase/decrease or forceApprove allowance should be used instead of approve
In order to prevent front running, increase/decrease or forceApprove allowance should be used in place of approve where possible 

*Instances (2)*:
<details><summary> see instances </summary>

```solidity
File: src/BaseBranchRouter.sol

168:                 ERC20(_hToken).approve(_localPortAddress, _amount - _deposit);

175:             ERC20(_token).approve(_localPortAddress, _deposit);

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L175)

</details>

### <a name="L-5-1693314276"></a>[L-5] `onlyOwner` functions not accessible if `owner` renounces ownership
The `owner` is able to perform certain privileged activities, but it's possible to set the owner to `address(0)`. This can represent a certain risk if the ownership is renounced for any other reason than by design.
Renouncing ownership will leave the contract without an `owner`, therefore limiting any functionality that needs authority.

*Instances (24)*:
<details><summary> see instances </summary>

```solidity
File: src/BaseBranchRouter.sol

60:     function initialize(address _localBridgeAgentAddress) external onlyOwner {

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L60)

```solidity
File: src/BranchBridgeAgentExecutor.sol

53:     function executeNoSettlement(address _router, bytes calldata _payload) external payable onlyOwner {

66:     function executeWithSettlement(address _recipient, address _router, bytes calldata _payload)

104:     function executeWithSettlementMultiple(address _recipient, address _router, bytes calldata _payload)

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgentExecutor.sol#L104)

```solidity
File: src/CoreRootRouter.sol

83:     function initialize(address _bridgeAgentAddress, address _hTokenFactory) external onlyOwner {

156:     function toggleBranchBridgeAgentFactory(

186:     function removeBranchBridgeAgent(

212:     function manageStrategyToken(

241:     function managePortStrategy(

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L241)

```solidity
File: src/MulticallRootRouter.sol

109:     function initialize(address _bridgeAgentAddress) external onlyOwner {

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L109)

```solidity
File: src/RootBridgeAgentExecutor.sol

50:     function executeSystemRequest(address _router, bytes calldata _payload, uint16 _srcChainId)

66:     function executeNoDeposit(address _router, bytes calldata _payload, uint16 _srcChainId)

82:     function executeWithDeposit(address _router, bytes calldata _payload, uint16 _srcChainId)

115:     function executeWithDepositMultiple(address _router, bytes calldata _payload, uint16 _srcChainId)

150:     function executeSignedNoDeposit(address _account, address _router, bytes calldata _payload, uint16 _srcChainId)

167:     function executeSignedWithDeposit(address _account, address _router, bytes calldata _payload, uint16 _srcChainId)

201:     function executeSignedWithDepositMultiple(

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L201)

```solidity
File: src/factories/BranchBridgeAgentFactory.sol

87:     function initialize(address _coreRootBridgeAgent) external virtual onlyOwner {

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/BranchBridgeAgentFactory.sol#L87)

```solidity
File: src/factories/ERC20hTokenBranchFactory.sol

60:     function initialize(address _wrappedNativeTokenAddress, address _coreRouter) external onlyOwner {

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenBranchFactory.sol#L60)

```solidity
File: src/factories/ERC20hTokenRootFactory.sol

49:     function initialize(address _coreRouter) external onlyOwner {

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenRootFactory.sol#L49)

```solidity
File: src/token/ERC20hTokenBranch.sol

29:     function mint(address account, uint256 amount) external override onlyOwner returns (bool) {

35:     function burn(uint256 amount) public override onlyOwner {

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/token/ERC20hTokenBranch.sol#L35)

```solidity
File: src/token/ERC20hTokenRoot.sol

57:     function mint(address to, uint256 amount, uint256 chainId) external onlyOwner returns (bool) {

69:     function burn(address from, uint256 amount, uint256 chainId) external onlyOwner {

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/token/ERC20hTokenRoot.sol#L69)

</details>

### <a name="L-6-1693314386"></a>[L-6] Consider using OpenZeppelin’s SafeCast library to prevent unexpected overflows when casting from various type int/uint values

*Instances (2)*:
<details><summary> see instances </summary>

```solidity
File: src/RootBridgeAgentExecutor.sol

//@audit `numOfAssets` is getting converted from `uint256` to `uint256`
137:                 _payload[PARAMS_END_OFFSET + uint256(numOfAssets) * PARAMS_TKN_SET_SIZE_MULTIPLE:], dParams, _srcChainId

//@audit `numOfAssets` is getting converted from `uint8` to `uint8`
281:         for (uint256 i = 0; i < uint256(uint8(numOfAssets));) {

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L281)

</details>

### <a name="L-7-1693315091"></a>[L-7] Gas grief possible on unsafe external calls
In Solidity, the use of low-level `call` methods can expose contracts to gas griefing attacks. The potential problem arises when the callee contract returns a large amount of data. This data is allocated in the memory of the calling contract, which pays for the gas costs. If the callee contract intentionally returns an enormous amount of data, the gas costs can skyrocket, causing the transaction to fail due to an Out of Gas error. Therefore, it's advisable to limit the use of `call` when interacting with untrusted contracts, or ensure that the callee's returned data size is capped or known in advance to prevent unexpected high gas costs. 
Now `(bool success, )` is actually the same as writing `(bool success, bytes memory data)` which basically means that even though the data is omitted it doesn't mean that the contract does not handle it. Actually, the way it works is the `bytes data` that was returned from the receiver will be copied to memory. Memory allocation becomes very costly if the payload is big

*Instances (18)*:
<details><summary> see instances </summary>

```solidity
File: src/ArbitrumBranchBridgeAgent.sol

103:         _rootBridgeAgentAddress.call{value: msg.value}("");

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchBridgeAgent.sol#L103)

```solidity
File: src/BranchBridgeAgent.sol

717:         (bool success,) = bridgeAgentExecutorAddress.call{value: address(this).balance}(_calldata);

737:         (bool success,) = bridgeAgentExecutorAddress.call{value: address(this).balance}(_calldata);

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L737)

```solidity
File: src/MulticallRootRouter.sol

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
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L452)

```solidity
File: src/RootBridgeAgent.sol

754:         (bool success,) = bridgeAgentExecutorAddress.call{value: address(this).balance}(_calldata);

779:         (bool success,) = bridgeAgentExecutorAddress.call{value: address(this).balance}(_calldata);

835:             callee.call{value: msg.value}("");

927:             callee.call{value: _value}("");

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L927)

```solidity
File: src/VirtualAccount.sol

74:             if (isContract(_call.target)) (success, returnData[i]) = _call.target.call(_call.callData);

101:             if (isContract(_call.target)) (success, returnData[i]) = _call.target.call{value: val}(_call.callData);

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L101)

</details>



## Non Critical Issues


| |Issue|Instances|
|-|:-|:-:|
| [NC-1](#NC-1-1693315336) | Consider using modifiers for address control | 3 |
| [NC-2](#NC-2-1693315622) | Variable names don't follow the Solidity style guide | 39 |
| [NC-3](#NC-3-1693315664) | Contracts containing only utility functions should be made into libraries | 1 |
| [NC-4](#NC-4-1693315680) | Control structures do not follow the Solidity Style Guide | 78 |
| [NC-5](#NC-5-1693315718) | Consider using `delete` rather than assigning `false` to clear values | 3 |
| [NC-6](#NC-6-1693315838) | Empty bytes check is missing | 87 |
| [NC-7](#NC-7-1694440518) | High cyclomatic complexity | 6 |
| [NC-8](#NC-8-1693316104) | Some if-statement can be converted to a ternary | 4 |
| [NC-9](#NC-9-1693316148) | Contract implements interface without extending the interface | 11 |
| [NC-10](#NC-10-1693316207) | Inconsistent usage of `require`/`error` | 54 |
| [NC-11](#NC-11-1693316644) | Some error strings are not descriptive | 1 |
| [NC-12](#NC-12-1693317147) | State variables should include comments | 22 |
| [NC-13](#NC-13-1693317214) | Top level pragma declarations should be separated by two blank lines | 37 |
| [NC-14](#NC-14-1693317342) | Uncommented fields in a struct | 7 |
| [NC-15](#NC-15-1693317551) | Missing upgradability functionality | 1 |
| [NC-16](#NC-16-1693317704) | Use a single file for system wide constants | 2 |
### <a name="NC-1-1693315336"></a>[NC-1] Consider using modifiers for address control
Modifiers in Solidity can improve code readability and modularity by encapsulating repetitive checks, such as address validity checks, into a reusable construct. For example, an `onlyOwner` modifier can be used to replace repetitive `require(msg.sender == owner)` checks across several functions, reducing code redundancy and enhancing maintainability. To implement, define a modifier with the check, then apply the modifier to relevant functions.

*Instances (3)*:
<details><summary> see instances </summary>

```solidity
File: src/CoreRootRouter.sol

278:         require(msg.sender == rootPortAddress, "Only root port can call");

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L278-L278)

```solidity
File: src/factories/ArbitrumBranchBridgeAgentFactory.sol

85:             msg.sender == localCoreBranchRouterAddress, "Only the Core Branch Router can create a new Bridge Agent."

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ArbitrumBranchBridgeAgentFactory.sol#L85-L85)

```solidity
File: src/factories/BranchBridgeAgentFactory.sol

121:             msg.sender == localCoreBranchRouterAddress, "Only the Core Branch Router can create a new Bridge Agent."

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/BranchBridgeAgentFactory.sol#L121-L121)

</details>

### <a name="NC-2-1693315622"></a>[NC-2] Variable names don't follow the Solidity style guide
For `constant` and `immutable` variable names, each word should use all capital letters, with underscores separating each word (CONSTANT_CASE)

*Instances (39)*:
<details><summary> see instances </summary>

```solidity
File: src/ArbitrumBranchPort.sol

22:     uint16 public immutable localChainId;

26:     address public immutable rootPortAddress;

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol#L26)

```solidity
File: src/BranchBridgeAgent.sol

53:     uint16 public immutable rootChainId;

56:     uint16 public immutable localChainId;

60:     address public immutable rootBridgeAgentAddress;

67:     address public immutable lzEndpointAddress;

70:     address public immutable localRouterAddress;

74:     address public immutable localPortAddress;

77:     address public immutable bridgeAgentExecutorAddress;

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L77)

```solidity
File: src/CoreBranchRouter.sol

20:     address public immutable hTokenFactoryAddress;

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L20)

```solidity
File: src/CoreRootRouter.sol

47:     uint256 public immutable rootChainId;

51:     address public immutable rootPortAddress;

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L51)

```solidity
File: src/MulticallRootRouter.sol

65:     uint256 public immutable localChainId;

68:     address public immutable localPortAddress;

71:     address public immutable multicallAddress;

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L71)

```solidity
File: src/RootBridgeAgent.sol

40:     uint16 public immutable localChainId;

43:     address public immutable factoryAddress;

46:     address public immutable localRouterAddress;

49:     address public immutable localPortAddress;

52:     address public immutable lzEndpointAddress;

55:     address public immutable bridgeAgentExecutorAddress;

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L55)

```solidity
File: src/RootPort.sol

34:     uint256 public immutable localChainId;

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L34)

```solidity
File: src/VirtualAccount.sol

21:     address public immutable override userAddress;

24:     address public immutable override localPortAddress;

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L24)

```solidity
File: src/factories/BranchBridgeAgentFactory.sol

21:     uint16 public immutable localChainId;

24:     uint16 public immutable rootChainId;

27:     address public immutable rootBridgeAgentFactoryAddress;

30:     address public immutable localCoreBranchRouterAddress;

33:     address public immutable localPortAddress;

36:     address public immutable lzEndpointAddress;

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/BranchBridgeAgentFactory.sol#L36)

```solidity
File: src/factories/ERC20hTokenBranchFactory.sol

14:     uint24 public immutable localChainId;

17:     address public immutable localPortAddress;

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenBranchFactory.sol#L17)

```solidity
File: src/factories/ERC20hTokenRootFactory.sol

14:     uint16 public immutable localChainId;

17:     address public immutable rootPortAddress;

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenRootFactory.sol#L17)

```solidity
File: src/factories/RootBridgeAgentFactory.sol

13:     uint16 public immutable rootChainId;

16:     address public immutable rootPortAddress;

19:     address public immutable lzEndpointAddress;

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/RootBridgeAgentFactory.sol#L19)

```solidity
File: src/token/ERC20hTokenRoot.sol

14:     uint16 public immutable override localChainId;

17:     address public immutable override factoryAddress;

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/token/ERC20hTokenRoot.sol#L17)

</details>

### <a name="NC-3-1693315664"></a>[NC-3] Contracts containing only utility functions should be made into libraries

*Instances (1)*:
<details><summary> see instances </summary>

```solidity
File: src/MulticallRootRouterLibZip.sol

//@audit the contract `MulticallRootRouterLibZip` contain only utility functions, it should turn to a `library`
26: contract MulticallRootRouterLibZip is MulticallRootRouter {

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouterLibZip.sol#L26)

</details>

### <a name="NC-4-1693315680"></a>[NC-4] Control structures do not follow the Solidity Style Guide
See the [control structures](https://docs.soliditylang.org/en/latest/style-guide.html#control-structures) section of the Solidity Style Guide

*Instances (78)*:
<details><summary> see instances </summary>

```solidity
File: src/ArbitrumBranchPort.sol

119:     function _bridgeOut(

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol#L119)

```solidity
File: src/ArbitrumCoreBranchRouter.sol

85:     function _receiveAddBridgeAgent(

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumCoreBranchRouter.sol#L85)

```solidity
File: src/BaseBranchRouter.sol

104:     function callOutAndBridgeMultiple(

186:     function _transferAndApproveMultipleTokens(

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L186)

```solidity
File: src/BranchBridgeAgent.sol

24:     function deploy(

209:     function callOutAndBridge(

231:     function callOutAndBridgeMultiple(

276:     function callOutSignedAndBridge(

306:     function callOutSignedAndBridgeMultiple(

343:     function retryDeposit(

464:     function retrySettlement(

812:     function _createDeposit(

860:     function _createDepositMultiple(

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L860)

```solidity
File: src/BranchPort.sol

246:     function bridgeInMultiple(

277:     function bridgeOut(

288:     function bridgeOutMultiple(

514:     function _bridgeOut(

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L514)

```solidity
File: src/CoreBranchRouter.sol

166:     function _receiveAddGlobalToken(

199:     function _receiveAddBridgeAgent(

307:     function _managePortStrategy(

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L307)

```solidity
File: src/CoreRootRouter.sol

103:     function addBranchToBridgeAgent(

156:     function toggleBranchBridgeAgentFactory(

186:     function removeBranchBridgeAgent(

212:     function manageStrategyToken(

241:     function managePortStrategy(

270:     function setCoreBranch(

406:     function _addGlobalToken(

452:     function _addLocalToken(

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L452)

```solidity
File: src/MulticallRootRouter.sol

401:     function executeSignedDepositMultiple(

508:     function _approveAndCallOut(

544:     function _approveMultipleAndCallOut(

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L544)

```solidity
File: src/RootBridgeAgent.sol

140:     function getFeeEstimate(

160:     function callOut(

175:     function callOutAndBridge(

202:     function callOutAndBridgeMultiple(

233:     function retrySettlement(

434:     function lzReceiveNonBlocking(

768:     function _execute(

808:     function _performCall(

856:     function _performRetrySettlementCall(

966:     function _createSettlement(

1045:     function _createSettlementMultiple(

1131:     function _updateStateOnBridgeOut(

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1131)

```solidity
File: src/RootBridgeAgentExecutor.sol

201:     function executeSignedWithDepositMultiple(

219:         if (

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L219)

```solidity
File: src/RootPort.sol

147:     function initializeCore(

239:     function setAddresses(

393:     function syncBranchBridgeAgentWithRoot(

438:     function addNewChain(

521:     function setCoreBranchRouter(

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L521)

```solidity
File: src/factories/ArbitrumBranchBridgeAgentFactory.sol

79:     function createBridgeAgent(

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ArbitrumBranchBridgeAgentFactory.sol#L79)

```solidity
File: src/factories/BranchBridgeAgentFactory.sol

115:     function createBridgeAgent(

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/BranchBridgeAgentFactory.sol#L115)

```solidity
File: src/interfaces/IBranchBridgeAgent.sol

163:     function callOutAndBridge(

179:     function callOutAndBridgeMultiple(

208:     function callOutSignedAndBridge(

227:     function callOutSignedAndBridgeMultiple(

248:     function retryDeposit(

285:     function retrySettlement(

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/IBranchBridgeAgent.sol#L285)

```solidity
File: src/interfaces/IBranchBridgeAgentFactory.sol

24:     function createBridgeAgent(

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/IBranchBridgeAgentFactory.sol#L24)

```solidity
File: src/interfaces/IBranchPort.sol

103:     function bridgeInMultiple(

118:     function bridgeOut(

135:     function bridgeOutMultiple(

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/IBranchPort.sol#L135)

```solidity
File: src/interfaces/IBranchRouter.sol

68:     function callOutAndBridgeMultiple(

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/IBranchRouter.sol#L68)

```solidity
File: src/interfaces/ILayerZeroEndpoint.sol

15:     function send(

31:     function receivePayload(

55:     function estimateFees(

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/ILayerZeroEndpoint.sol#L55)

```solidity
File: src/interfaces/IRootBridgeAgent.sol

149:     function getFeeEstimate(

168:     function callOut(

189:     function callOutAndBridge(

213:     function callOutAndBridgeMultiple(

236:     function retrySettlement(

309:     function lzReceiveNonBlocking(

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/IRootBridgeAgent.sol#L309)

```solidity
File: src/interfaces/IRootPort.sol

13:     function setCoreBranch(

238:     function setAddresses(

319:     function addNewChain(

350:     function setCoreBranchRouter(

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/IRootPort.sol#L350)

```solidity
File: src/interfaces/IRootRouter.sol

72:     function executeSignedDepositSingle(

86:     function executeSignedDepositMultiple(

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/IRootRouter.sol#L86)

</details>

### <a name="NC-5-1693315718"></a>[NC-5] Consider using `delete` rather than assigning `false` to clear values
The `delete` keyword more closely matches the semantics of what is being done, and draws more attention to the changing of state, which may lead to a more thorough audit of its associated logic

*Instances (3)*:
<details><summary> see instances </summary>

```solidity
File: src/CoreRootRouter.sol

85:         _setup = false;

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L85)

```solidity
File: src/RootPort.sol

133:         _setup = false;

157:         _setupCore = false;

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L157)

</details>

### <a name="NC-6-1693315838"></a>[NC-6] Empty bytes check is missing
When developing smart contracts in Solidity, it's crucial to validate the inputs of your functions. This includes ensuring that the bytes parameters are not empty, especially when they represent crucial data such as addresses, identifiers, or raw data that the contract needs to process.
Missing empty bytes checks can lead to unexpected behaviour in your contract.For instance, certain operations might fail, produce incorrect results, or consume unnecessary gas when performed with empty bytes.Moreover, missing input validation can potentially expose your contract to malicious activity, including exploitation of unhandled edge cases.
To mitigate these issues, always validate that bytes parameters are not empty when the logic of your contract requires it.

*Instances (87)*:
<details><summary> see instances </summary>

```solidity
File: src/ArbitrumBranchBridgeAgent.sol

//@audit  ,_calldata are not checked
99:     function _performCall(address payable, bytes memory _calldata, GasParams calldata) internal override {
            // Cache Root Bridge Agent Address

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchBridgeAgent.sol#L99-L100)

```solidity
File: src/BaseBranchRouter.sol

//@audit  ,_params are not checked
83:     function callOut(bytes calldata _params, GasParams calldata _gParams) external payable override lock {
            IBridgeAgent(localBridgeAgentAddress).callOut{value: msg.value}(payable(msg.sender), _params, _gParams);

//@audit  ,_params are not checked
88:     function callOutAndBridge(bytes calldata _params, DepositInput calldata _dParams, GasParams calldata _gParams)
            external
            payable
            override
            lock
        {

//@audit  ,_params are not checked
104:     function callOutAndBridgeMultiple(
             bytes calldata _params,
             DepositMultipleInput calldata _dParams,
             GasParams calldata _gParams
         ) external payable override lock {

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L104-L108)

```solidity
File: src/BranchBridgeAgent.sol

//@audit  ,_payload are not checked
161:     function getFeeEstimate(uint256 _gasLimit, uint256 _remoteBranchExecutionGas, bytes calldata _payload)
             external
             view
             returns (uint256 _fee)
         {

//@audit  ,_params are not checked
180:     function callOutSystem(address payable _refundee, bytes calldata _params, GasParams calldata _gParams)
             external
             payable
             override
             lock
             requiresRouter
         {

//@audit  ,_params are not checked
195:     function callOut(address payable _refundee, bytes calldata _params, GasParams calldata _gParams)
             external
             payable
             override
             lock
         {

//@audit  ,_params are not checked
209:     function callOutAndBridge(
             address payable _refundee,
             bytes calldata _params,
             DepositInput memory _dParams,
             GasParams calldata _gParams
         ) external payable override lock {

//@audit  ,_params are not checked
231:     function callOutAndBridgeMultiple(
             address payable _refundee,
             bytes calldata _params,
             DepositMultipleInput memory _dParams,
             GasParams calldata _gParams
         ) external payable override lock {

//@audit  ,_params are not checked
262:     function callOutSigned(address payable _refundee, bytes calldata _params, GasParams calldata _gParams)
             external
             payable
             override
             lock
         {

//@audit  ,_params are not checked
276:     function callOutSignedAndBridge(
             address payable _refundee,
             bytes calldata _params,
             DepositInput memory _dParams,
             GasParams calldata _gParams,
             bool _hasFallbackToggled
         ) external payable override lock {

//@audit  ,_params are not checked
306:     function callOutSignedAndBridgeMultiple(
             address payable _refundee,
             bytes calldata _params,
             DepositMultipleInput memory _dParams,
             GasParams calldata _gParams,
             bool _hasFallbackToggled
         ) external payable override lock {

//@audit  ,_params are not checked
343:     function retryDeposit(
             bool _isSigned,
             uint32 _depositNonce,
             bytes calldata _params,
             GasParams calldata _gParams,
             bool _hasFallbackToggled
         ) external payable override lock {

//@audit  ,_params are not checked
464:     function retrySettlement(
             uint32 _settlementNonce,
             bytes calldata _params,
             GasParams[2] calldata _gParams,
             bool _hasFallbackToggled
         ) external payable virtual override lock {

//@audit  ,_sParams are not checked
494:     function clearTokens(bytes calldata _sParams, address _recipient)
             external
             override
             requiresAgentExecutor
             returns (SettlementMultipleParams memory)
         {

//@audit  ,_srcAddress ,_payload are not checked
578:     function lzReceive(uint16, bytes calldata _srcAddress, uint64, bytes calldata _payload) public override {
             address(this).excessivelySafeCall(

//@audit  ,_srcAddress are not checked
587:     function lzReceiveNonBlocking(address _endpoint, bytes calldata _srcAddress, bytes calldata _payload)
             public
             override
             requiresEndpoint(_endpoint, _srcAddress)
         {

//@audit  ,_calldata are not checked
712:     function _execute(uint256 _settlementNonce, bytes memory _calldata) private {
             //Update tx state as executed

//@audit  ,_calldata are not checked
730:     function _execute(bool _hasFallbackToggled, uint32 _settlementNonce, address _refundee, bytes memory _calldata)
             private
         {

//@audit  ,_payload are not checked
765:     function _performCall(address payable _refundee, bytes memory _payload, GasParams calldata _gParams)
             internal
             virtual
         {

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L765-L768)

```solidity
File: src/BranchBridgeAgentExecutor.sol

//@audit  ,_payload are not checked
53:     function executeNoSettlement(address _router, bytes calldata _payload) external payable onlyOwner {
            // Execute Calldata if there is code in the destination router

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgentExecutor.sol#L53-L54)

```solidity
File: src/CoreRootRouter.sol

//@audit  ,_encodedData are not checked
297:     function executeResponse(bytes calldata _encodedData, uint16 _srcChainId)
             external
             payable
             override
             requiresExecutor
         {

//@audit  ,_encodedData are not checked
332:     function execute(bytes calldata _encodedData, uint16) external payable override requiresExecutor {
             // Parse funcId

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L332-L333)

```solidity
File: src/MulticallRootRouter.sol

//@audit  ,encodedData are not checked
137:     function execute(bytes calldata encodedData, uint16) external payable override lock requiresExecutor {
             // Parse funcId

//@audit  ,encodedData are not checked
223:     function executeSigned(bytes calldata encodedData, address userAccount, uint16)
             external
             payable
             override
             lock
             requiresExecutor
         {

//@audit  ,encodedData are not checked
312:     function executeSignedDepositSingle(bytes calldata encodedData, DepositParams calldata, address userAccount, uint16)
             external
             payable
             override
             requiresExecutor
             lock
         {

//@audit  ,encodedData are not checked
401:     function executeSignedDepositMultiple(
             bytes calldata encodedData,
             DepositMultipleParams calldata,
             address userAccount,
             uint16
         ) external payable override requiresExecutor lock {

//@audit  ,data are not checked
581:     function _decode(bytes calldata data) internal pure virtual returns (bytes memory) {
             return data;

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L581-L582)

```solidity
File: src/MulticallRootRouterLibZip.sol

//@audit  ,data are not checked
37:     function _decode(bytes calldata data) internal pure override returns (bytes memory) {
            return data.cdDecompress();

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouterLibZip.sol#L37-L38)

```solidity
File: src/RootBridgeAgent.sol

//@audit  ,_payload are not checked
140:     function getFeeEstimate(
             uint256 _gasLimit,
             uint256 _remoteBranchExecutionGas,
             bytes calldata _payload,
             uint16 _dstChainId
         ) external view returns (uint256 _fee) {

//@audit  ,_params are not checked
160:     function callOut(
             address payable _refundee,
             address _recipient,
             uint16 _dstChainId,
             bytes calldata _params,
             GasParams calldata _gParams
         ) external payable override lock requiresRouter {

//@audit  ,_params are not checked
175:     function callOutAndBridge(
             address payable _refundee,
             address _recipient,
             uint16 _dstChainId,
             bytes calldata _params,
             SettlementInput calldata _sParams,
             GasParams calldata _gParams,
             bool _hasFallbackToggled
         ) external payable override lock requiresRouter {

//@audit  ,_params are not checked
202:     function callOutAndBridgeMultiple(
             address payable _refundee,
             address _recipient,
             uint16 _dstChainId,
             bytes calldata _params,
             SettlementMultipleInput calldata _sParams,
             GasParams calldata _gParams,
             bool _hasFallbackToggled
         ) external payable override lock requiresRouter {

//@audit  ,_params are not checked
233:     function retrySettlement(
             uint32 _settlementNonce,
             address _recipient,
             bytes calldata _params,
             GasParams calldata _gParams,
             bool _hasFallbackToggled
         ) external payable override lock {

//@audit  ,_srcAddress ,_payload are not checked
423:     function lzReceive(uint16 _srcChainId, bytes calldata _srcAddress, uint64, bytes calldata _payload) public {
             (bool success,) = address(this).excessivelySafeCall(

//@audit  ,_srcAddress are not checked
434:     function lzReceiveNonBlocking(
             address _endpoint,
             uint16 _srcChainId,
             bytes calldata _srcAddress,
             bytes calldata _payload
         ) public override requiresEndpoint(_endpoint, _srcChainId, _srcAddress) {

//@audit  ,_calldata are not checked
749:     function _execute(uint256 _depositNonce, bytes memory _calldata, uint16 _srcChainId) private {
             //Update tx state as executed

//@audit  ,_calldata are not checked
768:     function _execute(
             bool _hasFallbackToggled,
             uint32 _depositNonce,
             address _refundee,
             bytes memory _calldata,
             uint16 _srcChainId
         ) private {

//@audit  ,_payload are not checked
808:     function _performCall(
             uint16 _dstChainId,
             address payable _refundee,
             bytes memory _payload,
             GasParams calldata _gParams
         ) internal {

//@audit  ,_params are not checked
856:     function _performRetrySettlementCall(
             bool _hasFallbackToggled,
             address[] memory _hTokens,
             address[] memory _tokens,
             uint256[] memory _amounts,
             uint256[] memory _deposits,
             bytes memory _params,
             uint32 _settlementNonce,
             address payable _refundee,
             address _recipient,
             uint16 _dstChainId,
             GasParams memory _gParams,
             uint256 _value
         ) internal {

//@audit  ,_params are not checked
966:     function _createSettlement(
             uint32 _settlementNonce,
             address payable _refundee,
             address _recipient,
             uint16 _dstChainId,
             bytes memory _params,
             address _globalAddress,
             uint256 _amount,
             uint256 _deposit,
             bool _hasFallbackToggled
         ) internal returns (bytes memory _payload) {

//@audit  ,_params are not checked
1045:     function _createSettlementMultiple(
              uint32 _settlementNonce,
              address payable _refundee,
              address _recipient,
              uint16 _dstChainId,
              address[] memory _globalAddresses,
              uint256[] memory _amounts,
              uint256[] memory _deposits,
              bytes memory _params,
              bool _hasFallbackToggled
          ) internal returns (bytes memory _payload) {

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1045-L1055)

```solidity
File: src/RootBridgeAgentExecutor.sol

//@audit  ,_payload are not checked
50:     function executeSystemRequest(address _router, bytes calldata _payload, uint16 _srcChainId)
            external
            payable
            onlyOwner
        {

//@audit  ,_payload are not checked
66:     function executeNoDeposit(address _router, bytes calldata _payload, uint16 _srcChainId)
            external
            payable
            onlyOwner
        {

//@audit  ,_payload are not checked
115:     function executeWithDepositMultiple(address _router, bytes calldata _payload, uint16 _srcChainId)
             external
             payable
             onlyOwner
         {

//@audit  ,_payload are not checked
150:     function executeSignedNoDeposit(address _account, address _router, bytes calldata _payload, uint16 _srcChainId)
             external
             payable
             onlyOwner
         {

//@audit  ,_dParams are not checked
268:     function _bridgeInMultiple(address _recipient, bytes calldata _dParams, uint16 _srcChainId)
             internal
             returns (DepositMultipleParams memory dParams)
         {

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L268-L271)

```solidity
File: src/interfaces/IBranchBridgeAgent.sol

//@audit  ,_payload are not checked
120:     function getFeeEstimate(uint256 _gasLimit, uint256 _remoteBranchExecutionGas, bytes calldata _payload)
             external
             view
             returns (uint256 _fee);
     
         /*///////////////////////////////////////////////////////////////
                         USER AND BRANCH ROUTER FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice Internal function performs call to Layerzero Enpoint Contract for cross-chain messaging.
          *   @param gasRefundee address to return excess gas deposited in `msg.value` to.
          *   @param params calldata for omnichain execution.
          *   @param gasParams gas parameters for the cross-chain call.
          *   @dev DEPOSIT ID: 0 (System Call / Response)
          *   @dev this flag allows for identifying system emitted request/responses.
          *
          */
         function callOutSystem(address payable gasRefundee, bytes calldata params, GasParams calldata gasParams)
             external
             payable;
     
         /**
          * @notice Function to perform a call to the Root Omnichain Router without token deposit.
          *   @param gasRefundee address to return excess gas deposited in `msg.value` to.
          *   @param params enconded parameters to execute on the root chain router.
          *   @param gasParams gas parameters for the cross-chain call.
          *   @dev DEPOSIT ID: 1 (Call without deposit)
          *
          */
         function callOut(address payable gasRefundee, bytes calldata params, GasParams calldata gasParams)
             external
             payable;
     
         /**
          * @notice Function to perform a call to the Root Omnichain Router while depositing a single asset.
          *   @param gasRefundee address to return excess gas deposited in `msg.value` to.
          *   @param params enconded parameters to execute on the root chain router.
          *   @param depositParams additional token deposit parameters.
          *   @param gasParams gas parameters for the cross-chain call.
          *   @dev DEPOSIT ID: 2 (Call with single deposit)
          *
          */
         function callOutAndBridge(
             address payable gasRefundee,
             bytes calldata params,
             DepositInput memory depositParams,
             GasParams calldata gasParams
         ) external payable;
     
         /**
          * @notice Function to perform a call to the Root Omnichain Router while depositing two or more assets.
          *   @param gasRefundee address to return excess gas deposited in `msg.value` to.
          *   @param params enconded parameters to execute on the root chain router.
          *   @param depositParams additional token deposit parameters.
          *   @param gasParams gas parameters for the cross-chain call.
          *   @dev DEPOSIT ID: 3 (Call with multiple deposit)
          *
          */
         function callOutAndBridgeMultiple(
             address payable gasRefundee,
             bytes calldata params,
             DepositMultipleInput memory depositParams,
             GasParams calldata gasParams
         ) external payable;
     
         /**
          * @notice Perform a call to the Root Omnichain Router without token deposit with msg.sender information.
          *   @param gasRefundee address to return excess gas deposited in `msg.value` to.
          *   @param params enconded parameters to execute on the root chain router.
          *   @param gasParams gas parameters for the cross-chain call.
          *   @dev DEPOSIT ID: 4 (Call without deposit and verified sender)
          *
          */
         function callOutSigned(address payable gasRefundee, bytes calldata params, GasParams calldata gasParams)
             external
             payable;
     
         /**
          * @notice Function to perform a call to the Root Omnichain Router while depositing a single asset msg.sender.
          *   @param gasRefundee address to return excess gas deposited in `msg.value` to.
          *   @param params enconded parameters to execute on the root chain router.
          *   @param depositParams additional token deposit parameters.
          *   @param gasParams gas parameters for the cross-chain call.
          *   @param hasFallbackToggled flag to indicate if the fallback function was toggled.
          *   @dev DEPOSIT ID: 5 (Call with single deposit and verified sender)
          *
          */
         function callOutSignedAndBridge(
             address payable gasRefundee,
             bytes calldata params,
             DepositInput memory depositParams,
             GasParams calldata gasParams,
             bool hasFallbackToggled
         ) external payable;
     
         /**
          * @notice Function to perform a call to the Root Omnichain Router while
          *         depositing two or more assets with msg.sender.
          *   @param gasRefundee address to return excess gas deposited in `msg.value` to.
          *   @param params enconded parameters to execute on the root chain router.
          *   @param depositParams additional token deposit parameters.
          *   @param gasParams gas parameters for the cross-chain call.
          *   @param hasFallbackToggled flag to indicate if the fallback function was toggled.
          *   @dev DEPOSIT ID: 6 (Call with multiple deposit and verified sender)
          *
          */
         function callOutSignedAndBridgeMultiple(
             address payable gasRefundee,
             bytes calldata params,
             DepositMultipleInput memory depositParams,
             GasParams calldata gasParams,
             bool hasFallbackToggled
         ) external payable;
     
         /*///////////////////////////////////////////////////////////////
                         DEPOSIT EXTERNAL FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice Function to perform a call to the Root Omnichain Environment
          *         retrying a failed deposit that hasn't been executed yet.
          *   @param isSigned Flag to indicate if the deposit was signed.
          *   @param depositNonce Identifier for user deposit.
          *   @param params parameters to execute on the root chain router.
          *   @param gasParams gas parameters for the cross-chain call.
          *   @param hasFallbackToggled flag to indicate if the fallback function was toggled.
          */
         function retryDeposit(
             bool isSigned,
             uint32 depositNonce,
             bytes calldata params,
             GasParams calldata gasParams,
             bool hasFallbackToggled
         ) external payable;
     
         /**
          * @notice External function to request tokens back to branch chain after failing omnichain environment interaction.
          *    @param depositNonce Identifier for user deposit to retrieve.
          *    @param gasParams gas parameters for the cross-chain call.
          *    @dev DEPOSIT ID: 8
          *
          */
         function retrieveDeposit(uint32 depositNonce, GasParams calldata gasParams) external payable;
     
         /**
          * @notice External function to retry a failed Deposit entry on this branch chain.
          *    @param depositNonce Identifier for user deposit.
          *
          */
         function redeemDeposit(uint32 depositNonce) external;
     
         /*///////////////////////////////////////////////////////////////
                         SETTLEMENT EXTERNAL FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice External function to retry a failed Settlement entry on the root chain.
          *   @param settlementNonce Identifier for user settlement.
          *   @param params parameters to execute on the root chain router.
          *   @param gasParams gas parameters for the cross-chain call to root chain and for the settlement to branch.
          *   @param hasFallbackToggled flag to indicate if the fallback function should be toggled.
          *   @dev DEPOSIT ID: 7
          *
          */
         function retrySettlement(
             uint32 settlementNonce,
             bytes calldata params,
             GasParams[2] calldata gasParams,
             bool hasFallbackToggled
         ) external payable;
     
         /*///////////////////////////////////////////////////////////////
                         TOKEN MANAGEMENT EXTERNAL FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice Function to request balance clearance from a Port to a given user.
          *     @param recipient token receiver.
          *     @param hToken  local hToken addresse to clear balance for.
          *     @param token  native / underlying token addresse to clear balance for.
          *     @param amount amounts of hToken to clear balance for.
          *     @param deposit amount of native / underlying tokens to clear balance for.
          *
          */
         function clearToken(address recipient, address hToken, address token, uint256 amount, uint256 deposit) external;
     
         /**
          * @notice Function to request balance clearance from a Port to a given address.
          *     @param sParams encode packed multiple settlement info.
          *
          */
         function clearTokens(bytes calldata sParams, address recipient)
             external
             returns (SettlementMultipleParams memory);
     
         /*///////////////////////////////////////////////////////////////
                                 LAYER ZERO FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice External function to receive cross-chain messages from LayerZero Endpoint Contract without blocking.
          *   @param _endpoint address of the LayerZero Endpoint Contract.
          *   @param _srcAddress address path of the recipient + sender.
          *   @param _payload Calldata for function call.
          */
         function lzReceiveNonBlocking(address _endpoint, bytes calldata _srcAddress, bytes calldata _payload) external;
     
         /*///////////////////////////////////////////////////////////////
                             EVENTS
         //////////////////////////////////////////////////////////////*/
     
         event LogExecute(uint256 indexed nonce);
         event LogFallback(uint256 indexed nonce);
     
         /*///////////////////////////////////////////////////////////////
                                     ERRORS
         //////////////////////////////////////////////////////////////*/
     
         error UnknownFlag();
         error ExecutionFailure();
     
         error LayerZeroUnauthorizedCaller();
         error LayerZeroUnauthorizedEndpoint();
     
         error AlreadyExecutedTransaction();
     
         error InvalidInput();
         error InsufficientGas();
     
         error NotDepositOwner();
         error DepositRetryUnavailableUseCallout();
         error DepositRedeemUnavailable();
     
         error UnrecognizedRouter();
         error UnrecognizedBridgeAgentExecutor();
     }
     

//@audit  ,params are not checked
138:     function callOutSystem(address payable gasRefundee, bytes calldata params, GasParams calldata gasParams)
             external
             payable;
     
         /**
          * @notice Function to perform a call to the Root Omnichain Router without token deposit.
          *   @param gasRefundee address to return excess gas deposited in `msg.value` to.
          *   @param params enconded parameters to execute on the root chain router.
          *   @param gasParams gas parameters for the cross-chain call.
          *   @dev DEPOSIT ID: 1 (Call without deposit)
          *
          */
         function callOut(address payable gasRefundee, bytes calldata params, GasParams calldata gasParams)
             external
             payable;
     
         /**
          * @notice Function to perform a call to the Root Omnichain Router while depositing a single asset.
          *   @param gasRefundee address to return excess gas deposited in `msg.value` to.
          *   @param params enconded parameters to execute on the root chain router.
          *   @param depositParams additional token deposit parameters.
          *   @param gasParams gas parameters for the cross-chain call.
          *   @dev DEPOSIT ID: 2 (Call with single deposit)
          *
          */
         function callOutAndBridge(
             address payable gasRefundee,
             bytes calldata params,
             DepositInput memory depositParams,
             GasParams calldata gasParams
         ) external payable;
     
         /**
          * @notice Function to perform a call to the Root Omnichain Router while depositing two or more assets.
          *   @param gasRefundee address to return excess gas deposited in `msg.value` to.
          *   @param params enconded parameters to execute on the root chain router.
          *   @param depositParams additional token deposit parameters.
          *   @param gasParams gas parameters for the cross-chain call.
          *   @dev DEPOSIT ID: 3 (Call with multiple deposit)
          *
          */
         function callOutAndBridgeMultiple(
             address payable gasRefundee,
             bytes calldata params,
             DepositMultipleInput memory depositParams,
             GasParams calldata gasParams
         ) external payable;
     
         /**
          * @notice Perform a call to the Root Omnichain Router without token deposit with msg.sender information.
          *   @param gasRefundee address to return excess gas deposited in `msg.value` to.
          *   @param params enconded parameters to execute on the root chain router.
          *   @param gasParams gas parameters for the cross-chain call.
          *   @dev DEPOSIT ID: 4 (Call without deposit and verified sender)
          *
          */
         function callOutSigned(address payable gasRefundee, bytes calldata params, GasParams calldata gasParams)
             external
             payable;
     
         /**
          * @notice Function to perform a call to the Root Omnichain Router while depositing a single asset msg.sender.
          *   @param gasRefundee address to return excess gas deposited in `msg.value` to.
          *   @param params enconded parameters to execute on the root chain router.
          *   @param depositParams additional token deposit parameters.
          *   @param gasParams gas parameters for the cross-chain call.
          *   @param hasFallbackToggled flag to indicate if the fallback function was toggled.
          *   @dev DEPOSIT ID: 5 (Call with single deposit and verified sender)
          *
          */
         function callOutSignedAndBridge(
             address payable gasRefundee,
             bytes calldata params,
             DepositInput memory depositParams,
             GasParams calldata gasParams,
             bool hasFallbackToggled
         ) external payable;
     
         /**
          * @notice Function to perform a call to the Root Omnichain Router while
          *         depositing two or more assets with msg.sender.
          *   @param gasRefundee address to return excess gas deposited in `msg.value` to.
          *   @param params enconded parameters to execute on the root chain router.
          *   @param depositParams additional token deposit parameters.
          *   @param gasParams gas parameters for the cross-chain call.
          *   @param hasFallbackToggled flag to indicate if the fallback function was toggled.
          *   @dev DEPOSIT ID: 6 (Call with multiple deposit and verified sender)
          *
          */
         function callOutSignedAndBridgeMultiple(
             address payable gasRefundee,
             bytes calldata params,
             DepositMultipleInput memory depositParams,
             GasParams calldata gasParams,
             bool hasFallbackToggled
         ) external payable;
     
         /*///////////////////////////////////////////////////////////////
                         DEPOSIT EXTERNAL FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice Function to perform a call to the Root Omnichain Environment
          *         retrying a failed deposit that hasn't been executed yet.
          *   @param isSigned Flag to indicate if the deposit was signed.
          *   @param depositNonce Identifier for user deposit.
          *   @param params parameters to execute on the root chain router.
          *   @param gasParams gas parameters for the cross-chain call.
          *   @param hasFallbackToggled flag to indicate if the fallback function was toggled.
          */
         function retryDeposit(
             bool isSigned,
             uint32 depositNonce,
             bytes calldata params,
             GasParams calldata gasParams,
             bool hasFallbackToggled
         ) external payable;
     
         /**
          * @notice External function to request tokens back to branch chain after failing omnichain environment interaction.
          *    @param depositNonce Identifier for user deposit to retrieve.
          *    @param gasParams gas parameters for the cross-chain call.
          *    @dev DEPOSIT ID: 8
          *
          */
         function retrieveDeposit(uint32 depositNonce, GasParams calldata gasParams) external payable;
     
         /**
          * @notice External function to retry a failed Deposit entry on this branch chain.
          *    @param depositNonce Identifier for user deposit.
          *
          */
         function redeemDeposit(uint32 depositNonce) external;
     
         /*///////////////////////////////////////////////////////////////
                         SETTLEMENT EXTERNAL FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice External function to retry a failed Settlement entry on the root chain.
          *   @param settlementNonce Identifier for user settlement.
          *   @param params parameters to execute on the root chain router.
          *   @param gasParams gas parameters for the cross-chain call to root chain and for the settlement to branch.
          *   @param hasFallbackToggled flag to indicate if the fallback function should be toggled.
          *   @dev DEPOSIT ID: 7
          *
          */
         function retrySettlement(
             uint32 settlementNonce,
             bytes calldata params,
             GasParams[2] calldata gasParams,
             bool hasFallbackToggled
         ) external payable;
     
         /*///////////////////////////////////////////////////////////////
                         TOKEN MANAGEMENT EXTERNAL FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice Function to request balance clearance from a Port to a given user.
          *     @param recipient token receiver.
          *     @param hToken  local hToken addresse to clear balance for.
          *     @param token  native / underlying token addresse to clear balance for.
          *     @param amount amounts of hToken to clear balance for.
          *     @param deposit amount of native / underlying tokens to clear balance for.
          *
          */
         function clearToken(address recipient, address hToken, address token, uint256 amount, uint256 deposit) external;
     
         /**
          * @notice Function to request balance clearance from a Port to a given address.
          *     @param sParams encode packed multiple settlement info.
          *
          */
         function clearTokens(bytes calldata sParams, address recipient)
             external
             returns (SettlementMultipleParams memory);
     
         /*///////////////////////////////////////////////////////////////
                                 LAYER ZERO FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice External function to receive cross-chain messages from LayerZero Endpoint Contract without blocking.
          *   @param _endpoint address of the LayerZero Endpoint Contract.
          *   @param _srcAddress address path of the recipient + sender.
          *   @param _payload Calldata for function call.
          */
         function lzReceiveNonBlocking(address _endpoint, bytes calldata _srcAddress, bytes calldata _payload) external;
     
         /*///////////////////////////////////////////////////////////////
                             EVENTS
         //////////////////////////////////////////////////////////////*/
     
         event LogExecute(uint256 indexed nonce);
         event LogFallback(uint256 indexed nonce);
     
         /*///////////////////////////////////////////////////////////////
                                     ERRORS
         //////////////////////////////////////////////////////////////*/
     
         error UnknownFlag();
         error ExecutionFailure();
     
         error LayerZeroUnauthorizedCaller();
         error LayerZeroUnauthorizedEndpoint();
     
         error AlreadyExecutedTransaction();
     
         error InvalidInput();
         error InsufficientGas();
     
         error NotDepositOwner();
         error DepositRetryUnavailableUseCallout();
         error DepositRedeemUnavailable();
     
         error UnrecognizedRouter();
         error UnrecognizedBridgeAgentExecutor();
     }
     

//@audit  ,params are not checked
150:     function callOut(address payable gasRefundee, bytes calldata params, GasParams calldata gasParams)
             external
             payable;
     
         /**
          * @notice Function to perform a call to the Root Omnichain Router while depositing a single asset.
          *   @param gasRefundee address to return excess gas deposited in `msg.value` to.
          *   @param params enconded parameters to execute on the root chain router.
          *   @param depositParams additional token deposit parameters.
          *   @param gasParams gas parameters for the cross-chain call.
          *   @dev DEPOSIT ID: 2 (Call with single deposit)
          *
          */
         function callOutAndBridge(
             address payable gasRefundee,
             bytes calldata params,
             DepositInput memory depositParams,
             GasParams calldata gasParams
         ) external payable;
     
         /**
          * @notice Function to perform a call to the Root Omnichain Router while depositing two or more assets.
          *   @param gasRefundee address to return excess gas deposited in `msg.value` to.
          *   @param params enconded parameters to execute on the root chain router.
          *   @param depositParams additional token deposit parameters.
          *   @param gasParams gas parameters for the cross-chain call.
          *   @dev DEPOSIT ID: 3 (Call with multiple deposit)
          *
          */
         function callOutAndBridgeMultiple(
             address payable gasRefundee,
             bytes calldata params,
             DepositMultipleInput memory depositParams,
             GasParams calldata gasParams
         ) external payable;
     
         /**
          * @notice Perform a call to the Root Omnichain Router without token deposit with msg.sender information.
          *   @param gasRefundee address to return excess gas deposited in `msg.value` to.
          *   @param params enconded parameters to execute on the root chain router.
          *   @param gasParams gas parameters for the cross-chain call.
          *   @dev DEPOSIT ID: 4 (Call without deposit and verified sender)
          *
          */
         function callOutSigned(address payable gasRefundee, bytes calldata params, GasParams calldata gasParams)
             external
             payable;
     
         /**
          * @notice Function to perform a call to the Root Omnichain Router while depositing a single asset msg.sender.
          *   @param gasRefundee address to return excess gas deposited in `msg.value` to.
          *   @param params enconded parameters to execute on the root chain router.
          *   @param depositParams additional token deposit parameters.
          *   @param gasParams gas parameters for the cross-chain call.
          *   @param hasFallbackToggled flag to indicate if the fallback function was toggled.
          *   @dev DEPOSIT ID: 5 (Call with single deposit and verified sender)
          *
          */
         function callOutSignedAndBridge(
             address payable gasRefundee,
             bytes calldata params,
             DepositInput memory depositParams,
             GasParams calldata gasParams,
             bool hasFallbackToggled
         ) external payable;
     
         /**
          * @notice Function to perform a call to the Root Omnichain Router while
          *         depositing two or more assets with msg.sender.
          *   @param gasRefundee address to return excess gas deposited in `msg.value` to.
          *   @param params enconded parameters to execute on the root chain router.
          *   @param depositParams additional token deposit parameters.
          *   @param gasParams gas parameters for the cross-chain call.
          *   @param hasFallbackToggled flag to indicate if the fallback function was toggled.
          *   @dev DEPOSIT ID: 6 (Call with multiple deposit and verified sender)
          *
          */
         function callOutSignedAndBridgeMultiple(
             address payable gasRefundee,
             bytes calldata params,
             DepositMultipleInput memory depositParams,
             GasParams calldata gasParams,
             bool hasFallbackToggled
         ) external payable;
     
         /*///////////////////////////////////////////////////////////////
                         DEPOSIT EXTERNAL FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice Function to perform a call to the Root Omnichain Environment
          *         retrying a failed deposit that hasn't been executed yet.
          *   @param isSigned Flag to indicate if the deposit was signed.
          *   @param depositNonce Identifier for user deposit.
          *   @param params parameters to execute on the root chain router.
          *   @param gasParams gas parameters for the cross-chain call.
          *   @param hasFallbackToggled flag to indicate if the fallback function was toggled.
          */
         function retryDeposit(
             bool isSigned,
             uint32 depositNonce,
             bytes calldata params,
             GasParams calldata gasParams,
             bool hasFallbackToggled
         ) external payable;
     
         /**
          * @notice External function to request tokens back to branch chain after failing omnichain environment interaction.
          *    @param depositNonce Identifier for user deposit to retrieve.
          *    @param gasParams gas parameters for the cross-chain call.
          *    @dev DEPOSIT ID: 8
          *
          */
         function retrieveDeposit(uint32 depositNonce, GasParams calldata gasParams) external payable;
     
         /**
          * @notice External function to retry a failed Deposit entry on this branch chain.
          *    @param depositNonce Identifier for user deposit.
          *
          */
         function redeemDeposit(uint32 depositNonce) external;
     
         /*///////////////////////////////////////////////////////////////
                         SETTLEMENT EXTERNAL FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice External function to retry a failed Settlement entry on the root chain.
          *   @param settlementNonce Identifier for user settlement.
          *   @param params parameters to execute on the root chain router.
          *   @param gasParams gas parameters for the cross-chain call to root chain and for the settlement to branch.
          *   @param hasFallbackToggled flag to indicate if the fallback function should be toggled.
          *   @dev DEPOSIT ID: 7
          *
          */
         function retrySettlement(
             uint32 settlementNonce,
             bytes calldata params,
             GasParams[2] calldata gasParams,
             bool hasFallbackToggled
         ) external payable;
     
         /*///////////////////////////////////////////////////////////////
                         TOKEN MANAGEMENT EXTERNAL FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice Function to request balance clearance from a Port to a given user.
          *     @param recipient token receiver.
          *     @param hToken  local hToken addresse to clear balance for.
          *     @param token  native / underlying token addresse to clear balance for.
          *     @param amount amounts of hToken to clear balance for.
          *     @param deposit amount of native / underlying tokens to clear balance for.
          *
          */
         function clearToken(address recipient, address hToken, address token, uint256 amount, uint256 deposit) external;
     
         /**
          * @notice Function to request balance clearance from a Port to a given address.
          *     @param sParams encode packed multiple settlement info.
          *
          */
         function clearTokens(bytes calldata sParams, address recipient)
             external
             returns (SettlementMultipleParams memory);
     
         /*///////////////////////////////////////////////////////////////
                                 LAYER ZERO FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice External function to receive cross-chain messages from LayerZero Endpoint Contract without blocking.
          *   @param _endpoint address of the LayerZero Endpoint Contract.
          *   @param _srcAddress address path of the recipient + sender.
          *   @param _payload Calldata for function call.
          */
         function lzReceiveNonBlocking(address _endpoint, bytes calldata _srcAddress, bytes calldata _payload) external;
     
         /*///////////////////////////////////////////////////////////////
                             EVENTS
         //////////////////////////////////////////////////////////////*/
     
         event LogExecute(uint256 indexed nonce);
         event LogFallback(uint256 indexed nonce);
     
         /*///////////////////////////////////////////////////////////////
                                     ERRORS
         //////////////////////////////////////////////////////////////*/
     
         error UnknownFlag();
         error ExecutionFailure();
     
         error LayerZeroUnauthorizedCaller();
         error LayerZeroUnauthorizedEndpoint();
     
         error AlreadyExecutedTransaction();
     
         error InvalidInput();
         error InsufficientGas();
     
         error NotDepositOwner();
         error DepositRetryUnavailableUseCallout();
         error DepositRedeemUnavailable();
     
         error UnrecognizedRouter();
         error UnrecognizedBridgeAgentExecutor();
     }
     

//@audit  ,params are not checked
163:     function callOutAndBridge(
             address payable gasRefundee,
             bytes calldata params,
             DepositInput memory depositParams,
             GasParams calldata gasParams
         ) external payable;
     
         /**
          * @notice Function to perform a call to the Root Omnichain Router while depositing two or more assets.
          *   @param gasRefundee address to return excess gas deposited in `msg.value` to.
          *   @param params enconded parameters to execute on the root chain router.
          *   @param depositParams additional token deposit parameters.
          *   @param gasParams gas parameters for the cross-chain call.
          *   @dev DEPOSIT ID: 3 (Call with multiple deposit)
          *
          */
         function callOutAndBridgeMultiple(
             address payable gasRefundee,
             bytes calldata params,
             DepositMultipleInput memory depositParams,
             GasParams calldata gasParams
         ) external payable;
     
         /**
          * @notice Perform a call to the Root Omnichain Router without token deposit with msg.sender information.
          *   @param gasRefundee address to return excess gas deposited in `msg.value` to.
          *   @param params enconded parameters to execute on the root chain router.
          *   @param gasParams gas parameters for the cross-chain call.
          *   @dev DEPOSIT ID: 4 (Call without deposit and verified sender)
          *
          */
         function callOutSigned(address payable gasRefundee, bytes calldata params, GasParams calldata gasParams)
             external
             payable;
     
         /**
          * @notice Function to perform a call to the Root Omnichain Router while depositing a single asset msg.sender.
          *   @param gasRefundee address to return excess gas deposited in `msg.value` to.
          *   @param params enconded parameters to execute on the root chain router.
          *   @param depositParams additional token deposit parameters.
          *   @param gasParams gas parameters for the cross-chain call.
          *   @param hasFallbackToggled flag to indicate if the fallback function was toggled.
          *   @dev DEPOSIT ID: 5 (Call with single deposit and verified sender)
          *
          */
         function callOutSignedAndBridge(
             address payable gasRefundee,
             bytes calldata params,
             DepositInput memory depositParams,
             GasParams calldata gasParams,
             bool hasFallbackToggled
         ) external payable;
     
         /**
          * @notice Function to perform a call to the Root Omnichain Router while
          *         depositing two or more assets with msg.sender.
          *   @param gasRefundee address to return excess gas deposited in `msg.value` to.
          *   @param params enconded parameters to execute on the root chain router.
          *   @param depositParams additional token deposit parameters.
          *   @param gasParams gas parameters for the cross-chain call.
          *   @param hasFallbackToggled flag to indicate if the fallback function was toggled.
          *   @dev DEPOSIT ID: 6 (Call with multiple deposit and verified sender)
          *
          */
         function callOutSignedAndBridgeMultiple(
             address payable gasRefundee,
             bytes calldata params,
             DepositMultipleInput memory depositParams,
             GasParams calldata gasParams,
             bool hasFallbackToggled
         ) external payable;
     
         /*///////////////////////////////////////////////////////////////
                         DEPOSIT EXTERNAL FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice Function to perform a call to the Root Omnichain Environment
          *         retrying a failed deposit that hasn't been executed yet.
          *   @param isSigned Flag to indicate if the deposit was signed.
          *   @param depositNonce Identifier for user deposit.
          *   @param params parameters to execute on the root chain router.
          *   @param gasParams gas parameters for the cross-chain call.
          *   @param hasFallbackToggled flag to indicate if the fallback function was toggled.
          */
         function retryDeposit(
             bool isSigned,
             uint32 depositNonce,
             bytes calldata params,
             GasParams calldata gasParams,
             bool hasFallbackToggled
         ) external payable;
     
         /**
          * @notice External function to request tokens back to branch chain after failing omnichain environment interaction.
          *    @param depositNonce Identifier for user deposit to retrieve.
          *    @param gasParams gas parameters for the cross-chain call.
          *    @dev DEPOSIT ID: 8
          *
          */
         function retrieveDeposit(uint32 depositNonce, GasParams calldata gasParams) external payable;
     
         /**
          * @notice External function to retry a failed Deposit entry on this branch chain.
          *    @param depositNonce Identifier for user deposit.
          *
          */
         function redeemDeposit(uint32 depositNonce) external;
     
         /*///////////////////////////////////////////////////////////////
                         SETTLEMENT EXTERNAL FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice External function to retry a failed Settlement entry on the root chain.
          *   @param settlementNonce Identifier for user settlement.
          *   @param params parameters to execute on the root chain router.
          *   @param gasParams gas parameters for the cross-chain call to root chain and for the settlement to branch.
          *   @param hasFallbackToggled flag to indicate if the fallback function should be toggled.
          *   @dev DEPOSIT ID: 7
          *
          */
         function retrySettlement(
             uint32 settlementNonce,
             bytes calldata params,
             GasParams[2] calldata gasParams,
             bool hasFallbackToggled
         ) external payable;
     
         /*///////////////////////////////////////////////////////////////
                         TOKEN MANAGEMENT EXTERNAL FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice Function to request balance clearance from a Port to a given user.
          *     @param recipient token receiver.
          *     @param hToken  local hToken addresse to clear balance for.
          *     @param token  native / underlying token addresse to clear balance for.
          *     @param amount amounts of hToken to clear balance for.
          *     @param deposit amount of native / underlying tokens to clear balance for.
          *
          */
         function clearToken(address recipient, address hToken, address token, uint256 amount, uint256 deposit) external;
     
         /**
          * @notice Function to request balance clearance from a Port to a given address.
          *     @param sParams encode packed multiple settlement info.
          *
          */
         function clearTokens(bytes calldata sParams, address recipient)
             external
             returns (SettlementMultipleParams memory);
     
         /*///////////////////////////////////////////////////////////////
                                 LAYER ZERO FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice External function to receive cross-chain messages from LayerZero Endpoint Contract without blocking.
          *   @param _endpoint address of the LayerZero Endpoint Contract.
          *   @param _srcAddress address path of the recipient + sender.
          *   @param _payload Calldata for function call.
          */
         function lzReceiveNonBlocking(address _endpoint, bytes calldata _srcAddress, bytes calldata _payload) external;
     
         /*///////////////////////////////////////////////////////////////
                             EVENTS
         //////////////////////////////////////////////////////////////*/
     
         event LogExecute(uint256 indexed nonce);
         event LogFallback(uint256 indexed nonce);
     
         /*///////////////////////////////////////////////////////////////
                                     ERRORS
         //////////////////////////////////////////////////////////////*/
     
         error UnknownFlag();
         error ExecutionFailure();
     
         error LayerZeroUnauthorizedCaller();
         error LayerZeroUnauthorizedEndpoint();
     
         error AlreadyExecutedTransaction();
     
         error InvalidInput();
         error InsufficientGas();
     
         error NotDepositOwner();
         error DepositRetryUnavailableUseCallout();
         error DepositRedeemUnavailable();
     
         error UnrecognizedRouter();
         error UnrecognizedBridgeAgentExecutor();
     }
     

//@audit  ,params are not checked
179:     function callOutAndBridgeMultiple(
             address payable gasRefundee,
             bytes calldata params,
             DepositMultipleInput memory depositParams,
             GasParams calldata gasParams
         ) external payable;
     
         /**
          * @notice Perform a call to the Root Omnichain Router without token deposit with msg.sender information.
          *   @param gasRefundee address to return excess gas deposited in `msg.value` to.
          *   @param params enconded parameters to execute on the root chain router.
          *   @param gasParams gas parameters for the cross-chain call.
          *   @dev DEPOSIT ID: 4 (Call without deposit and verified sender)
          *
          */
         function callOutSigned(address payable gasRefundee, bytes calldata params, GasParams calldata gasParams)
             external
             payable;
     
         /**
          * @notice Function to perform a call to the Root Omnichain Router while depositing a single asset msg.sender.
          *   @param gasRefundee address to return excess gas deposited in `msg.value` to.
          *   @param params enconded parameters to execute on the root chain router.
          *   @param depositParams additional token deposit parameters.
          *   @param gasParams gas parameters for the cross-chain call.
          *   @param hasFallbackToggled flag to indicate if the fallback function was toggled.
          *   @dev DEPOSIT ID: 5 (Call with single deposit and verified sender)
          *
          */
         function callOutSignedAndBridge(
             address payable gasRefundee,
             bytes calldata params,
             DepositInput memory depositParams,
             GasParams calldata gasParams,
             bool hasFallbackToggled
         ) external payable;
     
         /**
          * @notice Function to perform a call to the Root Omnichain Router while
          *         depositing two or more assets with msg.sender.
          *   @param gasRefundee address to return excess gas deposited in `msg.value` to.
          *   @param params enconded parameters to execute on the root chain router.
          *   @param depositParams additional token deposit parameters.
          *   @param gasParams gas parameters for the cross-chain call.
          *   @param hasFallbackToggled flag to indicate if the fallback function was toggled.
          *   @dev DEPOSIT ID: 6 (Call with multiple deposit and verified sender)
          *
          */
         function callOutSignedAndBridgeMultiple(
             address payable gasRefundee,
             bytes calldata params,
             DepositMultipleInput memory depositParams,
             GasParams calldata gasParams,
             bool hasFallbackToggled
         ) external payable;
     
         /*///////////////////////////////////////////////////////////////
                         DEPOSIT EXTERNAL FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice Function to perform a call to the Root Omnichain Environment
          *         retrying a failed deposit that hasn't been executed yet.
          *   @param isSigned Flag to indicate if the deposit was signed.
          *   @param depositNonce Identifier for user deposit.
          *   @param params parameters to execute on the root chain router.
          *   @param gasParams gas parameters for the cross-chain call.
          *   @param hasFallbackToggled flag to indicate if the fallback function was toggled.
          */
         function retryDeposit(
             bool isSigned,
             uint32 depositNonce,
             bytes calldata params,
             GasParams calldata gasParams,
             bool hasFallbackToggled
         ) external payable;
     
         /**
          * @notice External function to request tokens back to branch chain after failing omnichain environment interaction.
          *    @param depositNonce Identifier for user deposit to retrieve.
          *    @param gasParams gas parameters for the cross-chain call.
          *    @dev DEPOSIT ID: 8
          *
          */
         function retrieveDeposit(uint32 depositNonce, GasParams calldata gasParams) external payable;
     
         /**
          * @notice External function to retry a failed Deposit entry on this branch chain.
          *    @param depositNonce Identifier for user deposit.
          *
          */
         function redeemDeposit(uint32 depositNonce) external;
     
         /*///////////////////////////////////////////////////////////////
                         SETTLEMENT EXTERNAL FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice External function to retry a failed Settlement entry on the root chain.
          *   @param settlementNonce Identifier for user settlement.
          *   @param params parameters to execute on the root chain router.
          *   @param gasParams gas parameters for the cross-chain call to root chain and for the settlement to branch.
          *   @param hasFallbackToggled flag to indicate if the fallback function should be toggled.
          *   @dev DEPOSIT ID: 7
          *
          */
         function retrySettlement(
             uint32 settlementNonce,
             bytes calldata params,
             GasParams[2] calldata gasParams,
             bool hasFallbackToggled
         ) external payable;
     
         /*///////////////////////////////////////////////////////////////
                         TOKEN MANAGEMENT EXTERNAL FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice Function to request balance clearance from a Port to a given user.
          *     @param recipient token receiver.
          *     @param hToken  local hToken addresse to clear balance for.
          *     @param token  native / underlying token addresse to clear balance for.
          *     @param amount amounts of hToken to clear balance for.
          *     @param deposit amount of native / underlying tokens to clear balance for.
          *
          */
         function clearToken(address recipient, address hToken, address token, uint256 amount, uint256 deposit) external;
     
         /**
          * @notice Function to request balance clearance from a Port to a given address.
          *     @param sParams encode packed multiple settlement info.
          *
          */
         function clearTokens(bytes calldata sParams, address recipient)
             external
             returns (SettlementMultipleParams memory);
     
         /*///////////////////////////////////////////////////////////////
                                 LAYER ZERO FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice External function to receive cross-chain messages from LayerZero Endpoint Contract without blocking.
          *   @param _endpoint address of the LayerZero Endpoint Contract.
          *   @param _srcAddress address path of the recipient + sender.
          *   @param _payload Calldata for function call.
          */
         function lzReceiveNonBlocking(address _endpoint, bytes calldata _srcAddress, bytes calldata _payload) external;
     
         /*///////////////////////////////////////////////////////////////
                             EVENTS
         //////////////////////////////////////////////////////////////*/
     
         event LogExecute(uint256 indexed nonce);
         event LogFallback(uint256 indexed nonce);
     
         /*///////////////////////////////////////////////////////////////
                                     ERRORS
         //////////////////////////////////////////////////////////////*/
     
         error UnknownFlag();
         error ExecutionFailure();
     
         error LayerZeroUnauthorizedCaller();
         error LayerZeroUnauthorizedEndpoint();
     
         error AlreadyExecutedTransaction();
     
         error InvalidInput();
         error InsufficientGas();
     
         error NotDepositOwner();
         error DepositRetryUnavailableUseCallout();
         error DepositRedeemUnavailable();
     
         error UnrecognizedRouter();
         error UnrecognizedBridgeAgentExecutor();
     }
     

//@audit  ,params are not checked
194:     function callOutSigned(address payable gasRefundee, bytes calldata params, GasParams calldata gasParams)
             external
             payable;
     
         /**
          * @notice Function to perform a call to the Root Omnichain Router while depositing a single asset msg.sender.
          *   @param gasRefundee address to return excess gas deposited in `msg.value` to.
          *   @param params enconded parameters to execute on the root chain router.
          *   @param depositParams additional token deposit parameters.
          *   @param gasParams gas parameters for the cross-chain call.
          *   @param hasFallbackToggled flag to indicate if the fallback function was toggled.
          *   @dev DEPOSIT ID: 5 (Call with single deposit and verified sender)
          *
          */
         function callOutSignedAndBridge(
             address payable gasRefundee,
             bytes calldata params,
             DepositInput memory depositParams,
             GasParams calldata gasParams,
             bool hasFallbackToggled
         ) external payable;
     
         /**
          * @notice Function to perform a call to the Root Omnichain Router while
          *         depositing two or more assets with msg.sender.
          *   @param gasRefundee address to return excess gas deposited in `msg.value` to.
          *   @param params enconded parameters to execute on the root chain router.
          *   @param depositParams additional token deposit parameters.
          *   @param gasParams gas parameters for the cross-chain call.
          *   @param hasFallbackToggled flag to indicate if the fallback function was toggled.
          *   @dev DEPOSIT ID: 6 (Call with multiple deposit and verified sender)
          *
          */
         function callOutSignedAndBridgeMultiple(
             address payable gasRefundee,
             bytes calldata params,
             DepositMultipleInput memory depositParams,
             GasParams calldata gasParams,
             bool hasFallbackToggled
         ) external payable;
     
         /*///////////////////////////////////////////////////////////////
                         DEPOSIT EXTERNAL FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice Function to perform a call to the Root Omnichain Environment
          *         retrying a failed deposit that hasn't been executed yet.
          *   @param isSigned Flag to indicate if the deposit was signed.
          *   @param depositNonce Identifier for user deposit.
          *   @param params parameters to execute on the root chain router.
          *   @param gasParams gas parameters for the cross-chain call.
          *   @param hasFallbackToggled flag to indicate if the fallback function was toggled.
          */
         function retryDeposit(
             bool isSigned,
             uint32 depositNonce,
             bytes calldata params,
             GasParams calldata gasParams,
             bool hasFallbackToggled
         ) external payable;
     
         /**
          * @notice External function to request tokens back to branch chain after failing omnichain environment interaction.
          *    @param depositNonce Identifier for user deposit to retrieve.
          *    @param gasParams gas parameters for the cross-chain call.
          *    @dev DEPOSIT ID: 8
          *
          */
         function retrieveDeposit(uint32 depositNonce, GasParams calldata gasParams) external payable;
     
         /**
          * @notice External function to retry a failed Deposit entry on this branch chain.
          *    @param depositNonce Identifier for user deposit.
          *
          */
         function redeemDeposit(uint32 depositNonce) external;
     
         /*///////////////////////////////////////////////////////////////
                         SETTLEMENT EXTERNAL FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice External function to retry a failed Settlement entry on the root chain.
          *   @param settlementNonce Identifier for user settlement.
          *   @param params parameters to execute on the root chain router.
          *   @param gasParams gas parameters for the cross-chain call to root chain and for the settlement to branch.
          *   @param hasFallbackToggled flag to indicate if the fallback function should be toggled.
          *   @dev DEPOSIT ID: 7
          *
          */
         function retrySettlement(
             uint32 settlementNonce,
             bytes calldata params,
             GasParams[2] calldata gasParams,
             bool hasFallbackToggled
         ) external payable;
     
         /*///////////////////////////////////////////////////////////////
                         TOKEN MANAGEMENT EXTERNAL FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice Function to request balance clearance from a Port to a given user.
          *     @param recipient token receiver.
          *     @param hToken  local hToken addresse to clear balance for.
          *     @param token  native / underlying token addresse to clear balance for.
          *     @param amount amounts of hToken to clear balance for.
          *     @param deposit amount of native / underlying tokens to clear balance for.
          *
          */
         function clearToken(address recipient, address hToken, address token, uint256 amount, uint256 deposit) external;
     
         /**
          * @notice Function to request balance clearance from a Port to a given address.
          *     @param sParams encode packed multiple settlement info.
          *
          */
         function clearTokens(bytes calldata sParams, address recipient)
             external
             returns (SettlementMultipleParams memory);
     
         /*///////////////////////////////////////////////////////////////
                                 LAYER ZERO FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice External function to receive cross-chain messages from LayerZero Endpoint Contract without blocking.
          *   @param _endpoint address of the LayerZero Endpoint Contract.
          *   @param _srcAddress address path of the recipient + sender.
          *   @param _payload Calldata for function call.
          */
         function lzReceiveNonBlocking(address _endpoint, bytes calldata _srcAddress, bytes calldata _payload) external;
     
         /*///////////////////////////////////////////////////////////////
                             EVENTS
         //////////////////////////////////////////////////////////////*/
     
         event LogExecute(uint256 indexed nonce);
         event LogFallback(uint256 indexed nonce);
     
         /*///////////////////////////////////////////////////////////////
                                     ERRORS
         //////////////////////////////////////////////////////////////*/
     
         error UnknownFlag();
         error ExecutionFailure();
     
         error LayerZeroUnauthorizedCaller();
         error LayerZeroUnauthorizedEndpoint();
     
         error AlreadyExecutedTransaction();
     
         error InvalidInput();
         error InsufficientGas();
     
         error NotDepositOwner();
         error DepositRetryUnavailableUseCallout();
         error DepositRedeemUnavailable();
     
         error UnrecognizedRouter();
         error UnrecognizedBridgeAgentExecutor();
     }
     

//@audit  ,params are not checked
208:     function callOutSignedAndBridge(
             address payable gasRefundee,
             bytes calldata params,
             DepositInput memory depositParams,
             GasParams calldata gasParams,
             bool hasFallbackToggled
         ) external payable;
     
         /**
          * @notice Function to perform a call to the Root Omnichain Router while
          *         depositing two or more assets with msg.sender.
          *   @param gasRefundee address to return excess gas deposited in `msg.value` to.
          *   @param params enconded parameters to execute on the root chain router.
          *   @param depositParams additional token deposit parameters.
          *   @param gasParams gas parameters for the cross-chain call.
          *   @param hasFallbackToggled flag to indicate if the fallback function was toggled.
          *   @dev DEPOSIT ID: 6 (Call with multiple deposit and verified sender)
          *
          */
         function callOutSignedAndBridgeMultiple(
             address payable gasRefundee,
             bytes calldata params,
             DepositMultipleInput memory depositParams,
             GasParams calldata gasParams,
             bool hasFallbackToggled
         ) external payable;
     
         /*///////////////////////////////////////////////////////////////
                         DEPOSIT EXTERNAL FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice Function to perform a call to the Root Omnichain Environment
          *         retrying a failed deposit that hasn't been executed yet.
          *   @param isSigned Flag to indicate if the deposit was signed.
          *   @param depositNonce Identifier for user deposit.
          *   @param params parameters to execute on the root chain router.
          *   @param gasParams gas parameters for the cross-chain call.
          *   @param hasFallbackToggled flag to indicate if the fallback function was toggled.
          */
         function retryDeposit(
             bool isSigned,
             uint32 depositNonce,
             bytes calldata params,
             GasParams calldata gasParams,
             bool hasFallbackToggled
         ) external payable;
     
         /**
          * @notice External function to request tokens back to branch chain after failing omnichain environment interaction.
          *    @param depositNonce Identifier for user deposit to retrieve.
          *    @param gasParams gas parameters for the cross-chain call.
          *    @dev DEPOSIT ID: 8
          *
          */
         function retrieveDeposit(uint32 depositNonce, GasParams calldata gasParams) external payable;
     
         /**
          * @notice External function to retry a failed Deposit entry on this branch chain.
          *    @param depositNonce Identifier for user deposit.
          *
          */
         function redeemDeposit(uint32 depositNonce) external;
     
         /*///////////////////////////////////////////////////////////////
                         SETTLEMENT EXTERNAL FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice External function to retry a failed Settlement entry on the root chain.
          *   @param settlementNonce Identifier for user settlement.
          *   @param params parameters to execute on the root chain router.
          *   @param gasParams gas parameters for the cross-chain call to root chain and for the settlement to branch.
          *   @param hasFallbackToggled flag to indicate if the fallback function should be toggled.
          *   @dev DEPOSIT ID: 7
          *
          */
         function retrySettlement(
             uint32 settlementNonce,
             bytes calldata params,
             GasParams[2] calldata gasParams,
             bool hasFallbackToggled
         ) external payable;
     
         /*///////////////////////////////////////////////////////////////
                         TOKEN MANAGEMENT EXTERNAL FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice Function to request balance clearance from a Port to a given user.
          *     @param recipient token receiver.
          *     @param hToken  local hToken addresse to clear balance for.
          *     @param token  native / underlying token addresse to clear balance for.
          *     @param amount amounts of hToken to clear balance for.
          *     @param deposit amount of native / underlying tokens to clear balance for.
          *
          */
         function clearToken(address recipient, address hToken, address token, uint256 amount, uint256 deposit) external;
     
         /**
          * @notice Function to request balance clearance from a Port to a given address.
          *     @param sParams encode packed multiple settlement info.
          *
          */
         function clearTokens(bytes calldata sParams, address recipient)
             external
             returns (SettlementMultipleParams memory);
     
         /*///////////////////////////////////////////////////////////////
                                 LAYER ZERO FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice External function to receive cross-chain messages from LayerZero Endpoint Contract without blocking.
          *   @param _endpoint address of the LayerZero Endpoint Contract.
          *   @param _srcAddress address path of the recipient + sender.
          *   @param _payload Calldata for function call.
          */
         function lzReceiveNonBlocking(address _endpoint, bytes calldata _srcAddress, bytes calldata _payload) external;
     
         /*///////////////////////////////////////////////////////////////
                             EVENTS
         //////////////////////////////////////////////////////////////*/
     
         event LogExecute(uint256 indexed nonce);
         event LogFallback(uint256 indexed nonce);
     
         /*///////////////////////////////////////////////////////////////
                                     ERRORS
         //////////////////////////////////////////////////////////////*/
     
         error UnknownFlag();
         error ExecutionFailure();
     
         error LayerZeroUnauthorizedCaller();
         error LayerZeroUnauthorizedEndpoint();
     
         error AlreadyExecutedTransaction();
     
         error InvalidInput();
         error InsufficientGas();
     
         error NotDepositOwner();
         error DepositRetryUnavailableUseCallout();
         error DepositRedeemUnavailable();
     
         error UnrecognizedRouter();
         error UnrecognizedBridgeAgentExecutor();
     }
     

//@audit  ,params are not checked
227:     function callOutSignedAndBridgeMultiple(
             address payable gasRefundee,
             bytes calldata params,
             DepositMultipleInput memory depositParams,
             GasParams calldata gasParams,
             bool hasFallbackToggled
         ) external payable;
     
         /*///////////////////////////////////////////////////////////////
                         DEPOSIT EXTERNAL FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice Function to perform a call to the Root Omnichain Environment
          *         retrying a failed deposit that hasn't been executed yet.
          *   @param isSigned Flag to indicate if the deposit was signed.
          *   @param depositNonce Identifier for user deposit.
          *   @param params parameters to execute on the root chain router.
          *   @param gasParams gas parameters for the cross-chain call.
          *   @param hasFallbackToggled flag to indicate if the fallback function was toggled.
          */
         function retryDeposit(
             bool isSigned,
             uint32 depositNonce,
             bytes calldata params,
             GasParams calldata gasParams,
             bool hasFallbackToggled
         ) external payable;
     
         /**
          * @notice External function to request tokens back to branch chain after failing omnichain environment interaction.
          *    @param depositNonce Identifier for user deposit to retrieve.
          *    @param gasParams gas parameters for the cross-chain call.
          *    @dev DEPOSIT ID: 8
          *
          */
         function retrieveDeposit(uint32 depositNonce, GasParams calldata gasParams) external payable;
     
         /**
          * @notice External function to retry a failed Deposit entry on this branch chain.
          *    @param depositNonce Identifier for user deposit.
          *
          */
         function redeemDeposit(uint32 depositNonce) external;
     
         /*///////////////////////////////////////////////////////////////
                         SETTLEMENT EXTERNAL FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice External function to retry a failed Settlement entry on the root chain.
          *   @param settlementNonce Identifier for user settlement.
          *   @param params parameters to execute on the root chain router.
          *   @param gasParams gas parameters for the cross-chain call to root chain and for the settlement to branch.
          *   @param hasFallbackToggled flag to indicate if the fallback function should be toggled.
          *   @dev DEPOSIT ID: 7
          *
          */
         function retrySettlement(
             uint32 settlementNonce,
             bytes calldata params,
             GasParams[2] calldata gasParams,
             bool hasFallbackToggled
         ) external payable;
     
         /*///////////////////////////////////////////////////////////////
                         TOKEN MANAGEMENT EXTERNAL FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice Function to request balance clearance from a Port to a given user.
          *     @param recipient token receiver.
          *     @param hToken  local hToken addresse to clear balance for.
          *     @param token  native / underlying token addresse to clear balance for.
          *     @param amount amounts of hToken to clear balance for.
          *     @param deposit amount of native / underlying tokens to clear balance for.
          *
          */
         function clearToken(address recipient, address hToken, address token, uint256 amount, uint256 deposit) external;
     
         /**
          * @notice Function to request balance clearance from a Port to a given address.
          *     @param sParams encode packed multiple settlement info.
          *
          */
         function clearTokens(bytes calldata sParams, address recipient)
             external
             returns (SettlementMultipleParams memory);
     
         /*///////////////////////////////////////////////////////////////
                                 LAYER ZERO FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice External function to receive cross-chain messages from LayerZero Endpoint Contract without blocking.
          *   @param _endpoint address of the LayerZero Endpoint Contract.
          *   @param _srcAddress address path of the recipient + sender.
          *   @param _payload Calldata for function call.
          */
         function lzReceiveNonBlocking(address _endpoint, bytes calldata _srcAddress, bytes calldata _payload) external;
     
         /*///////////////////////////////////////////////////////////////
                             EVENTS
         //////////////////////////////////////////////////////////////*/
     
         event LogExecute(uint256 indexed nonce);
         event LogFallback(uint256 indexed nonce);
     
         /*///////////////////////////////////////////////////////////////
                                     ERRORS
         //////////////////////////////////////////////////////////////*/
     
         error UnknownFlag();
         error ExecutionFailure();
     
         error LayerZeroUnauthorizedCaller();
         error LayerZeroUnauthorizedEndpoint();
     
         error AlreadyExecutedTransaction();
     
         error InvalidInput();
         error InsufficientGas();
     
         error NotDepositOwner();
         error DepositRetryUnavailableUseCallout();
         error DepositRedeemUnavailable();
     
         error UnrecognizedRouter();
         error UnrecognizedBridgeAgentExecutor();
     }
     

//@audit  ,params are not checked
248:     function retryDeposit(
             bool isSigned,
             uint32 depositNonce,
             bytes calldata params,
             GasParams calldata gasParams,
             bool hasFallbackToggled
         ) external payable;
     
         /**
          * @notice External function to request tokens back to branch chain after failing omnichain environment interaction.
          *    @param depositNonce Identifier for user deposit to retrieve.
          *    @param gasParams gas parameters for the cross-chain call.
          *    @dev DEPOSIT ID: 8
          *
          */
         function retrieveDeposit(uint32 depositNonce, GasParams calldata gasParams) external payable;
     
         /**
          * @notice External function to retry a failed Deposit entry on this branch chain.
          *    @param depositNonce Identifier for user deposit.
          *
          */
         function redeemDeposit(uint32 depositNonce) external;
     
         /*///////////////////////////////////////////////////////////////
                         SETTLEMENT EXTERNAL FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice External function to retry a failed Settlement entry on the root chain.
          *   @param settlementNonce Identifier for user settlement.
          *   @param params parameters to execute on the root chain router.
          *   @param gasParams gas parameters for the cross-chain call to root chain and for the settlement to branch.
          *   @param hasFallbackToggled flag to indicate if the fallback function should be toggled.
          *   @dev DEPOSIT ID: 7
          *
          */
         function retrySettlement(
             uint32 settlementNonce,
             bytes calldata params,
             GasParams[2] calldata gasParams,
             bool hasFallbackToggled
         ) external payable;
     
         /*///////////////////////////////////////////////////////////////
                         TOKEN MANAGEMENT EXTERNAL FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice Function to request balance clearance from a Port to a given user.
          *     @param recipient token receiver.
          *     @param hToken  local hToken addresse to clear balance for.
          *     @param token  native / underlying token addresse to clear balance for.
          *     @param amount amounts of hToken to clear balance for.
          *     @param deposit amount of native / underlying tokens to clear balance for.
          *
          */
         function clearToken(address recipient, address hToken, address token, uint256 amount, uint256 deposit) external;
     
         /**
          * @notice Function to request balance clearance from a Port to a given address.
          *     @param sParams encode packed multiple settlement info.
          *
          */
         function clearTokens(bytes calldata sParams, address recipient)
             external
             returns (SettlementMultipleParams memory);
     
         /*///////////////////////////////////////////////////////////////
                                 LAYER ZERO FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice External function to receive cross-chain messages from LayerZero Endpoint Contract without blocking.
          *   @param _endpoint address of the LayerZero Endpoint Contract.
          *   @param _srcAddress address path of the recipient + sender.
          *   @param _payload Calldata for function call.
          */
         function lzReceiveNonBlocking(address _endpoint, bytes calldata _srcAddress, bytes calldata _payload) external;
     
         /*///////////////////////////////////////////////////////////////
                             EVENTS
         //////////////////////////////////////////////////////////////*/
     
         event LogExecute(uint256 indexed nonce);
         event LogFallback(uint256 indexed nonce);
     
         /*///////////////////////////////////////////////////////////////
                                     ERRORS
         //////////////////////////////////////////////////////////////*/
     
         error UnknownFlag();
         error ExecutionFailure();
     
         error LayerZeroUnauthorizedCaller();
         error LayerZeroUnauthorizedEndpoint();
     
         error AlreadyExecutedTransaction();
     
         error InvalidInput();
         error InsufficientGas();
     
         error NotDepositOwner();
         error DepositRetryUnavailableUseCallout();
         error DepositRedeemUnavailable();
     
         error UnrecognizedRouter();
         error UnrecognizedBridgeAgentExecutor();
     }
     

//@audit  ,params are not checked
285:     function retrySettlement(
             uint32 settlementNonce,
             bytes calldata params,
             GasParams[2] calldata gasParams,
             bool hasFallbackToggled
         ) external payable;
     
         /*///////////////////////////////////////////////////////////////
                         TOKEN MANAGEMENT EXTERNAL FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice Function to request balance clearance from a Port to a given user.
          *     @param recipient token receiver.
          *     @param hToken  local hToken addresse to clear balance for.
          *     @param token  native / underlying token addresse to clear balance for.
          *     @param amount amounts of hToken to clear balance for.
          *     @param deposit amount of native / underlying tokens to clear balance for.
          *
          */
         function clearToken(address recipient, address hToken, address token, uint256 amount, uint256 deposit) external;
     
         /**
          * @notice Function to request balance clearance from a Port to a given address.
          *     @param sParams encode packed multiple settlement info.
          *
          */
         function clearTokens(bytes calldata sParams, address recipient)
             external
             returns (SettlementMultipleParams memory);
     
         /*///////////////////////////////////////////////////////////////
                                 LAYER ZERO FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice External function to receive cross-chain messages from LayerZero Endpoint Contract without blocking.
          *   @param _endpoint address of the LayerZero Endpoint Contract.
          *   @param _srcAddress address path of the recipient + sender.
          *   @param _payload Calldata for function call.
          */
         function lzReceiveNonBlocking(address _endpoint, bytes calldata _srcAddress, bytes calldata _payload) external;
     
         /*///////////////////////////////////////////////////////////////
                             EVENTS
         //////////////////////////////////////////////////////////////*/
     
         event LogExecute(uint256 indexed nonce);
         event LogFallback(uint256 indexed nonce);
     
         /*///////////////////////////////////////////////////////////////
                                     ERRORS
         //////////////////////////////////////////////////////////////*/
     
         error UnknownFlag();
         error ExecutionFailure();
     
         error LayerZeroUnauthorizedCaller();
         error LayerZeroUnauthorizedEndpoint();
     
         error AlreadyExecutedTransaction();
     
         error InvalidInput();
         error InsufficientGas();
     
         error NotDepositOwner();
         error DepositRetryUnavailableUseCallout();
         error DepositRedeemUnavailable();
     
         error UnrecognizedRouter();
         error UnrecognizedBridgeAgentExecutor();
     }
     

//@audit  ,sParams are not checked
312:     function clearTokens(bytes calldata sParams, address recipient)
             external
             returns (SettlementMultipleParams memory);
     
         /*///////////////////////////////////////////////////////////////
                                 LAYER ZERO FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice External function to receive cross-chain messages from LayerZero Endpoint Contract without blocking.
          *   @param _endpoint address of the LayerZero Endpoint Contract.
          *   @param _srcAddress address path of the recipient + sender.
          *   @param _payload Calldata for function call.
          */
         function lzReceiveNonBlocking(address _endpoint, bytes calldata _srcAddress, bytes calldata _payload) external;
     
         /*///////////////////////////////////////////////////////////////
                             EVENTS
         //////////////////////////////////////////////////////////////*/
     
         event LogExecute(uint256 indexed nonce);
         event LogFallback(uint256 indexed nonce);
     
         /*///////////////////////////////////////////////////////////////
                                     ERRORS
         //////////////////////////////////////////////////////////////*/
     
         error UnknownFlag();
         error ExecutionFailure();
     
         error LayerZeroUnauthorizedCaller();
         error LayerZeroUnauthorizedEndpoint();
     
         error AlreadyExecutedTransaction();
     
         error InvalidInput();
         error InsufficientGas();
     
         error NotDepositOwner();
         error DepositRetryUnavailableUseCallout();
         error DepositRedeemUnavailable();
     
         error UnrecognizedRouter();
         error UnrecognizedBridgeAgentExecutor();
     }
     

//@audit  ,_srcAddress ,_payload are not checked
326:     function lzReceiveNonBlocking(address _endpoint, bytes calldata _srcAddress, bytes calldata _payload) external;
     
         /*///////////////////////////////////////////////////////////////
                             EVENTS
         //////////////////////////////////////////////////////////////*/
     
         event LogExecute(uint256 indexed nonce);
         event LogFallback(uint256 indexed nonce);
     
         /*///////////////////////////////////////////////////////////////
                                     ERRORS
         //////////////////////////////////////////////////////////////*/
     
         error UnknownFlag();
         error ExecutionFailure();
     
         error LayerZeroUnauthorizedCaller();
         error LayerZeroUnauthorizedEndpoint();
     
         error AlreadyExecutedTransaction();
     
         error InvalidInput();
         error InsufficientGas();
     
         error NotDepositOwner();
         error DepositRetryUnavailableUseCallout();
         error DepositRedeemUnavailable();
     
         error UnrecognizedRouter();
         error UnrecognizedBridgeAgentExecutor();
     }
     

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/IBranchBridgeAgent.sol#L326-L357)

```solidity
File: src/interfaces/IBranchRouter.sol

//@audit  ,params are not checked
46:     function callOut(bytes calldata params, GasParams calldata gParams) external payable;
    
        /**
         * @notice Function to perform a call to the Root Omnichain Router while depositing a single asset.
         *   @param params encoded parameters to execute on the root chain.
         *   @param dParams additional token deposit parameters.
         *   @param gParams gas parameters for the cross-chain call.
         *   @dev ACTION ID: 2 (Call with single deposit)
         *
         */
        function callOutAndBridge(bytes calldata params, DepositInput calldata dParams, GasParams calldata gParams)
            external
            payable;
    
        /**
         * @notice Function to perform a call to the Root Omnichain Router while depositing two or more assets.
         *   @param params encoded parameters to execute on the root chain.
         *   @param dParams additional token deposit parameters.
         *   @param gParams gas parameters for the cross-chain call.
         *   @dev ACTION ID: 3 (Call with multiple deposit)
         *
         */
        function callOutAndBridgeMultiple(
            bytes calldata params,
            DepositMultipleInput calldata dParams,
            GasParams calldata gParams
        ) external payable;
    
        /**
         * @notice External function that returns a given deposit entry.
         *     @param depositNonce Identifier for user deposit.
         *
         */
        function getDepositEntry(uint32 depositNonce) external view returns (Deposit memory);
    
        /*///////////////////////////////////////////////////////////////
                            LAYERZERO EXTERNAL FUNCTIONS
        //////////////////////////////////////////////////////////////*/
    
        /**
         * @notice Function responsible of executing a branch router response.
         *     @param params data received from messaging layer.
         */
        function executeNoSettlement(bytes calldata params) external payable;
    
        /**
         * @dev Function responsible of executing a crosschain request without any deposit.
         *     @param params data received from messaging layer.
         *     @param sParams SettlementParams struct.
         */
        function executeSettlement(bytes calldata params, SettlementParams calldata sParams) external payable;
    
        /**
         * @dev Function responsible of executing a crosschain request which contains
         *      cross-chain deposit information attached.
         *     @param params data received from messaging layer.
         *     @param sParams SettlementParams struct containing deposit information.
         *
         */
        function executeSettlementMultiple(bytes calldata params, SettlementMultipleParams calldata sParams)
            external
            payable;
    
        /*///////////////////////////////////////////////////////////////
                                 ERRORS
        //////////////////////////////////////////////////////////////*/
    
        error UnrecognizedFunctionId();
    
        error UnrecognizedBridgeAgentExecutor();
    }
    

//@audit  ,params are not checked
56:     function callOutAndBridge(bytes calldata params, DepositInput calldata dParams, GasParams calldata gParams)
            external
            payable;
    
        /**
         * @notice Function to perform a call to the Root Omnichain Router while depositing two or more assets.
         *   @param params encoded parameters to execute on the root chain.
         *   @param dParams additional token deposit parameters.
         *   @param gParams gas parameters for the cross-chain call.
         *   @dev ACTION ID: 3 (Call with multiple deposit)
         *
         */
        function callOutAndBridgeMultiple(
            bytes calldata params,
            DepositMultipleInput calldata dParams,
            GasParams calldata gParams
        ) external payable;
    
        /**
         * @notice External function that returns a given deposit entry.
         *     @param depositNonce Identifier for user deposit.
         *
         */
        function getDepositEntry(uint32 depositNonce) external view returns (Deposit memory);
    
        /*///////////////////////////////////////////////////////////////
                            LAYERZERO EXTERNAL FUNCTIONS
        //////////////////////////////////////////////////////////////*/
    
        /**
         * @notice Function responsible of executing a branch router response.
         *     @param params data received from messaging layer.
         */
        function executeNoSettlement(bytes calldata params) external payable;
    
        /**
         * @dev Function responsible of executing a crosschain request without any deposit.
         *     @param params data received from messaging layer.
         *     @param sParams SettlementParams struct.
         */
        function executeSettlement(bytes calldata params, SettlementParams calldata sParams) external payable;
    
        /**
         * @dev Function responsible of executing a crosschain request which contains
         *      cross-chain deposit information attached.
         *     @param params data received from messaging layer.
         *     @param sParams SettlementParams struct containing deposit information.
         *
         */
        function executeSettlementMultiple(bytes calldata params, SettlementMultipleParams calldata sParams)
            external
            payable;
    
        /*///////////////////////////////////////////////////////////////
                                 ERRORS
        //////////////////////////////////////////////////////////////*/
    
        error UnrecognizedFunctionId();
    
        error UnrecognizedBridgeAgentExecutor();
    }
    

//@audit  ,params are not checked
68:     function callOutAndBridgeMultiple(
            bytes calldata params,
            DepositMultipleInput calldata dParams,
            GasParams calldata gParams
        ) external payable;
    
        /**
         * @notice External function that returns a given deposit entry.
         *     @param depositNonce Identifier for user deposit.
         *
         */
        function getDepositEntry(uint32 depositNonce) external view returns (Deposit memory);
    
        /*///////////////////////////////////////////////////////////////
                            LAYERZERO EXTERNAL FUNCTIONS
        //////////////////////////////////////////////////////////////*/
    
        /**
         * @notice Function responsible of executing a branch router response.
         *     @param params data received from messaging layer.
         */
        function executeNoSettlement(bytes calldata params) external payable;
    
        /**
         * @dev Function responsible of executing a crosschain request without any deposit.
         *     @param params data received from messaging layer.
         *     @param sParams SettlementParams struct.
         */
        function executeSettlement(bytes calldata params, SettlementParams calldata sParams) external payable;
    
        /**
         * @dev Function responsible of executing a crosschain request which contains
         *      cross-chain deposit information attached.
         *     @param params data received from messaging layer.
         *     @param sParams SettlementParams struct containing deposit information.
         *
         */
        function executeSettlementMultiple(bytes calldata params, SettlementMultipleParams calldata sParams)
            external
            payable;
    
        /*///////////////////////////////////////////////////////////////
                                 ERRORS
        //////////////////////////////////////////////////////////////*/
    
        error UnrecognizedFunctionId();
    
        error UnrecognizedBridgeAgentExecutor();
    }
    

//@audit  ,params are not checked
89:     function executeNoSettlement(bytes calldata params) external payable;
    
        /**
         * @dev Function responsible of executing a crosschain request without any deposit.
         *     @param params data received from messaging layer.
         *     @param sParams SettlementParams struct.
         */
        function executeSettlement(bytes calldata params, SettlementParams calldata sParams) external payable;
    
        /**
         * @dev Function responsible of executing a crosschain request which contains
         *      cross-chain deposit information attached.
         *     @param params data received from messaging layer.
         *     @param sParams SettlementParams struct containing deposit information.
         *
         */
        function executeSettlementMultiple(bytes calldata params, SettlementMultipleParams calldata sParams)
            external
            payable;
    
        /*///////////////////////////////////////////////////////////////
                                 ERRORS
        //////////////////////////////////////////////////////////////*/
    
        error UnrecognizedFunctionId();
    
        error UnrecognizedBridgeAgentExecutor();
    }
    

//@audit  ,params are not checked
96:     function executeSettlement(bytes calldata params, SettlementParams calldata sParams) external payable;
    
        /**
         * @dev Function responsible of executing a crosschain request which contains
         *      cross-chain deposit information attached.
         *     @param params data received from messaging layer.
         *     @param sParams SettlementParams struct containing deposit information.
         *
         */
        function executeSettlementMultiple(bytes calldata params, SettlementMultipleParams calldata sParams)
            external
            payable;
    
        /*///////////////////////////////////////////////////////////////
                                 ERRORS
        //////////////////////////////////////////////////////////////*/
    
        error UnrecognizedFunctionId();
    
        error UnrecognizedBridgeAgentExecutor();
    }
    

//@audit  ,params are not checked
105:     function executeSettlementMultiple(bytes calldata params, SettlementMultipleParams calldata sParams)
             external
             payable;
     
         /*///////////////////////////////////////////////////////////////
                                  ERRORS
         //////////////////////////////////////////////////////////////*/
     
         error UnrecognizedFunctionId();
     
         error UnrecognizedBridgeAgentExecutor();
     }
     

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/IBranchRouter.sol#L105-L117)

```solidity
File: src/interfaces/ILayerZeroEndpoint.sol

//@audit  ,_destination ,_payload ,_adapterParams are not checked
15:     function send(
            uint16 _dstChainId,
            bytes calldata _destination,
            bytes calldata _payload,
            address payable _refundAddress,
            address _zroPaymentAddress,
            bytes calldata _adapterParams
        ) external payable;
    
        // @notice used by the messaging library to publish verified payload
        // @param _srcChainId - the source chain identifier
        // @param _srcAddress - the source contract (as bytes) at the source chain
        // @param _dstAddress - the address on destination chain
        // @param _nonce - the unbound message ordering nonce
        // @param _gasLimit - the gas limit for external contract execution
        // @param _payload - verified payload to send to the destination contract
        function receivePayload(
            uint16 _srcChainId,
            bytes calldata _srcAddress,
            address _dstAddress,
            uint64 _nonce,
            uint256 _gasLimit,
            bytes calldata _payload
        ) external;
    
        // @notice get the inboundNonce of a receiver from a source chain which could be EVM or non-EVM chain
        // @param _srcChainId - the source chain identifier
        // @param _srcAddress - the source chain contract address
        function getInboundNonce(uint16 _srcChainId, bytes calldata _srcAddress) external view returns (uint64);
    
        // @notice get the outboundNonce from this source chain which, consequently, is always an EVM
        // @param _srcAddress - the source chain contract address
        function getOutboundNonce(uint16 _dstChainId, address _srcAddress) external view returns (uint64);
    
        // @notice gets a quote in source native gas, for the amount that send() requires to pay for message delivery
        // @param _dstChainId - the destination chain identifier
        // @param _userApplication - the user app address on this EVM chain
        // @param _payload - the custom message to send over LayerZero
        // @param _payInZRO - if false, user app pays the protocol fee in native token
        // @param _adapterParam - parameters for the adapter service, e.g. send some dust native token to dstChain
        function estimateFees(
            uint16 _dstChainId,
            address _userApplication,
            bytes calldata _payload,
            bool _payInZRO,
            bytes calldata _adapterParam
        ) external view returns (uint256 nativeFee, uint256 zroFee);
    
        // @notice get this Endpoint's immutable source identifier
        function getChainId() external view returns (uint16);
    
        // @notice the interface to retry failed message on this Endpoint destination
        // @param _srcChainId - the source chain identifier
        // @param _srcAddress - the source chain contract address
        // @param _payload - the payload to be retried
        function retryPayload(uint16 _srcChainId, bytes calldata _srcAddress, bytes calldata _payload) external;
    
        // @notice query if any STORED payload (message blocking) at the endpoint.
        // @param _srcChainId - the source chain identifier
        // @param _srcAddress - the source chain contract address
        function hasStoredPayload(uint16 _srcChainId, bytes calldata _srcAddress) external view returns (bool);
    
        // @notice query if the _libraryAddress is valid for sending msgs.
        // @param _userApplication - the user app address on this EVM chain
        function getSendLibraryAddress(address _userApplication) external view returns (address);
    
        // @notice query if the _libraryAddress is valid for receiving msgs.
        // @param _userApplication - the user app address on this EVM chain
        function getReceiveLibraryAddress(address _userApplication) external view returns (address);
    
        // @notice query if the non-reentrancy guard for send() is on
        // @return true if the guard is on. false otherwise
        function isSendingPayload() external view returns (bool);
    
        // @notice query if the non-reentrancy guard for receive() is on
        // @return true if the guard is on. false otherwise
        function isReceivingPayload() external view returns (bool);
    
        // @notice get the configuration of the LayerZero messaging library of the specified version
        // @param _version - messaging library version
        // @param _chainId - the chainId for the pending config change
        // @param _userApplication - the contract address of the user application
        // @param _configType - type of configuration. every messaging library has its own convention.
        function getConfig(uint16 _version, uint16 _chainId, address _userApplication, uint256 _configType)
            external
            view
            returns (bytes memory);
    
        // @notice get the send() LayerZero messaging library version
        // @param _userApplication - the contract address of the user application
        function getSendVersion(address _userApplication) external view returns (uint16);
    
        // @notice get the lzReceive() LayerZero messaging library version
        // @param _userApplication - the contract address of the user application
        function getReceiveVersion(address _userApplication) external view returns (uint16);
    }
    

//@audit  ,_srcAddress ,_payload are not checked
31:     function receivePayload(
            uint16 _srcChainId,
            bytes calldata _srcAddress,
            address _dstAddress,
            uint64 _nonce,
            uint256 _gasLimit,
            bytes calldata _payload
        ) external;
    
        // @notice get the inboundNonce of a receiver from a source chain which could be EVM or non-EVM chain
        // @param _srcChainId - the source chain identifier
        // @param _srcAddress - the source chain contract address
        function getInboundNonce(uint16 _srcChainId, bytes calldata _srcAddress) external view returns (uint64);
    
        // @notice get the outboundNonce from this source chain which, consequently, is always an EVM
        // @param _srcAddress - the source chain contract address
        function getOutboundNonce(uint16 _dstChainId, address _srcAddress) external view returns (uint64);
    
        // @notice gets a quote in source native gas, for the amount that send() requires to pay for message delivery
        // @param _dstChainId - the destination chain identifier
        // @param _userApplication - the user app address on this EVM chain
        // @param _payload - the custom message to send over LayerZero
        // @param _payInZRO - if false, user app pays the protocol fee in native token
        // @param _adapterParam - parameters for the adapter service, e.g. send some dust native token to dstChain
        function estimateFees(
            uint16 _dstChainId,
            address _userApplication,
            bytes calldata _payload,
            bool _payInZRO,
            bytes calldata _adapterParam
        ) external view returns (uint256 nativeFee, uint256 zroFee);
    
        // @notice get this Endpoint's immutable source identifier
        function getChainId() external view returns (uint16);
    
        // @notice the interface to retry failed message on this Endpoint destination
        // @param _srcChainId - the source chain identifier
        // @param _srcAddress - the source chain contract address
        // @param _payload - the payload to be retried
        function retryPayload(uint16 _srcChainId, bytes calldata _srcAddress, bytes calldata _payload) external;
    
        // @notice query if any STORED payload (message blocking) at the endpoint.
        // @param _srcChainId - the source chain identifier
        // @param _srcAddress - the source chain contract address
        function hasStoredPayload(uint16 _srcChainId, bytes calldata _srcAddress) external view returns (bool);
    
        // @notice query if the _libraryAddress is valid for sending msgs.
        // @param _userApplication - the user app address on this EVM chain
        function getSendLibraryAddress(address _userApplication) external view returns (address);
    
        // @notice query if the _libraryAddress is valid for receiving msgs.
        // @param _userApplication - the user app address on this EVM chain
        function getReceiveLibraryAddress(address _userApplication) external view returns (address);
    
        // @notice query if the non-reentrancy guard for send() is on
        // @return true if the guard is on. false otherwise
        function isSendingPayload() external view returns (bool);
    
        // @notice query if the non-reentrancy guard for receive() is on
        // @return true if the guard is on. false otherwise
        function isReceivingPayload() external view returns (bool);
    
        // @notice get the configuration of the LayerZero messaging library of the specified version
        // @param _version - messaging library version
        // @param _chainId - the chainId for the pending config change
        // @param _userApplication - the contract address of the user application
        // @param _configType - type of configuration. every messaging library has its own convention.
        function getConfig(uint16 _version, uint16 _chainId, address _userApplication, uint256 _configType)
            external
            view
            returns (bytes memory);
    
        // @notice get the send() LayerZero messaging library version
        // @param _userApplication - the contract address of the user application
        function getSendVersion(address _userApplication) external view returns (uint16);
    
        // @notice get the lzReceive() LayerZero messaging library version
        // @param _userApplication - the contract address of the user application
        function getReceiveVersion(address _userApplication) external view returns (uint16);
    }
    

//@audit  ,_srcAddress are not checked
43:     function getInboundNonce(uint16 _srcChainId, bytes calldata _srcAddress) external view returns (uint64);
    
        // @notice get the outboundNonce from this source chain which, consequently, is always an EVM
        // @param _srcAddress - the source chain contract address
        function getOutboundNonce(uint16 _dstChainId, address _srcAddress) external view returns (uint64);
    
        // @notice gets a quote in source native gas, for the amount that send() requires to pay for message delivery
        // @param _dstChainId - the destination chain identifier
        // @param _userApplication - the user app address on this EVM chain
        // @param _payload - the custom message to send over LayerZero
        // @param _payInZRO - if false, user app pays the protocol fee in native token
        // @param _adapterParam - parameters for the adapter service, e.g. send some dust native token to dstChain
        function estimateFees(
            uint16 _dstChainId,
            address _userApplication,
            bytes calldata _payload,
            bool _payInZRO,
            bytes calldata _adapterParam
        ) external view returns (uint256 nativeFee, uint256 zroFee);
    
        // @notice get this Endpoint's immutable source identifier
        function getChainId() external view returns (uint16);
    
        // @notice the interface to retry failed message on this Endpoint destination
        // @param _srcChainId - the source chain identifier
        // @param _srcAddress - the source chain contract address
        // @param _payload - the payload to be retried
        function retryPayload(uint16 _srcChainId, bytes calldata _srcAddress, bytes calldata _payload) external;
    
        // @notice query if any STORED payload (message blocking) at the endpoint.
        // @param _srcChainId - the source chain identifier
        // @param _srcAddress - the source chain contract address
        function hasStoredPayload(uint16 _srcChainId, bytes calldata _srcAddress) external view returns (bool);
    
        // @notice query if the _libraryAddress is valid for sending msgs.
        // @param _userApplication - the user app address on this EVM chain
        function getSendLibraryAddress(address _userApplication) external view returns (address);
    
        // @notice query if the _libraryAddress is valid for receiving msgs.
        // @param _userApplication - the user app address on this EVM chain
        function getReceiveLibraryAddress(address _userApplication) external view returns (address);
    
        // @notice query if the non-reentrancy guard for send() is on
        // @return true if the guard is on. false otherwise
        function isSendingPayload() external view returns (bool);
    
        // @notice query if the non-reentrancy guard for receive() is on
        // @return true if the guard is on. false otherwise
        function isReceivingPayload() external view returns (bool);
    
        // @notice get the configuration of the LayerZero messaging library of the specified version
        // @param _version - messaging library version
        // @param _chainId - the chainId for the pending config change
        // @param _userApplication - the contract address of the user application
        // @param _configType - type of configuration. every messaging library has its own convention.
        function getConfig(uint16 _version, uint16 _chainId, address _userApplication, uint256 _configType)
            external
            view
            returns (bytes memory);
    
        // @notice get the send() LayerZero messaging library version
        // @param _userApplication - the contract address of the user application
        function getSendVersion(address _userApplication) external view returns (uint16);
    
        // @notice get the lzReceive() LayerZero messaging library version
        // @param _userApplication - the contract address of the user application
        function getReceiveVersion(address _userApplication) external view returns (uint16);
    }
    

//@audit  ,_payload ,_adapterParam are not checked
55:     function estimateFees(
            uint16 _dstChainId,
            address _userApplication,
            bytes calldata _payload,
            bool _payInZRO,
            bytes calldata _adapterParam
        ) external view returns (uint256 nativeFee, uint256 zroFee);
    
        // @notice get this Endpoint's immutable source identifier
        function getChainId() external view returns (uint16);
    
        // @notice the interface to retry failed message on this Endpoint destination
        // @param _srcChainId - the source chain identifier
        // @param _srcAddress - the source chain contract address
        // @param _payload - the payload to be retried
        function retryPayload(uint16 _srcChainId, bytes calldata _srcAddress, bytes calldata _payload) external;
    
        // @notice query if any STORED payload (message blocking) at the endpoint.
        // @param _srcChainId - the source chain identifier
        // @param _srcAddress - the source chain contract address
        function hasStoredPayload(uint16 _srcChainId, bytes calldata _srcAddress) external view returns (bool);
    
        // @notice query if the _libraryAddress is valid for sending msgs.
        // @param _userApplication - the user app address on this EVM chain
        function getSendLibraryAddress(address _userApplication) external view returns (address);
    
        // @notice query if the _libraryAddress is valid for receiving msgs.
        // @param _userApplication - the user app address on this EVM chain
        function getReceiveLibraryAddress(address _userApplication) external view returns (address);
    
        // @notice query if the non-reentrancy guard for send() is on
        // @return true if the guard is on. false otherwise
        function isSendingPayload() external view returns (bool);
    
        // @notice query if the non-reentrancy guard for receive() is on
        // @return true if the guard is on. false otherwise
        function isReceivingPayload() external view returns (bool);
    
        // @notice get the configuration of the LayerZero messaging library of the specified version
        // @param _version - messaging library version
        // @param _chainId - the chainId for the pending config change
        // @param _userApplication - the contract address of the user application
        // @param _configType - type of configuration. every messaging library has its own convention.
        function getConfig(uint16 _version, uint16 _chainId, address _userApplication, uint256 _configType)
            external
            view
            returns (bytes memory);
    
        // @notice get the send() LayerZero messaging library version
        // @param _userApplication - the contract address of the user application
        function getSendVersion(address _userApplication) external view returns (uint16);
    
        // @notice get the lzReceive() LayerZero messaging library version
        // @param _userApplication - the contract address of the user application
        function getReceiveVersion(address _userApplication) external view returns (uint16);
    }
    

//@audit  ,_srcAddress ,_payload are not checked
70:     function retryPayload(uint16 _srcChainId, bytes calldata _srcAddress, bytes calldata _payload) external;
    
        // @notice query if any STORED payload (message blocking) at the endpoint.
        // @param _srcChainId - the source chain identifier
        // @param _srcAddress - the source chain contract address
        function hasStoredPayload(uint16 _srcChainId, bytes calldata _srcAddress) external view returns (bool);
    
        // @notice query if the _libraryAddress is valid for sending msgs.
        // @param _userApplication - the user app address on this EVM chain
        function getSendLibraryAddress(address _userApplication) external view returns (address);
    
        // @notice query if the _libraryAddress is valid for receiving msgs.
        // @param _userApplication - the user app address on this EVM chain
        function getReceiveLibraryAddress(address _userApplication) external view returns (address);
    
        // @notice query if the non-reentrancy guard for send() is on
        // @return true if the guard is on. false otherwise
        function isSendingPayload() external view returns (bool);
    
        // @notice query if the non-reentrancy guard for receive() is on
        // @return true if the guard is on. false otherwise
        function isReceivingPayload() external view returns (bool);
    
        // @notice get the configuration of the LayerZero messaging library of the specified version
        // @param _version - messaging library version
        // @param _chainId - the chainId for the pending config change
        // @param _userApplication - the contract address of the user application
        // @param _configType - type of configuration. every messaging library has its own convention.
        function getConfig(uint16 _version, uint16 _chainId, address _userApplication, uint256 _configType)
            external
            view
            returns (bytes memory);
    
        // @notice get the send() LayerZero messaging library version
        // @param _userApplication - the contract address of the user application
        function getSendVersion(address _userApplication) external view returns (uint16);
    
        // @notice get the lzReceive() LayerZero messaging library version
        // @param _userApplication - the contract address of the user application
        function getReceiveVersion(address _userApplication) external view returns (uint16);
    }
    

//@audit  ,_srcAddress are not checked
75:     function hasStoredPayload(uint16 _srcChainId, bytes calldata _srcAddress) external view returns (bool);
    
        // @notice query if the _libraryAddress is valid for sending msgs.
        // @param _userApplication - the user app address on this EVM chain
        function getSendLibraryAddress(address _userApplication) external view returns (address);
    
        // @notice query if the _libraryAddress is valid for receiving msgs.
        // @param _userApplication - the user app address on this EVM chain
        function getReceiveLibraryAddress(address _userApplication) external view returns (address);
    
        // @notice query if the non-reentrancy guard for send() is on
        // @return true if the guard is on. false otherwise
        function isSendingPayload() external view returns (bool);
    
        // @notice query if the non-reentrancy guard for receive() is on
        // @return true if the guard is on. false otherwise
        function isReceivingPayload() external view returns (bool);
    
        // @notice get the configuration of the LayerZero messaging library of the specified version
        // @param _version - messaging library version
        // @param _chainId - the chainId for the pending config change
        // @param _userApplication - the contract address of the user application
        // @param _configType - type of configuration. every messaging library has its own convention.
        function getConfig(uint16 _version, uint16 _chainId, address _userApplication, uint256 _configType)
            external
            view
            returns (bytes memory);
    
        // @notice get the send() LayerZero messaging library version
        // @param _userApplication - the contract address of the user application
        function getSendVersion(address _userApplication) external view returns (uint16);
    
        // @notice get the lzReceive() LayerZero messaging library version
        // @param _userApplication - the contract address of the user application
        function getReceiveVersion(address _userApplication) external view returns (uint16);
    }
    

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/ILayerZeroEndpoint.sol#L75-L111)

```solidity
File: src/interfaces/ILayerZeroReceiver.sol

//@audit  ,_srcAddress ,_payload are not checked
11:     function lzReceive(uint16 _srcChainId, bytes calldata _srcAddress, uint64 _nonce, bytes calldata _payload) external;
    }
    

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/ILayerZeroReceiver.sol#L11-L13)

```solidity
File: src/interfaces/ILayerZeroUserApplicationConfig.sol

//@audit  ,_config are not checked
11:     function setConfig(uint16 _version, uint16 _chainId, uint256 _configType, bytes calldata _config) external;
    
        // @notice set the send() LayerZero messaging library version to _version
        // @param _version - new messaging library version
        function setSendVersion(uint16 _version) external;
    
        // @notice set the lzReceive() LayerZero messaging library version to _version
        // @param _version - new messaging library version
        function setReceiveVersion(uint16 _version) external;
    
        // @notice Only when the UA needs to resume the message flow in blocking mode and clear the stored payload
        // @param _srcChainId - the chainId of the source chain
        // @param _srcAddress - the contract address of the source contract at the source chain
        function forceResumeReceive(uint16 _srcChainId, bytes calldata _srcAddress) external;
    }
    

//@audit  ,_srcAddress are not checked
24:     function forceResumeReceive(uint16 _srcChainId, bytes calldata _srcAddress) external;
    }
    

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/ILayerZeroUserApplicationConfig.sol#L24-L26)

```solidity
File: src/interfaces/IRootBridgeAgent.sol

//@audit  ,_payload are not checked
149:     function getFeeEstimate(
             uint256 _gasLimit,
             uint256 _remoteBranchExecutionGas,
             bytes calldata _payload,
             uint16 _dstChainId
         ) external view returns (uint256 _fee);
     
         /*///////////////////////////////////////////////////////////////
                             ROOT ROUTER CALL FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice External function performs call to LayerZero Endpoint Contract for cross-chain messaging.
          *   @param _recipient address to receive any outstanding gas on the destination chain.
          *   @param _dstChainId Chain to bridge to.
          *   @param _params Calldata for function call.
          *   @param _gParams Gas Parameters for cross-chain message.
          *   @dev Internal function performs call to LayerZero Endpoint Contract for cross-chain messaging.
          */
         function callOut(
             address payable _refundee,
             address _recipient,
             uint16 _dstChainId,
             bytes memory _params,
             GasParams calldata _gParams
         ) external payable;
     
         /**
          * @notice External function to move assets from root chain to branch omnichain environment.
          *   @param _refundee the effective owner of the settlement this address receives excess gas deposited on source chain
          *                    for a cross-chain call and is allowed to redeeming assets after a failed settlement fallback.
          *                    This address' Virtual Account is also allowed.
          *   @param _recipient recipient of bridged tokens and any outstanding gas on the destination chain.
          *   @param _dstChainId chain to bridge to.
          *   @param _params parameters for function call on branch chain.
          *   @param _sParams settlement parameters for asset bridging to branch chains.
          *   @param _gParams Gas Parameters for cross-chain message.
          *   @param _hasFallbackToggled Flag to toggle fallback function.
          *
          */
         function callOutAndBridge(
             address payable _refundee,
             address _recipient,
             uint16 _dstChainId,
             bytes calldata _params,
             SettlementInput calldata _sParams,
             GasParams calldata _gParams,
             bool _hasFallbackToggled
         ) external payable;
     
         /**
          * @notice External function to move assets from branch chain to root omnichain environment.
          *   @param _refundee the effective owner of the settlement this address receives excess gas deposited on source chain
          *                    for a cross-chain call and is allowed to redeeming assets after a failed settlement fallback.
          *                    This address' Virtual Account is also allowed.
          *   @param _recipient recipient of bridged tokens.
          *   @param _dstChainId chain to bridge to.
          *   @param _params parameters for function call on branch chain.
          *   @param _sParams settlement parameters for asset bridging to branch chains.
          *   @param _gParams Gas Parameters for cross-chain message.
          *   @param _hasFallbackToggled Flag to toggle fallback function.
          *
          *
          */
         function callOutAndBridgeMultiple(
             address payable _refundee,
             address _recipient,
             uint16 _dstChainId,
             bytes calldata _params,
             SettlementMultipleInput calldata _sParams,
             GasParams calldata _gParams,
             bool _hasFallbackToggled
         ) external payable;
     
         /*///////////////////////////////////////////////////////////////
                             SETTLEMENT FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice Function to retry a user's Settlement balance.
          *   @param _settlementNonce Identifier for token settlement.
          *   @param _recipient recipient of bridged tokens and any outstanding gas on the destination chain.
          *   @param _params Calldata for function call in branch chain.
          *   @param _gParams Gas Parameters for cross-chain message.
          *   @param _hasFallbackToggled Flag to toggle fallback function.
          *
          */
         function retrySettlement(
             uint32 _settlementNonce,
             address _recipient,
             bytes calldata _params,
             GasParams calldata _gParams,
             bool _hasFallbackToggled
         ) external payable;
     
         /**
          * @notice Function that allows retrieval of failed Settlement's foricng fallback to be triggered.
          *   @param _settlementNonce Identifier for token settlement.
          *   @param _gParams Gas Parameters for cross-chain message.
          *
          */
         function retrieveSettlement(uint32 _settlementNonce, GasParams calldata _gParams) external payable;
     
         /**
          * @notice Function that allows redemption of failed Settlement's global tokens.
          *   @param _depositNonce Identifier for token deposit.
          *
          */
         function redeemSettlement(uint32 _depositNonce) external;
     
         /*///////////////////////////////////////////////////////////////
                         TOKEN MANAGEMENT FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice Function to move assets from branch chain to root omnichain environment.
          * @dev Called in response to Bridge Agent Executor.
          *   @param _recipient recipient of bridged token.
          *   @param _dParams Cross-Chain Deposit of Multiple Tokens Params.
          *   @param _srcChainId chain to bridge from.
          *
          */
         function bridgeIn(address _recipient, DepositParams memory _dParams, uint256 _srcChainId) external;
     
         /**
          * @notice Function to move assets from branch chain to root omnichain environment.
          * @dev Called in response to Bridge Agent Executor.
          *   @param _recipient recipient of bridged tokens.
          *   @param _dParams Cross-Chain Deposit of Multiple Tokens Params.
          *   @param _srcChainId chain to bridge from.
          *   @dev Since the input data is encodePacked we need to parse it:
          *     1. First byte is the number of assets to be bridged in. Equals length of all arrays.
          *     2. Next 4 bytes are the nonce of the deposit.
          *     3. Last 32 bytes after the token related information are the chain to bridge to.
          *     4. Token related information starts at index PARAMS_TKN_START is encoded as follows:
          *         1. N * 32 bytes for the hToken address.
          *         2. N * 32 bytes for the underlying token address.
          *         3. N * 32 bytes for the amount of hTokens to be bridged in.
          *         4. N * 32 bytes for the amount of underlying tokens to be bridged in.
          *     5. Each of the 4 token related arrays are of length N and start at the following indexes:
          *         1. PARAMS_TKN_START [hToken address has no offset from token information start].
          *         2. PARAMS_TKN_START + (PARAMS_ADDRESS_SIZE * N)
          *         3. PARAMS_TKN_START + (PARAMS_AMT_OFFSET * N)
          *         4. PARAMS_TKN_START + (PARAMS_DEPOSIT_OFFSET * N)
          *
          */
         function bridgeInMultiple(address _recipient, DepositMultipleParams calldata _dParams, uint256 _srcChainId)
             external;
     
         /*///////////////////////////////////////////////////////////////
                                 LAYER ZERO FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice External function to receive cross-chain messages from LayerZero Endpoint Contract without blocking.
          *   @param _endpoint address of the LayerZero Endpoint Contract.
          *   @param _srcChainId Chain ID of the sender.
          *   @param _srcAddress address path of the recipient + sender.
          *   @param _payload Calldata for function call.
          */
         function lzReceiveNonBlocking(
             address _endpoint,
             uint16 _srcChainId,
             bytes calldata _srcAddress,
             bytes calldata _payload
         ) external;
     
         /*///////////////////////////////////////////////////////////////
                                 ADMIN FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice Adds a new branch bridge agent to a given branch chainId
          *   @param _branchChainId chainId of the branch chain
          */
         function approveBranchBridgeAgent(uint256 _branchChainId) external;
     
         /**
          * @notice Updates the address of the branch bridge agent
          *   @param _newBranchBridgeAgent address of the new branch bridge agent
          *   @param _branchChainId chainId of the branch chain
          */
         function syncBranchBridgeAgent(address _newBranchBridgeAgent, uint256 _branchChainId) external;
     
         /*///////////////////////////////////////////////////////////////
                                  EVENTS
         //////////////////////////////////////////////////////////////*/
     
         event LogExecute(uint256 indexed depositNonce, uint256 indexed srcChainId);
         event LogFallback(uint256 indexed settlementNonce, uint256 indexed dstChainId);
     
         /*///////////////////////////////////////////////////////////////
                                 ERRORS
         //////////////////////////////////////////////////////////////*/
     
         error ExecutionFailure();
         error GasErrorOrRepeatedTx();
         error AlreadyExecutedTransaction();
         error UnknownFlag();
     
         error NotDao();
     
         error LayerZeroUnauthorizedEndpoint();
         error LayerZeroUnauthorizedCaller();
     
         error AlreadyAddedBridgeAgent();
         error UnrecognizedExecutor();
         error UnrecognizedPort();
         error UnrecognizedBridgeAgent();
         error UnrecognizedLocalBridgeAgent();
         error UnrecognizedBridgeAgentManager();
         error UnrecognizedRouter();
     
         error UnrecognizedUnderlyingAddress();
         error UnrecognizedLocalAddress();
     
         error SettlementRetryUnavailable();
         error SettlementRetryUnavailableUseCallout();
         error SettlementRedeemUnavailable();
         error SettlementRetrieveUnavailable();
         error NotSettlementOwner();
     
         error InsufficientBalanceForSettlement();
         error InsufficientGasForFees();
         error InvalidInputParams();
         error InvalidInputParamsLength();
     
         error CallerIsNotPool();
         error AmountsAreZero();
     }
     

//@audit  ,_params are not checked
168:     function callOut(
             address payable _refundee,
             address _recipient,
             uint16 _dstChainId,
             bytes memory _params,
             GasParams calldata _gParams
         ) external payable;
     
         /**
          * @notice External function to move assets from root chain to branch omnichain environment.
          *   @param _refundee the effective owner of the settlement this address receives excess gas deposited on source chain
          *                    for a cross-chain call and is allowed to redeeming assets after a failed settlement fallback.
          *                    This address' Virtual Account is also allowed.
          *   @param _recipient recipient of bridged tokens and any outstanding gas on the destination chain.
          *   @param _dstChainId chain to bridge to.
          *   @param _params parameters for function call on branch chain.
          *   @param _sParams settlement parameters for asset bridging to branch chains.
          *   @param _gParams Gas Parameters for cross-chain message.
          *   @param _hasFallbackToggled Flag to toggle fallback function.
          *
          */
         function callOutAndBridge(
             address payable _refundee,
             address _recipient,
             uint16 _dstChainId,
             bytes calldata _params,
             SettlementInput calldata _sParams,
             GasParams calldata _gParams,
             bool _hasFallbackToggled
         ) external payable;
     
         /**
          * @notice External function to move assets from branch chain to root omnichain environment.
          *   @param _refundee the effective owner of the settlement this address receives excess gas deposited on source chain
          *                    for a cross-chain call and is allowed to redeeming assets after a failed settlement fallback.
          *                    This address' Virtual Account is also allowed.
          *   @param _recipient recipient of bridged tokens.
          *   @param _dstChainId chain to bridge to.
          *   @param _params parameters for function call on branch chain.
          *   @param _sParams settlement parameters for asset bridging to branch chains.
          *   @param _gParams Gas Parameters for cross-chain message.
          *   @param _hasFallbackToggled Flag to toggle fallback function.
          *
          *
          */
         function callOutAndBridgeMultiple(
             address payable _refundee,
             address _recipient,
             uint16 _dstChainId,
             bytes calldata _params,
             SettlementMultipleInput calldata _sParams,
             GasParams calldata _gParams,
             bool _hasFallbackToggled
         ) external payable;
     
         /*///////////////////////////////////////////////////////////////
                             SETTLEMENT FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice Function to retry a user's Settlement balance.
          *   @param _settlementNonce Identifier for token settlement.
          *   @param _recipient recipient of bridged tokens and any outstanding gas on the destination chain.
          *   @param _params Calldata for function call in branch chain.
          *   @param _gParams Gas Parameters for cross-chain message.
          *   @param _hasFallbackToggled Flag to toggle fallback function.
          *
          */
         function retrySettlement(
             uint32 _settlementNonce,
             address _recipient,
             bytes calldata _params,
             GasParams calldata _gParams,
             bool _hasFallbackToggled
         ) external payable;
     
         /**
          * @notice Function that allows retrieval of failed Settlement's foricng fallback to be triggered.
          *   @param _settlementNonce Identifier for token settlement.
          *   @param _gParams Gas Parameters for cross-chain message.
          *
          */
         function retrieveSettlement(uint32 _settlementNonce, GasParams calldata _gParams) external payable;
     
         /**
          * @notice Function that allows redemption of failed Settlement's global tokens.
          *   @param _depositNonce Identifier for token deposit.
          *
          */
         function redeemSettlement(uint32 _depositNonce) external;
     
         /*///////////////////////////////////////////////////////////////
                         TOKEN MANAGEMENT FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice Function to move assets from branch chain to root omnichain environment.
          * @dev Called in response to Bridge Agent Executor.
          *   @param _recipient recipient of bridged token.
          *   @param _dParams Cross-Chain Deposit of Multiple Tokens Params.
          *   @param _srcChainId chain to bridge from.
          *
          */
         function bridgeIn(address _recipient, DepositParams memory _dParams, uint256 _srcChainId) external;
     
         /**
          * @notice Function to move assets from branch chain to root omnichain environment.
          * @dev Called in response to Bridge Agent Executor.
          *   @param _recipient recipient of bridged tokens.
          *   @param _dParams Cross-Chain Deposit of Multiple Tokens Params.
          *   @param _srcChainId chain to bridge from.
          *   @dev Since the input data is encodePacked we need to parse it:
          *     1. First byte is the number of assets to be bridged in. Equals length of all arrays.
          *     2. Next 4 bytes are the nonce of the deposit.
          *     3. Last 32 bytes after the token related information are the chain to bridge to.
          *     4. Token related information starts at index PARAMS_TKN_START is encoded as follows:
          *         1. N * 32 bytes for the hToken address.
          *         2. N * 32 bytes for the underlying token address.
          *         3. N * 32 bytes for the amount of hTokens to be bridged in.
          *         4. N * 32 bytes for the amount of underlying tokens to be bridged in.
          *     5. Each of the 4 token related arrays are of length N and start at the following indexes:
          *         1. PARAMS_TKN_START [hToken address has no offset from token information start].
          *         2. PARAMS_TKN_START + (PARAMS_ADDRESS_SIZE * N)
          *         3. PARAMS_TKN_START + (PARAMS_AMT_OFFSET * N)
          *         4. PARAMS_TKN_START + (PARAMS_DEPOSIT_OFFSET * N)
          *
          */
         function bridgeInMultiple(address _recipient, DepositMultipleParams calldata _dParams, uint256 _srcChainId)
             external;
     
         /*///////////////////////////////////////////////////////////////
                                 LAYER ZERO FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice External function to receive cross-chain messages from LayerZero Endpoint Contract without blocking.
          *   @param _endpoint address of the LayerZero Endpoint Contract.
          *   @param _srcChainId Chain ID of the sender.
          *   @param _srcAddress address path of the recipient + sender.
          *   @param _payload Calldata for function call.
          */
         function lzReceiveNonBlocking(
             address _endpoint,
             uint16 _srcChainId,
             bytes calldata _srcAddress,
             bytes calldata _payload
         ) external;
     
         /*///////////////////////////////////////////////////////////////
                                 ADMIN FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice Adds a new branch bridge agent to a given branch chainId
          *   @param _branchChainId chainId of the branch chain
          */
         function approveBranchBridgeAgent(uint256 _branchChainId) external;
     
         /**
          * @notice Updates the address of the branch bridge agent
          *   @param _newBranchBridgeAgent address of the new branch bridge agent
          *   @param _branchChainId chainId of the branch chain
          */
         function syncBranchBridgeAgent(address _newBranchBridgeAgent, uint256 _branchChainId) external;
     
         /*///////////////////////////////////////////////////////////////
                                  EVENTS
         //////////////////////////////////////////////////////////////*/
     
         event LogExecute(uint256 indexed depositNonce, uint256 indexed srcChainId);
         event LogFallback(uint256 indexed settlementNonce, uint256 indexed dstChainId);
     
         /*///////////////////////////////////////////////////////////////
                                 ERRORS
         //////////////////////////////////////////////////////////////*/
     
         error ExecutionFailure();
         error GasErrorOrRepeatedTx();
         error AlreadyExecutedTransaction();
         error UnknownFlag();
     
         error NotDao();
     
         error LayerZeroUnauthorizedEndpoint();
         error LayerZeroUnauthorizedCaller();
     
         error AlreadyAddedBridgeAgent();
         error UnrecognizedExecutor();
         error UnrecognizedPort();
         error UnrecognizedBridgeAgent();
         error UnrecognizedLocalBridgeAgent();
         error UnrecognizedBridgeAgentManager();
         error UnrecognizedRouter();
     
         error UnrecognizedUnderlyingAddress();
         error UnrecognizedLocalAddress();
     
         error SettlementRetryUnavailable();
         error SettlementRetryUnavailableUseCallout();
         error SettlementRedeemUnavailable();
         error SettlementRetrieveUnavailable();
         error NotSettlementOwner();
     
         error InsufficientBalanceForSettlement();
         error InsufficientGasForFees();
         error InvalidInputParams();
         error InvalidInputParamsLength();
     
         error CallerIsNotPool();
         error AmountsAreZero();
     }
     

//@audit  ,_params are not checked
189:     function callOutAndBridge(
             address payable _refundee,
             address _recipient,
             uint16 _dstChainId,
             bytes calldata _params,
             SettlementInput calldata _sParams,
             GasParams calldata _gParams,
             bool _hasFallbackToggled
         ) external payable;
     
         /**
          * @notice External function to move assets from branch chain to root omnichain environment.
          *   @param _refundee the effective owner of the settlement this address receives excess gas deposited on source chain
          *                    for a cross-chain call and is allowed to redeeming assets after a failed settlement fallback.
          *                    This address' Virtual Account is also allowed.
          *   @param _recipient recipient of bridged tokens.
          *   @param _dstChainId chain to bridge to.
          *   @param _params parameters for function call on branch chain.
          *   @param _sParams settlement parameters for asset bridging to branch chains.
          *   @param _gParams Gas Parameters for cross-chain message.
          *   @param _hasFallbackToggled Flag to toggle fallback function.
          *
          *
          */
         function callOutAndBridgeMultiple(
             address payable _refundee,
             address _recipient,
             uint16 _dstChainId,
             bytes calldata _params,
             SettlementMultipleInput calldata _sParams,
             GasParams calldata _gParams,
             bool _hasFallbackToggled
         ) external payable;
     
         /*///////////////////////////////////////////////////////////////
                             SETTLEMENT FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice Function to retry a user's Settlement balance.
          *   @param _settlementNonce Identifier for token settlement.
          *   @param _recipient recipient of bridged tokens and any outstanding gas on the destination chain.
          *   @param _params Calldata for function call in branch chain.
          *   @param _gParams Gas Parameters for cross-chain message.
          *   @param _hasFallbackToggled Flag to toggle fallback function.
          *
          */
         function retrySettlement(
             uint32 _settlementNonce,
             address _recipient,
             bytes calldata _params,
             GasParams calldata _gParams,
             bool _hasFallbackToggled
         ) external payable;
     
         /**
          * @notice Function that allows retrieval of failed Settlement's foricng fallback to be triggered.
          *   @param _settlementNonce Identifier for token settlement.
          *   @param _gParams Gas Parameters for cross-chain message.
          *
          */
         function retrieveSettlement(uint32 _settlementNonce, GasParams calldata _gParams) external payable;
     
         /**
          * @notice Function that allows redemption of failed Settlement's global tokens.
          *   @param _depositNonce Identifier for token deposit.
          *
          */
         function redeemSettlement(uint32 _depositNonce) external;
     
         /*///////////////////////////////////////////////////////////////
                         TOKEN MANAGEMENT FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice Function to move assets from branch chain to root omnichain environment.
          * @dev Called in response to Bridge Agent Executor.
          *   @param _recipient recipient of bridged token.
          *   @param _dParams Cross-Chain Deposit of Multiple Tokens Params.
          *   @param _srcChainId chain to bridge from.
          *
          */
         function bridgeIn(address _recipient, DepositParams memory _dParams, uint256 _srcChainId) external;
     
         /**
          * @notice Function to move assets from branch chain to root omnichain environment.
          * @dev Called in response to Bridge Agent Executor.
          *   @param _recipient recipient of bridged tokens.
          *   @param _dParams Cross-Chain Deposit of Multiple Tokens Params.
          *   @param _srcChainId chain to bridge from.
          *   @dev Since the input data is encodePacked we need to parse it:
          *     1. First byte is the number of assets to be bridged in. Equals length of all arrays.
          *     2. Next 4 bytes are the nonce of the deposit.
          *     3. Last 32 bytes after the token related information are the chain to bridge to.
          *     4. Token related information starts at index PARAMS_TKN_START is encoded as follows:
          *         1. N * 32 bytes for the hToken address.
          *         2. N * 32 bytes for the underlying token address.
          *         3. N * 32 bytes for the amount of hTokens to be bridged in.
          *         4. N * 32 bytes for the amount of underlying tokens to be bridged in.
          *     5. Each of the 4 token related arrays are of length N and start at the following indexes:
          *         1. PARAMS_TKN_START [hToken address has no offset from token information start].
          *         2. PARAMS_TKN_START + (PARAMS_ADDRESS_SIZE * N)
          *         3. PARAMS_TKN_START + (PARAMS_AMT_OFFSET * N)
          *         4. PARAMS_TKN_START + (PARAMS_DEPOSIT_OFFSET * N)
          *
          */
         function bridgeInMultiple(address _recipient, DepositMultipleParams calldata _dParams, uint256 _srcChainId)
             external;
     
         /*///////////////////////////////////////////////////////////////
                                 LAYER ZERO FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice External function to receive cross-chain messages from LayerZero Endpoint Contract without blocking.
          *   @param _endpoint address of the LayerZero Endpoint Contract.
          *   @param _srcChainId Chain ID of the sender.
          *   @param _srcAddress address path of the recipient + sender.
          *   @param _payload Calldata for function call.
          */
         function lzReceiveNonBlocking(
             address _endpoint,
             uint16 _srcChainId,
             bytes calldata _srcAddress,
             bytes calldata _payload
         ) external;
     
         /*///////////////////////////////////////////////////////////////
                                 ADMIN FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice Adds a new branch bridge agent to a given branch chainId
          *   @param _branchChainId chainId of the branch chain
          */
         function approveBranchBridgeAgent(uint256 _branchChainId) external;
     
         /**
          * @notice Updates the address of the branch bridge agent
          *   @param _newBranchBridgeAgent address of the new branch bridge agent
          *   @param _branchChainId chainId of the branch chain
          */
         function syncBranchBridgeAgent(address _newBranchBridgeAgent, uint256 _branchChainId) external;
     
         /*///////////////////////////////////////////////////////////////
                                  EVENTS
         //////////////////////////////////////////////////////////////*/
     
         event LogExecute(uint256 indexed depositNonce, uint256 indexed srcChainId);
         event LogFallback(uint256 indexed settlementNonce, uint256 indexed dstChainId);
     
         /*///////////////////////////////////////////////////////////////
                                 ERRORS
         //////////////////////////////////////////////////////////////*/
     
         error ExecutionFailure();
         error GasErrorOrRepeatedTx();
         error AlreadyExecutedTransaction();
         error UnknownFlag();
     
         error NotDao();
     
         error LayerZeroUnauthorizedEndpoint();
         error LayerZeroUnauthorizedCaller();
     
         error AlreadyAddedBridgeAgent();
         error UnrecognizedExecutor();
         error UnrecognizedPort();
         error UnrecognizedBridgeAgent();
         error UnrecognizedLocalBridgeAgent();
         error UnrecognizedBridgeAgentManager();
         error UnrecognizedRouter();
     
         error UnrecognizedUnderlyingAddress();
         error UnrecognizedLocalAddress();
     
         error SettlementRetryUnavailable();
         error SettlementRetryUnavailableUseCallout();
         error SettlementRedeemUnavailable();
         error SettlementRetrieveUnavailable();
         error NotSettlementOwner();
     
         error InsufficientBalanceForSettlement();
         error InsufficientGasForFees();
         error InvalidInputParams();
         error InvalidInputParamsLength();
     
         error CallerIsNotPool();
         error AmountsAreZero();
     }
     

//@audit  ,_params are not checked
213:     function callOutAndBridgeMultiple(
             address payable _refundee,
             address _recipient,
             uint16 _dstChainId,
             bytes calldata _params,
             SettlementMultipleInput calldata _sParams,
             GasParams calldata _gParams,
             bool _hasFallbackToggled
         ) external payable;
     
         /*///////////////////////////////////////////////////////////////
                             SETTLEMENT FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice Function to retry a user's Settlement balance.
          *   @param _settlementNonce Identifier for token settlement.
          *   @param _recipient recipient of bridged tokens and any outstanding gas on the destination chain.
          *   @param _params Calldata for function call in branch chain.
          *   @param _gParams Gas Parameters for cross-chain message.
          *   @param _hasFallbackToggled Flag to toggle fallback function.
          *
          */
         function retrySettlement(
             uint32 _settlementNonce,
             address _recipient,
             bytes calldata _params,
             GasParams calldata _gParams,
             bool _hasFallbackToggled
         ) external payable;
     
         /**
          * @notice Function that allows retrieval of failed Settlement's foricng fallback to be triggered.
          *   @param _settlementNonce Identifier for token settlement.
          *   @param _gParams Gas Parameters for cross-chain message.
          *
          */
         function retrieveSettlement(uint32 _settlementNonce, GasParams calldata _gParams) external payable;
     
         /**
          * @notice Function that allows redemption of failed Settlement's global tokens.
          *   @param _depositNonce Identifier for token deposit.
          *
          */
         function redeemSettlement(uint32 _depositNonce) external;
     
         /*///////////////////////////////////////////////////////////////
                         TOKEN MANAGEMENT FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice Function to move assets from branch chain to root omnichain environment.
          * @dev Called in response to Bridge Agent Executor.
          *   @param _recipient recipient of bridged token.
          *   @param _dParams Cross-Chain Deposit of Multiple Tokens Params.
          *   @param _srcChainId chain to bridge from.
          *
          */
         function bridgeIn(address _recipient, DepositParams memory _dParams, uint256 _srcChainId) external;
     
         /**
          * @notice Function to move assets from branch chain to root omnichain environment.
          * @dev Called in response to Bridge Agent Executor.
          *   @param _recipient recipient of bridged tokens.
          *   @param _dParams Cross-Chain Deposit of Multiple Tokens Params.
          *   @param _srcChainId chain to bridge from.
          *   @dev Since the input data is encodePacked we need to parse it:
          *     1. First byte is the number of assets to be bridged in. Equals length of all arrays.
          *     2. Next 4 bytes are the nonce of the deposit.
          *     3. Last 32 bytes after the token related information are the chain to bridge to.
          *     4. Token related information starts at index PARAMS_TKN_START is encoded as follows:
          *         1. N * 32 bytes for the hToken address.
          *         2. N * 32 bytes for the underlying token address.
          *         3. N * 32 bytes for the amount of hTokens to be bridged in.
          *         4. N * 32 bytes for the amount of underlying tokens to be bridged in.
          *     5. Each of the 4 token related arrays are of length N and start at the following indexes:
          *         1. PARAMS_TKN_START [hToken address has no offset from token information start].
          *         2. PARAMS_TKN_START + (PARAMS_ADDRESS_SIZE * N)
          *         3. PARAMS_TKN_START + (PARAMS_AMT_OFFSET * N)
          *         4. PARAMS_TKN_START + (PARAMS_DEPOSIT_OFFSET * N)
          *
          */
         function bridgeInMultiple(address _recipient, DepositMultipleParams calldata _dParams, uint256 _srcChainId)
             external;
     
         /*///////////////////////////////////////////////////////////////
                                 LAYER ZERO FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice External function to receive cross-chain messages from LayerZero Endpoint Contract without blocking.
          *   @param _endpoint address of the LayerZero Endpoint Contract.
          *   @param _srcChainId Chain ID of the sender.
          *   @param _srcAddress address path of the recipient + sender.
          *   @param _payload Calldata for function call.
          */
         function lzReceiveNonBlocking(
             address _endpoint,
             uint16 _srcChainId,
             bytes calldata _srcAddress,
             bytes calldata _payload
         ) external;
     
         /*///////////////////////////////////////////////////////////////
                                 ADMIN FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice Adds a new branch bridge agent to a given branch chainId
          *   @param _branchChainId chainId of the branch chain
          */
         function approveBranchBridgeAgent(uint256 _branchChainId) external;
     
         /**
          * @notice Updates the address of the branch bridge agent
          *   @param _newBranchBridgeAgent address of the new branch bridge agent
          *   @param _branchChainId chainId of the branch chain
          */
         function syncBranchBridgeAgent(address _newBranchBridgeAgent, uint256 _branchChainId) external;
     
         /*///////////////////////////////////////////////////////////////
                                  EVENTS
         //////////////////////////////////////////////////////////////*/
     
         event LogExecute(uint256 indexed depositNonce, uint256 indexed srcChainId);
         event LogFallback(uint256 indexed settlementNonce, uint256 indexed dstChainId);
     
         /*///////////////////////////////////////////////////////////////
                                 ERRORS
         //////////////////////////////////////////////////////////////*/
     
         error ExecutionFailure();
         error GasErrorOrRepeatedTx();
         error AlreadyExecutedTransaction();
         error UnknownFlag();
     
         error NotDao();
     
         error LayerZeroUnauthorizedEndpoint();
         error LayerZeroUnauthorizedCaller();
     
         error AlreadyAddedBridgeAgent();
         error UnrecognizedExecutor();
         error UnrecognizedPort();
         error UnrecognizedBridgeAgent();
         error UnrecognizedLocalBridgeAgent();
         error UnrecognizedBridgeAgentManager();
         error UnrecognizedRouter();
     
         error UnrecognizedUnderlyingAddress();
         error UnrecognizedLocalAddress();
     
         error SettlementRetryUnavailable();
         error SettlementRetryUnavailableUseCallout();
         error SettlementRedeemUnavailable();
         error SettlementRetrieveUnavailable();
         error NotSettlementOwner();
     
         error InsufficientBalanceForSettlement();
         error InsufficientGasForFees();
         error InvalidInputParams();
         error InvalidInputParamsLength();
     
         error CallerIsNotPool();
         error AmountsAreZero();
     }
     

//@audit  ,_params are not checked
236:     function retrySettlement(
             uint32 _settlementNonce,
             address _recipient,
             bytes calldata _params,
             GasParams calldata _gParams,
             bool _hasFallbackToggled
         ) external payable;
     
         /**
          * @notice Function that allows retrieval of failed Settlement's foricng fallback to be triggered.
          *   @param _settlementNonce Identifier for token settlement.
          *   @param _gParams Gas Parameters for cross-chain message.
          *
          */
         function retrieveSettlement(uint32 _settlementNonce, GasParams calldata _gParams) external payable;
     
         /**
          * @notice Function that allows redemption of failed Settlement's global tokens.
          *   @param _depositNonce Identifier for token deposit.
          *
          */
         function redeemSettlement(uint32 _depositNonce) external;
     
         /*///////////////////////////////////////////////////////////////
                         TOKEN MANAGEMENT FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice Function to move assets from branch chain to root omnichain environment.
          * @dev Called in response to Bridge Agent Executor.
          *   @param _recipient recipient of bridged token.
          *   @param _dParams Cross-Chain Deposit of Multiple Tokens Params.
          *   @param _srcChainId chain to bridge from.
          *
          */
         function bridgeIn(address _recipient, DepositParams memory _dParams, uint256 _srcChainId) external;
     
         /**
          * @notice Function to move assets from branch chain to root omnichain environment.
          * @dev Called in response to Bridge Agent Executor.
          *   @param _recipient recipient of bridged tokens.
          *   @param _dParams Cross-Chain Deposit of Multiple Tokens Params.
          *   @param _srcChainId chain to bridge from.
          *   @dev Since the input data is encodePacked we need to parse it:
          *     1. First byte is the number of assets to be bridged in. Equals length of all arrays.
          *     2. Next 4 bytes are the nonce of the deposit.
          *     3. Last 32 bytes after the token related information are the chain to bridge to.
          *     4. Token related information starts at index PARAMS_TKN_START is encoded as follows:
          *         1. N * 32 bytes for the hToken address.
          *         2. N * 32 bytes for the underlying token address.
          *         3. N * 32 bytes for the amount of hTokens to be bridged in.
          *         4. N * 32 bytes for the amount of underlying tokens to be bridged in.
          *     5. Each of the 4 token related arrays are of length N and start at the following indexes:
          *         1. PARAMS_TKN_START [hToken address has no offset from token information start].
          *         2. PARAMS_TKN_START + (PARAMS_ADDRESS_SIZE * N)
          *         3. PARAMS_TKN_START + (PARAMS_AMT_OFFSET * N)
          *         4. PARAMS_TKN_START + (PARAMS_DEPOSIT_OFFSET * N)
          *
          */
         function bridgeInMultiple(address _recipient, DepositMultipleParams calldata _dParams, uint256 _srcChainId)
             external;
     
         /*///////////////////////////////////////////////////////////////
                                 LAYER ZERO FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice External function to receive cross-chain messages from LayerZero Endpoint Contract without blocking.
          *   @param _endpoint address of the LayerZero Endpoint Contract.
          *   @param _srcChainId Chain ID of the sender.
          *   @param _srcAddress address path of the recipient + sender.
          *   @param _payload Calldata for function call.
          */
         function lzReceiveNonBlocking(
             address _endpoint,
             uint16 _srcChainId,
             bytes calldata _srcAddress,
             bytes calldata _payload
         ) external;
     
         /*///////////////////////////////////////////////////////////////
                                 ADMIN FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice Adds a new branch bridge agent to a given branch chainId
          *   @param _branchChainId chainId of the branch chain
          */
         function approveBranchBridgeAgent(uint256 _branchChainId) external;
     
         /**
          * @notice Updates the address of the branch bridge agent
          *   @param _newBranchBridgeAgent address of the new branch bridge agent
          *   @param _branchChainId chainId of the branch chain
          */
         function syncBranchBridgeAgent(address _newBranchBridgeAgent, uint256 _branchChainId) external;
     
         /*///////////////////////////////////////////////////////////////
                                  EVENTS
         //////////////////////////////////////////////////////////////*/
     
         event LogExecute(uint256 indexed depositNonce, uint256 indexed srcChainId);
         event LogFallback(uint256 indexed settlementNonce, uint256 indexed dstChainId);
     
         /*///////////////////////////////////////////////////////////////
                                 ERRORS
         //////////////////////////////////////////////////////////////*/
     
         error ExecutionFailure();
         error GasErrorOrRepeatedTx();
         error AlreadyExecutedTransaction();
         error UnknownFlag();
     
         error NotDao();
     
         error LayerZeroUnauthorizedEndpoint();
         error LayerZeroUnauthorizedCaller();
     
         error AlreadyAddedBridgeAgent();
         error UnrecognizedExecutor();
         error UnrecognizedPort();
         error UnrecognizedBridgeAgent();
         error UnrecognizedLocalBridgeAgent();
         error UnrecognizedBridgeAgentManager();
         error UnrecognizedRouter();
     
         error UnrecognizedUnderlyingAddress();
         error UnrecognizedLocalAddress();
     
         error SettlementRetryUnavailable();
         error SettlementRetryUnavailableUseCallout();
         error SettlementRedeemUnavailable();
         error SettlementRetrieveUnavailable();
         error NotSettlementOwner();
     
         error InsufficientBalanceForSettlement();
         error InsufficientGasForFees();
         error InvalidInputParams();
         error InvalidInputParamsLength();
     
         error CallerIsNotPool();
         error AmountsAreZero();
     }
     

//@audit  ,_srcAddress ,_payload are not checked
309:     function lzReceiveNonBlocking(
             address _endpoint,
             uint16 _srcChainId,
             bytes calldata _srcAddress,
             bytes calldata _payload
         ) external;
     
         /*///////////////////////////////////////////////////////////////
                                 ADMIN FUNCTIONS
         //////////////////////////////////////////////////////////////*/
     
         /**
          * @notice Adds a new branch bridge agent to a given branch chainId
          *   @param _branchChainId chainId of the branch chain
          */
         function approveBranchBridgeAgent(uint256 _branchChainId) external;
     
         /**
          * @notice Updates the address of the branch bridge agent
          *   @param _newBranchBridgeAgent address of the new branch bridge agent
          *   @param _branchChainId chainId of the branch chain
          */
         function syncBranchBridgeAgent(address _newBranchBridgeAgent, uint256 _branchChainId) external;
     
         /*///////////////////////////////////////////////////////////////
                                  EVENTS
         //////////////////////////////////////////////////////////////*/
     
         event LogExecute(uint256 indexed depositNonce, uint256 indexed srcChainId);
         event LogFallback(uint256 indexed settlementNonce, uint256 indexed dstChainId);
     
         /*///////////////////////////////////////////////////////////////
                                 ERRORS
         //////////////////////////////////////////////////////////////*/
     
         error ExecutionFailure();
         error GasErrorOrRepeatedTx();
         error AlreadyExecutedTransaction();
         error UnknownFlag();
     
         error NotDao();
     
         error LayerZeroUnauthorizedEndpoint();
         error LayerZeroUnauthorizedCaller();
     
         error AlreadyAddedBridgeAgent();
         error UnrecognizedExecutor();
         error UnrecognizedPort();
         error UnrecognizedBridgeAgent();
         error UnrecognizedLocalBridgeAgent();
         error UnrecognizedBridgeAgentManager();
         error UnrecognizedRouter();
     
         error UnrecognizedUnderlyingAddress();
         error UnrecognizedLocalAddress();
     
         error SettlementRetryUnavailable();
         error SettlementRetryUnavailableUseCallout();
         error SettlementRedeemUnavailable();
         error SettlementRetrieveUnavailable();
         error NotSettlementOwner();
     
         error InsufficientBalanceForSettlement();
         error InsufficientGasForFees();
         error InvalidInputParams();
         error InvalidInputParamsLength();
     
         error CallerIsNotPool();
         error AmountsAreZero();
     }
     

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/IRootBridgeAgent.sol#L309-L379)

```solidity
File: src/interfaces/IRootRouter.sol

//@audit  ,params are not checked
25:     function executeResponse(bytes memory params, uint16 srcChainId) external payable;
    
        /**
         *     @notice Function responsible of executing a crosschain request without any deposit.
         *     @param params data received from messaging layer.
         *     @param srcChainId chain where the request originated from.
         *
         */
        function execute(bytes memory params, uint16 srcChainId) external payable;
    
        /**
         *   @notice Function responsible of executing a crosschain request which contains cross-chain deposit information attached.
         *   @param params execution data received from messaging layer.
         *   @param dParams cross-chain deposit information.
         *   @param srcChainId chain where the request originated from.
         *
         */
        function executeDepositSingle(bytes memory params, DepositParams memory dParams, uint16 srcChainId)
            external
            payable;
    
        /**
         *   @notice Function responsible of executing a crosschain request which contains cross-chain deposit information for multiple assets attached.
         *   @param params execution data received from messaging layer.
         *   @param dParams cross-chain multiple deposit information.
         *   @param srcChainId chain where the request originated from.
         *
         */
        function executeDepositMultiple(bytes memory params, DepositMultipleParams memory dParams, uint16 srcChainId)
            external
            payable;
    
        /**
         * @notice Function responsible of executing a crosschain request with msg.sender without any deposit.
         * @param params execution data received from messaging layer.
         * @param userAccount user account address.
         * @param srcChainId chain where the request originated from.
         */
        function executeSigned(bytes memory params, address userAccount, uint16 srcChainId) external payable;
    
        /**
         * @notice Function responsible of executing a crosschain request which contains cross-chain deposit information and msg.sender attached.
         * @param params execution data received from messaging layer.
         * @param dParams cross-chain deposit information.
         * @param userAccount user account address.
         * @param srcChainId chain where the request originated from.
         */
        function executeSignedDepositSingle(
            bytes memory params,
            DepositParams memory dParams,
            address userAccount,
            uint16 srcChainId
        ) external payable;
    
        /**
         * @notice Function responsible of executing a crosschain request which contains cross-chain deposit information for multiple assets and msg.sender attached.
         * @param params execution data received from messaging layer.
         * @param dParams cross-chain multiple deposit information.
         * @param userAccount user account address.
         * @param srcChainId chain where the request originated from.
         */
        function executeSignedDepositMultiple(
            bytes memory params,
            DepositMultipleParams memory dParams,
            address userAccount,
            uint16 srcChainId
        ) external payable;
    
        /*///////////////////////////////////////////////////////////////
                                 ERRORS
        //////////////////////////////////////////////////////////////*/
    
        error UnrecognizedFunctionId();
        error UnrecognizedBridgeAgentExecutor();
    }
    

//@audit  ,params are not checked
33:     function execute(bytes memory params, uint16 srcChainId) external payable;
    
        /**
         *   @notice Function responsible of executing a crosschain request which contains cross-chain deposit information attached.
         *   @param params execution data received from messaging layer.
         *   @param dParams cross-chain deposit information.
         *   @param srcChainId chain where the request originated from.
         *
         */
        function executeDepositSingle(bytes memory params, DepositParams memory dParams, uint16 srcChainId)
            external
            payable;
    
        /**
         *   @notice Function responsible of executing a crosschain request which contains cross-chain deposit information for multiple assets attached.
         *   @param params execution data received from messaging layer.
         *   @param dParams cross-chain multiple deposit information.
         *   @param srcChainId chain where the request originated from.
         *
         */
        function executeDepositMultiple(bytes memory params, DepositMultipleParams memory dParams, uint16 srcChainId)
            external
            payable;
    
        /**
         * @notice Function responsible of executing a crosschain request with msg.sender without any deposit.
         * @param params execution data received from messaging layer.
         * @param userAccount user account address.
         * @param srcChainId chain where the request originated from.
         */
        function executeSigned(bytes memory params, address userAccount, uint16 srcChainId) external payable;
    
        /**
         * @notice Function responsible of executing a crosschain request which contains cross-chain deposit information and msg.sender attached.
         * @param params execution data received from messaging layer.
         * @param dParams cross-chain deposit information.
         * @param userAccount user account address.
         * @param srcChainId chain where the request originated from.
         */
        function executeSignedDepositSingle(
            bytes memory params,
            DepositParams memory dParams,
            address userAccount,
            uint16 srcChainId
        ) external payable;
    
        /**
         * @notice Function responsible of executing a crosschain request which contains cross-chain deposit information for multiple assets and msg.sender attached.
         * @param params execution data received from messaging layer.
         * @param dParams cross-chain multiple deposit information.
         * @param userAccount user account address.
         * @param srcChainId chain where the request originated from.
         */
        function executeSignedDepositMultiple(
            bytes memory params,
            DepositMultipleParams memory dParams,
            address userAccount,
            uint16 srcChainId
        ) external payable;
    
        /*///////////////////////////////////////////////////////////////
                                 ERRORS
        //////////////////////////////////////////////////////////////*/
    
        error UnrecognizedFunctionId();
        error UnrecognizedBridgeAgentExecutor();
    }
    

//@audit  ,params are not checked
42:     function executeDepositSingle(bytes memory params, DepositParams memory dParams, uint16 srcChainId)
            external
            payable;
    
        /**
         *   @notice Function responsible of executing a crosschain request which contains cross-chain deposit information for multiple assets attached.
         *   @param params execution data received from messaging layer.
         *   @param dParams cross-chain multiple deposit information.
         *   @param srcChainId chain where the request originated from.
         *
         */
        function executeDepositMultiple(bytes memory params, DepositMultipleParams memory dParams, uint16 srcChainId)
            external
            payable;
    
        /**
         * @notice Function responsible of executing a crosschain request with msg.sender without any deposit.
         * @param params execution data received from messaging layer.
         * @param userAccount user account address.
         * @param srcChainId chain where the request originated from.
         */
        function executeSigned(bytes memory params, address userAccount, uint16 srcChainId) external payable;
    
        /**
         * @notice Function responsible of executing a crosschain request which contains cross-chain deposit information and msg.sender attached.
         * @param params execution data received from messaging layer.
         * @param dParams cross-chain deposit information.
         * @param userAccount user account address.
         * @param srcChainId chain where the request originated from.
         */
        function executeSignedDepositSingle(
            bytes memory params,
            DepositParams memory dParams,
            address userAccount,
            uint16 srcChainId
        ) external payable;
    
        /**
         * @notice Function responsible of executing a crosschain request which contains cross-chain deposit information for multiple assets and msg.sender attached.
         * @param params execution data received from messaging layer.
         * @param dParams cross-chain multiple deposit information.
         * @param userAccount user account address.
         * @param srcChainId chain where the request originated from.
         */
        function executeSignedDepositMultiple(
            bytes memory params,
            DepositMultipleParams memory dParams,
            address userAccount,
            uint16 srcChainId
        ) external payable;
    
        /*///////////////////////////////////////////////////////////////
                                 ERRORS
        //////////////////////////////////////////////////////////////*/
    
        error UnrecognizedFunctionId();
        error UnrecognizedBridgeAgentExecutor();
    }
    

//@audit  ,params are not checked
53:     function executeDepositMultiple(bytes memory params, DepositMultipleParams memory dParams, uint16 srcChainId)
            external
            payable;
    
        /**
         * @notice Function responsible of executing a crosschain request with msg.sender without any deposit.
         * @param params execution data received from messaging layer.
         * @param userAccount user account address.
         * @param srcChainId chain where the request originated from.
         */
        function executeSigned(bytes memory params, address userAccount, uint16 srcChainId) external payable;
    
        /**
         * @notice Function responsible of executing a crosschain request which contains cross-chain deposit information and msg.sender attached.
         * @param params execution data received from messaging layer.
         * @param dParams cross-chain deposit information.
         * @param userAccount user account address.
         * @param srcChainId chain where the request originated from.
         */
        function executeSignedDepositSingle(
            bytes memory params,
            DepositParams memory dParams,
            address userAccount,
            uint16 srcChainId
        ) external payable;
    
        /**
         * @notice Function responsible of executing a crosschain request which contains cross-chain deposit information for multiple assets and msg.sender attached.
         * @param params execution data received from messaging layer.
         * @param dParams cross-chain multiple deposit information.
         * @param userAccount user account address.
         * @param srcChainId chain where the request originated from.
         */
        function executeSignedDepositMultiple(
            bytes memory params,
            DepositMultipleParams memory dParams,
            address userAccount,
            uint16 srcChainId
        ) external payable;
    
        /*///////////////////////////////////////////////////////////////
                                 ERRORS
        //////////////////////////////////////////////////////////////*/
    
        error UnrecognizedFunctionId();
        error UnrecognizedBridgeAgentExecutor();
    }
    

//@audit  ,params are not checked
63:     function executeSigned(bytes memory params, address userAccount, uint16 srcChainId) external payable;
    
        /**
         * @notice Function responsible of executing a crosschain request which contains cross-chain deposit information and msg.sender attached.
         * @param params execution data received from messaging layer.
         * @param dParams cross-chain deposit information.
         * @param userAccount user account address.
         * @param srcChainId chain where the request originated from.
         */
        function executeSignedDepositSingle(
            bytes memory params,
            DepositParams memory dParams,
            address userAccount,
            uint16 srcChainId
        ) external payable;
    
        /**
         * @notice Function responsible of executing a crosschain request which contains cross-chain deposit information for multiple assets and msg.sender attached.
         * @param params execution data received from messaging layer.
         * @param dParams cross-chain multiple deposit information.
         * @param userAccount user account address.
         * @param srcChainId chain where the request originated from.
         */
        function executeSignedDepositMultiple(
            bytes memory params,
            DepositMultipleParams memory dParams,
            address userAccount,
            uint16 srcChainId
        ) external payable;
    
        /*///////////////////////////////////////////////////////////////
                                 ERRORS
        //////////////////////////////////////////////////////////////*/
    
        error UnrecognizedFunctionId();
        error UnrecognizedBridgeAgentExecutor();
    }
    

//@audit  ,params are not checked
72:     function executeSignedDepositSingle(
            bytes memory params,
            DepositParams memory dParams,
            address userAccount,
            uint16 srcChainId
        ) external payable;
    
        /**
         * @notice Function responsible of executing a crosschain request which contains cross-chain deposit information for multiple assets and msg.sender attached.
         * @param params execution data received from messaging layer.
         * @param dParams cross-chain multiple deposit information.
         * @param userAccount user account address.
         * @param srcChainId chain where the request originated from.
         */
        function executeSignedDepositMultiple(
            bytes memory params,
            DepositMultipleParams memory dParams,
            address userAccount,
            uint16 srcChainId
        ) external payable;
    
        /*///////////////////////////////////////////////////////////////
                                 ERRORS
        //////////////////////////////////////////////////////////////*/
    
        error UnrecognizedFunctionId();
        error UnrecognizedBridgeAgentExecutor();
    }
    

//@audit  ,params are not checked
86:     function executeSignedDepositMultiple(
            bytes memory params,
            DepositMultipleParams memory dParams,
            address userAccount,
            uint16 srcChainId
        ) external payable;
    
        /*///////////////////////////////////////////////////////////////
                                 ERRORS
        //////////////////////////////////////////////////////////////*/
    
        error UnrecognizedFunctionId();
        error UnrecognizedBridgeAgentExecutor();
    }
    

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/IRootRouter.sol#L86-L100)

</details>

### <a name="NC-7-1694440518"></a>[NC-7] High cyclomatic complexity
Consider breaking down these blocks into more manageable units, by splitting things into utility functions, by reducing nesting, and by using early returns

*Instances (6)*:
<details><summary> see instances </summary>

```solidity
File: src/BranchBridgeAgent.sol

343:     function retryDeposit(
             bool _isSigned,
             uint32 _depositNonce,
             bytes calldata _params,
             GasParams calldata _gParams,
             bool _hasFallbackToggled
         ) external payable override lock {
             // Get Settlement Reference
             Deposit storage deposit = getDeposit[_depositNonce];
     
             //Check if deposit belongs to message sender
             if (deposit.owner != msg.sender) revert NotDepositOwner();
     
             //Encode Data for cross-chain call.
             bytes memory payload;
     
             if (uint8(deposit.hTokens.length) == 1) {
                 if (_isSigned) {
                     //Pack new Data
                     payload = abi.encodePacked(
                         _hasFallbackToggled ? bytes1(0x85) : bytes1(0x05),
                         msg.sender,
                         _depositNonce,
                         deposit.hTokens[0],
                         deposit.tokens[0],
                         deposit.amounts[0],
                         deposit.deposits[0],
                         _params
                     );
                 } else {
                     payload = abi.encodePacked(
                         bytes1(0x02),
                         _depositNonce,
                         deposit.hTokens[0],
                         deposit.tokens[0],
                         deposit.amounts[0],
                         deposit.deposits[0],
                         _params
                     );
                 }
             } else if (uint8(deposit.hTokens.length) > 1) {
                 if (_isSigned) {
                     //Pack new Data
                     payload = abi.encodePacked(
                         _hasFallbackToggled ? bytes1(0x86) : bytes1(0x06),
                         msg.sender,
                         uint8(deposit.hTokens.length),
                         _depositNonce,
                         deposit.hTokens,
                         deposit.tokens,
                         deposit.amounts,
                         deposit.deposits,
                         _params
                     );
                 } else {
                     payload = abi.encodePacked(
                         bytes1(0x03),
                         uint8(deposit.hTokens.length),
                         _depositNonce,
                         deposit.hTokens,
                         deposit.tokens,
                         deposit.amounts,
                         deposit.deposits,
                         _params
                     );
                 }
             }
     
             // Check if payload is empty
             if (payload.length == 0) revert DepositRetryUnavailableUseCallout();
     
             // Ensure success Status
             deposit.status = STATUS_SUCCESS;
     
             // Perform Call
             _performCall(payable(msg.sender), payload, _gParams);
         }

587:     function lzReceiveNonBlocking(address _endpoint, bytes calldata _srcAddress, bytes calldata _payload)
             public
             override
             requiresEndpoint(_endpoint, _srcAddress)
         {
             //Save Action Flag
             bytes1 flag = _payload[0] & 0x7F;
     
             // Save settlement nonce
             uint32 nonce;
     
             // DEPOSIT FLAG: 0 (No settlement)
             if (flag == 0x00) {
                 // Get Settlement Nonce
                 nonce = uint32(bytes4(_payload[PARAMS_START_SIGNED:PARAMS_TKN_START_SIGNED]));
     
                 //Check if tx has already been executed
                 if (executionState[nonce] != STATUS_READY) revert AlreadyExecutedTransaction();
     
                 //Try to execute the remote request
                 //Flag 0 - BranchBridgeAgentExecutor(bridgeAgentExecutorAddress).executeNoSettlement(localRouterAddress, _payload)
                 _execute(
                     nonce,
                     abi.encodeWithSelector(
                         BranchBridgeAgentExecutor.executeNoSettlement.selector, localRouterAddress, _payload
                     )
                 );
     
                 // DEPOSIT FLAG: 1 (Single Asset Settlement)
             } else if (flag == 0x01) {
                 // Parse recipient
                 address payable recipient = payable(address(uint160(bytes20(_payload[PARAMS_START:PARAMS_START_SIGNED]))));
     
                 // Parse Settlement Nonce
                 nonce = uint32(bytes4(_payload[PARAMS_START_SIGNED:PARAMS_TKN_START_SIGNED]));
     
                 //Check if tx has already been executed
                 if (executionState[nonce] != STATUS_READY) revert AlreadyExecutedTransaction();
     
                 //Try to execute the remote request
                 //Flag 1 - BranchBridgeAgentExecutor(bridgeAgentExecutorAddress).executeWithSettlement(recipient, localRouterAddress, _payload)
                 _execute(
                     _payload[0] == 0x81,
                     nonce,
                     recipient,
                     abi.encodeWithSelector(
                         BranchBridgeAgentExecutor.executeWithSettlement.selector, recipient, localRouterAddress, _payload
                     )
                 );
     
                 // DEPOSIT FLAG: 2 (Multiple Settlement)
             } else if (flag == 0x02) {
                 // Parse recipient
                 address payable recipient = payable(address(uint160(bytes20(_payload[PARAMS_START:PARAMS_START_SIGNED]))));
     
                 // Parse deposit nonce
                 nonce = uint32(bytes4(_payload[22:26]));
     
                 //Check if tx has already been executed
                 if (executionState[nonce] != STATUS_READY) revert AlreadyExecutedTransaction();
     
                 //Try to execute remote request
                 // Flag 2 - BranchBridgeAgentExecutor(bridgeAgentExecutorAddress).executeWithSettlementMultiple(recipient, localRouterAddress, _payload)
                 _execute(
                     _payload[0] == 0x82,
                     nonce,
                     recipient,
                     abi.encodeWithSelector(
                         BranchBridgeAgentExecutor.executeWithSettlementMultiple.selector,
                         recipient,
                         localRouterAddress,
                         _payload
                     )
                 );
     
                 //DEPOSIT FLAG: 3 (Retrieve Settlement)
             } else if (flag == 0x03) {
                 // Parse recipient
                 address payable recipient = payable(address(uint160(bytes20(_payload[PARAMS_START:PARAMS_START_SIGNED]))));
     
                 //Get nonce
                 nonce = uint32(bytes4(_payload[PARAMS_START_SIGNED:PARAMS_TKN_START_SIGNED]));
     
                 //Check if settlement is in retrieve mode
                 if (executionState[nonce] == STATUS_DONE) {
                     revert AlreadyExecutedTransaction();
                 } else {
                     //Set settlement to retrieve mode, if not already set.
                     if (executionState[nonce] == STATUS_READY) executionState[nonce] = STATUS_RETRIEVE;
                     //Trigger fallback/Retry failed fallback
                     _performFallbackCall(recipient, nonce);
                 }
     
                 //DEPOSIT FLAG: 4 (Fallback)
             } else if (flag == 0x04) {
                 //Get nonce
                 nonce = uint32(bytes4(_payload[PARAMS_START:PARAMS_TKN_START]));
     
                 // Reopen Deposit for redemption
                 getDeposit[nonce].status = STATUS_FAILED;
     
                 // Emit Fallback Event
                 emit LogFallback(nonce);
     
                 // Return to prevent unnecessary logic/emits
                 return;
     
                 //Unrecognized Function Selector
             } else {
                 revert UnknownFlag();
             }
     
             // Emit Execution Event
             emit LogExecute(nonce);
         }

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L587-L701)

```solidity
File: src/CoreBranchRouter.sol

86:     function executeNoSettlement(bytes calldata _params) external payable virtual override requiresAgentExecutor {
            /// _receiveAddGlobalToken
            if (_params[0] == 0x01) {
                (
                    address globalAddress,
                    string memory name,
                    string memory symbol,
                    uint8 decimals,
                    address refundee,
                    GasParams memory gParams
                ) = abi.decode(_params[1:], (address, string, string, uint8, address, GasParams));
    
                _receiveAddGlobalToken(globalAddress, name, symbol, decimals, refundee, gParams);
                /// _receiveAddBridgeAgent
            } else if (_params[0] == 0x02) {
                (
                    address newBranchRouter,
                    address branchBridgeAgentFactory,
                    address rootBridgeAgent,
                    address rootBridgeAgentFactory,
                    address refundee,
                    GasParams memory gParams
                ) = abi.decode(_params[1:], (address, address, address, address, address, GasParams));
    
                _receiveAddBridgeAgent(
                    newBranchRouter, branchBridgeAgentFactory, rootBridgeAgent, rootBridgeAgentFactory, refundee, gParams
                );
    
                /// _toggleBranchBridgeAgentFactory
            } else if (_params[0] == 0x03) {
                (address bridgeAgentFactoryAddress) = abi.decode(_params[1:], (address));
    
                _toggleBranchBridgeAgentFactory(bridgeAgentFactoryAddress);
    
                /// _removeBranchBridgeAgent
            } else if (_params[0] == 0x04) {
                (address branchBridgeAgent) = abi.decode(_params[1:], (address));
    
                _removeBranchBridgeAgent(branchBridgeAgent);
    
                /// _manageStrategyToken
            } else if (_params[0] == 0x05) {
                (address underlyingToken, uint256 minimumReservesRatio) = abi.decode(_params[1:], (address, uint256));
    
                _manageStrategyToken(underlyingToken, minimumReservesRatio);
    
                /// _managePortStrategy
            } else if (_params[0] == 0x06) {
                (address portStrategy, address underlyingToken, uint256 dailyManagementLimit, bool isUpdateDailyLimit) =
                    abi.decode(_params[1:], (address, address, uint256, bool));
    
                _managePortStrategy(portStrategy, underlyingToken, dailyManagementLimit, isUpdateDailyLimit);
    
                /// _setCoreBranchRouter
            } else if (_params[0] == 0x07) {
                (address coreBranchRouter, address coreBranchBridgeAgent) = abi.decode(_params[1:], (address, address));
    
                IPort(localPortAddress).setCoreBranchRouter(coreBranchRouter, coreBranchBridgeAgent);
    
                /// Unrecognized Function Selector
            } else {
                revert UnrecognizedFunctionId();
            }
        }

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L86-L149)

```solidity
File: src/RootBridgeAgent.sol

434:     function lzReceiveNonBlocking(
             address _endpoint,
             uint16 _srcChainId,
             bytes calldata _srcAddress,
             bytes calldata _payload
         ) public override requiresEndpoint(_endpoint, _srcChainId, _srcAddress) {
             // Deposit Nonce
             uint32 nonce;
     
             // DEPOSIT FLAG: 0 (System request / response)
             if (_payload[0] == 0x00) {
                 // Parse deposit nonce
                 nonce = uint32(bytes4(_payload[PARAMS_START:PARAMS_TKN_START]));
     
                 // Check if tx has already been executed
                 if (executionState[_srcChainId][nonce] != STATUS_READY) {
                     revert AlreadyExecutedTransaction();
                 }
     
                 // Avoid stack too deep
                 uint16 srcChainId = _srcChainId;
     
                 // Try to execute remote request
                 // Flag 0 - RootBridgeAgentExecutor(bridgeAgentExecutorAddress).executeSystemRequest(_localRouterAddress, _payload, _srcChainId)
                 _execute(
                     nonce,
                     abi.encodeWithSelector(
                         RootBridgeAgentExecutor.executeSystemRequest.selector, localRouterAddress, _payload, srcChainId
                     ),
                     srcChainId
                 );
     
                 // DEPOSIT FLAG: 1 (Call without Deposit)
             } else if (_payload[0] == 0x01) {
                 // Parse Deposit Nonce
                 nonce = uint32(bytes4(_payload[PARAMS_START:PARAMS_TKN_START]));
     
                 // Check if tx has already been executed
                 if (executionState[_srcChainId][nonce] != STATUS_READY) {
                     revert AlreadyExecutedTransaction();
                 }
     
                 // Avoid stack too deep
                 uint16 srcChainId = _srcChainId;
     
                 // Try to execute remote request
                 // Flag 1 - RootBridgeAgentExecutor(bridgeAgentExecutorAddress).executeNoDeposit(localRouterAddress, payload, _srcChainId)
                 _execute(
                     nonce,
                     abi.encodeWithSelector(
                         RootBridgeAgentExecutor.executeNoDeposit.selector, localRouterAddress, _payload, srcChainId
                     ),
                     srcChainId
                 );
     
                 // DEPOSIT FLAG: 2 (Call with Deposit)
             } else if (_payload[0] == 0x02) {
                 //Parse Deposit Nonce
                 nonce = uint32(bytes4(_payload[PARAMS_START:PARAMS_TKN_START]));
     
                 //Check if tx has already been executed
                 if (executionState[_srcChainId][nonce] != STATUS_READY) {
                     revert AlreadyExecutedTransaction();
                 }
     
                 // Avoid stack too deep
                 uint16 srcChainId = _srcChainId;
     
                 // Try to execute remote request
                 // Flag 2 - RootBridgeAgentExecutor(bridgeAgentExecutorAddress).executeWithDeposit(localRouterAddress, _payload, _srcChainId)
                 _execute(
                     nonce,
                     abi.encodeWithSelector(
                         RootBridgeAgentExecutor.executeWithDeposit.selector, localRouterAddress, _payload, srcChainId
                     ),
                     srcChainId
                 );
     
                 // DEPOSIT FLAG: 3 (Call with multiple asset Deposit)
             } else if (_payload[0] == 0x03) {
                 // Parse deposit nonce
                 nonce = uint32(bytes4(_payload[2:6]));
     
                 // Check if tx has already been executed
                 if (executionState[_srcChainId][nonce] != STATUS_READY) {
                     revert AlreadyExecutedTransaction();
                 }
     
                 // Avoid stack too deep
                 uint16 srcChainId = _srcChainId;
     
                 // Try to execute remote request
                 // Flag 3 - RootBridgeAgentExecutor(bridgeAgentExecutorAddress).executeWithDepositMultiple(localRouterAddress, _payload, _srcChainId)
                 _execute(
                     nonce,
                     abi.encodeWithSelector(
                         RootBridgeAgentExecutor.executeWithDepositMultiple.selector,
                         localRouterAddress,
                         _payload,
                         srcChainId
                     ),
                     srcChainId
                 );
     
                 // DEPOSIT FLAG: 4 (Call without Deposit + msg.sender)
             } else if (_payload[0] == 0x04) {
                 // Parse deposit nonce
                 nonce = uint32(bytes4(_payload[PARAMS_START_SIGNED:PARAMS_TKN_START_SIGNED]));
     
                 //Check if tx has already been executed
                 if (executionState[_srcChainId][nonce] != STATUS_READY) {
                     revert AlreadyExecutedTransaction();
                 }
     
                 // Get User Virtual Account
                 VirtualAccount userAccount = IPort(localPortAddress).fetchVirtualAccount(
                     address(uint160(bytes20(_payload[PARAMS_START:PARAMS_START_SIGNED])))
                 );
     
                 // Toggle Router Virtual Account use for tx execution
                 IPort(localPortAddress).toggleVirtualAccountApproved(userAccount, localRouterAddress);
     
                 // Avoid stack too deep
                 uint16 srcChainId = _srcChainId;
     
                 // Try to execute remote request
                 // Flag 4 - RootBridgeAgentExecutor(bridgeAgentExecutorAddress).executeSignedNoDeposit(address(userAccount), localRouterAddress, data, _srcChainId
                 _execute(
                     nonce,
                     abi.encodeWithSelector(
                         RootBridgeAgentExecutor.executeSignedNoDeposit.selector,
                         address(userAccount),
                         localRouterAddress,
                         _payload,
                         srcChainId
                     ),
                     srcChainId
                 );
     
                 // Toggle Router Virtual Account use for tx execution
                 IPort(localPortAddress).toggleVirtualAccountApproved(userAccount, localRouterAddress);
     
                 //DEPOSIT FLAG: 5 (Call with Deposit + msg.sender)
             } else if (_payload[0] & 0x7F == 0x05) {
                 // Parse deposit nonce
                 nonce = uint32(bytes4(_payload[PARAMS_START_SIGNED:PARAMS_TKN_START_SIGNED]));
     
                 //Check if tx has already been executed
                 if (executionState[_srcChainId][nonce] != STATUS_READY) {
                     revert AlreadyExecutedTransaction();
                 }
     
                 // Get User Virtual Account
                 VirtualAccount userAccount = IPort(localPortAddress).fetchVirtualAccount(
                     address(uint160(bytes20(_payload[PARAMS_START:PARAMS_START_SIGNED])))
                 );
     
                 // Toggle Router Virtual Account use for tx execution
                 IPort(localPortAddress).toggleVirtualAccountApproved(userAccount, localRouterAddress);
     
                 // Avoid stack too deep
                 uint16 srcChainId = _srcChainId;
     
                 // Try to execute remote request
                 // Flag 5 - RootBridgeAgentExecutor(bridgeAgentExecutorAddress).executeSignedWithDeposit(address(userAccount), localRouterAddress, data, _srcChainId)
                 _execute(
                     _payload[0] == 0x85,
                     nonce,
                     address(uint160(bytes20(_payload[PARAMS_START:PARAMS_START_SIGNED]))),
                     abi.encodeWithSelector(
                         RootBridgeAgentExecutor.executeSignedWithDeposit.selector,
                         address(userAccount),
                         localRouterAddress,
                         _payload,
                         srcChainId
                     ),
                     srcChainId
                 );
     
                 // Toggle Router Virtual Account use for tx execution
                 IPort(localPortAddress).toggleVirtualAccountApproved(userAccount, localRouterAddress);
     
                 // DEPOSIT FLAG: 6 (Call with multiple asset Deposit + msg.sender)
             } else if (_payload[0] & 0x7F == 0x06) {
                 // Parse deposit nonce
                 nonce = uint32(bytes4(_payload[PARAMS_START_SIGNED + PARAMS_START:PARAMS_START_SIGNED + PARAMS_TKN_START]));
     
                 // Check if tx has already been executed
                 if (executionState[_srcChainId][nonce] != STATUS_READY) {
                     revert AlreadyExecutedTransaction();
                 }
     
                 // Get User Virtual Account
                 VirtualAccount userAccount = IPort(localPortAddress).fetchVirtualAccount(
                     address(uint160(bytes20(_payload[PARAMS_START:PARAMS_START_SIGNED])))
                 );
     
                 // Toggle Router Virtual Account use for tx execution
                 IPort(localPortAddress).toggleVirtualAccountApproved(userAccount, localRouterAddress);
     
                 // Avoid stack too deep
                 uint16 srcChainId = _srcChainId;
     
                 // Try to execute remote request
                 // Flag 6 - RootBridgeAgentExecutor(bridgeAgentExecutorAddress).executeSignedWithDepositMultiple(address(userAccount), localRouterAddress, data, _srcChainId)
                 _execute(
                     _payload[0] == 0x86,
                     nonce,
                     address(uint160(bytes20(_payload[PARAMS_START:PARAMS_START_SIGNED]))),
                     abi.encodeWithSelector(
                         RootBridgeAgentExecutor.executeSignedWithDepositMultiple.selector,
                         address(userAccount),
                         localRouterAddress,
                         _payload,
                         srcChainId
                     ),
                     srcChainId
                 );
     
                 // Toggle Router Virtual Account use for tx execution
                 IPort(localPortAddress).toggleVirtualAccountApproved(userAccount, localRouterAddress);
     
                 /// DEPOSIT FLAG: 7 (retrySettlement)
             } else if (_payload[0] & 0x7F == 0x07) {
                 // Prepare Variables for decoding
                 address owner;
                 bytes memory params;
                 GasParams memory gParams;
     
                 // Decode Input
                 (nonce, owner, params, gParams) = abi.decode(_payload[PARAMS_START:], (uint32, address, bytes, GasParams));
     
                 // Get storage reference
                 Settlement storage settlementReference = getSettlement[nonce];
     
                 // Check if Settlement hasn't been redeemed.
                 if (settlementReference.owner == address(0)) revert SettlementRetryUnavailable();
     
                 // Check settlement owner
                 if (owner != settlementReference.owner) {
                     if (owner != address(IPort(localPortAddress).getUserAccount(settlementReference.owner))) {
                         revert NotSettlementOwner();
                     }
                 }
     
                 //Update Settlement Staus
                 settlementReference.status = STATUS_SUCCESS;
     
                 //Retry settlement call with new params and gas
                 _performRetrySettlementCall(
                     _payload[0] == 0x87,
                     settlementReference.hTokens,
                     settlementReference.tokens,
                     settlementReference.amounts,
                     settlementReference.deposits,
                     params,
                     nonce,
                     payable(settlementReference.owner),
                     settlementReference.recipient,
                     settlementReference.dstChainId,
                     gParams,
                     address(this).balance
                 );
     
                 /// DEPOSIT FLAG: 8 (retrieveDeposit)
             } else if (_payload[0] == 0x08) {
                 //Parse deposit nonce
                 nonce = uint32(bytes4(_payload[PARAMS_START_SIGNED:PARAMS_TKN_START_SIGNED]));
     
                 //Check if deposit is in retrieve mode
                 if (executionState[_srcChainId][nonce] == STATUS_DONE) {
                     revert AlreadyExecutedTransaction();
                 } else {
                     //Set settlement to retrieve mode, if not already set.
                     if (executionState[_srcChainId][nonce] == STATUS_READY) {
                         executionState[_srcChainId][nonce] = STATUS_RETRIEVE;
                     }
                     //Trigger fallback/Retry failed fallback
                     _performFallbackCall(
                         payable(address(uint160(bytes20(_payload[PARAMS_START:PARAMS_START_SIGNED])))), nonce, _srcChainId
                     );
                 }
     
                 //DEPOSIT FLAG: 9 (Fallback)
             } else if (_payload[0] == 0x09) {
                 // Parse nonce
                 nonce = uint32(bytes4(_payload[PARAMS_START:PARAMS_TKN_START]));
     
                 // Reopen Settlement for redemption
                 getSettlement[nonce].status = STATUS_FAILED;
     
                 // Emit LogFallback
                 emit LogFallback(nonce, _srcChainId);
     
                 // return to prevent unnecessary emits/logic
                 return;
     
                 // Unrecognized Function Selector
             } else {
                 revert UnknownFlag();
             }
     
             emit LogExecute(nonce, _srcChainId);
         }

856:     function _performRetrySettlementCall(
             bool _hasFallbackToggled,
             address[] memory _hTokens,
             address[] memory _tokens,
             uint256[] memory _amounts,
             uint256[] memory _deposits,
             bytes memory _params,
             uint32 _settlementNonce,
             address payable _refundee,
             address _recipient,
             uint16 _dstChainId,
             GasParams memory _gParams,
             uint256 _value
         ) internal {
             // Check if payload is ready for message
             if (_hTokens.length == 0) revert SettlementRetryUnavailableUseCallout();
     
             // Get packed data
             bytes memory payload;
     
             // Check if it's a single asset settlement
             if (_hTokens.length == 1) {
                 //Pack new Data
                 payload = abi.encodePacked(
                     _hasFallbackToggled ? bytes1(0x81) : bytes1(0x01),
                     _recipient,
                     _settlementNonce,
                     _hTokens[0],
                     _tokens[0],
                     _amounts[0],
                     _deposits[0],
                     _params
                 );
     
                 // Check if it's mulitple asset settlement
             } else if (_hTokens.length > 1) {
                 //Pack new Data
                 payload = abi.encodePacked(
                     _hasFallbackToggled ? bytes1(0x82) : bytes1(0x02),
                     _recipient,
                     uint8(_hTokens.length),
                     _settlementNonce,
                     _hTokens,
                     _tokens,
                     _amounts,
                     _deposits,
                     _params
                 );
             }
     
             // Get destination Branch Bridge Agent
             address callee = getBranchBridgeAgent[_dstChainId];
     
             // Check if valid destination
             if (callee == address(0)) revert UnrecognizedBridgeAgent();
     
             // Check if call to remote chain
             if (_dstChainId != localChainId) {
                 //Sends message to Layerzero Enpoint
                 ILayerZeroEndpoint(lzEndpointAddress).send{value: _value}(
                     _dstChainId,
                     getBranchBridgeAgentPath[_dstChainId],
                     payload,
                     payable(_refundee),
                     address(0),
                     abi.encodePacked(uint16(2), _gParams.gasLimit, _gParams.remoteBranchExecutionGas, callee)
                 );
     
                 // Check if call to local chain
             } else {
                 //Send Gas to Local Branch Bridge Agent
                 callee.call{value: _value}("");
                 //Execute locally
                 IBranchBridgeAgent(callee).lzReceive(0, "", 0, payload);
             }
         }

1131:     function _updateStateOnBridgeOut(
              address _depositor,
              address _globalAddress,
              address _localAddress,
              address _underlyingAddress,
              uint256 _amount,
              uint256 _deposit,
              uint16 _dstChainId
          ) internal {
              // Check if valid inputs
              if (_amount == 0 && _deposit == 0) revert InvalidInputParams();
              if (_amount < _deposit) revert InvalidInputParams();
      
              // Check if valid assets
              if (_localAddress == address(0)) revert UnrecognizedLocalAddress();
              if (_underlyingAddress == address(0)) if (_deposit > 0) revert UnrecognizedUnderlyingAddress();
      
              // Move output hTokens from Root to Branch
              if (_amount - _deposit > 0) {
                  unchecked {
                      _globalAddress.safeTransferFrom(_depositor, localPortAddress, _amount - _deposit);
                  }
              }
      
              // Clear Underlying Tokens from the destination Branch
              if (_deposit > 0) {
                  // Verify there is enough balance to clear native tokens if needed
                  if (IERC20hTokenRoot(_globalAddress).getTokenBalance(_dstChainId) < _deposit) {
                      revert InsufficientBalanceForSettlement();
                  }
                  IPort(localPortAddress).burn(_depositor, _globalAddress, _deposit, _dstChainId);
              }
          }

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1131-L1163)

</details>

### <a name="NC-8-1693316104"></a>[NC-8] Some if-statement can be converted to a ternary
Improving code readability and compactness is an integral part of optimal programming practices. The use of ternary operators in place of if-else conditions is one such measure. Ternary operators allow us to write conditional statements in a more concise manner, thereby enhancing readability and simplicity. They follow the syntax `condition ? exprIfTrue : exprIfFalse`, which interprets as "if the condition is true, evaluate to `exprIfTrue`, else evaluate to `exprIfFalse`". By adopting this approach, we make our code more streamlined and intuitive, which could potentially aid in better understanding and maintenance of the codebase.

*Instances (4)*:
<details><summary> see instances </summary>

```solidity
File: src/BranchBridgeAgent.sol

360:             if (_isSigned) {
                     //Pack new Data
                     payload = abi.encodePacked(
                         _hasFallbackToggled ? bytes1(0x85) : bytes1(0x05),
                         msg.sender,
                         _depositNonce,
                         deposit.hTokens[0],
                         deposit.tokens[0],
                         deposit.amounts[0],
                         deposit.deposits[0],
                         _params
                     );
                 } else {

384:             if (_isSigned) {
                     //Pack new Data
                     payload = abi.encodePacked(
                         _hasFallbackToggled ? bytes1(0x86) : bytes1(0x06),
                         msg.sender,
                         uint8(deposit.hTokens.length),
                         _depositNonce,
                         deposit.hTokens,
                         deposit.tokens,
                         deposit.amounts,
                         deposit.deposits,
                         _params
                     );
                 } else {

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L384-L397)

```solidity
File: src/RootBridgeAgent.sol

708:                 if (executionState[_srcChainId][nonce] == STATUS_READY) {
                         executionState[_srcChainId][nonce] = STATUS_RETRIEVE;
                     }

891:         } else if (_hTokens.length > 1) {
                 //Pack new Data

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L891-L892)

</details>

### <a name="NC-9-1693316148"></a>[NC-9] Contract implements interface without extending the interface
Not extending the interface may lead to the wrong function signature being used, leading to unexpected behavior. If the interface is in fact being implemented, use the `override` keyword to indicate that fact

*Instances (11)*:
<details><summary> see instances </summary>

```solidity
File: src/BranchBridgeAgent.sol

//@audit IBranchBridgeAgent.getFeeEstimate(),  
45: contract BranchBridgeAgent is IBranchBridgeAgent, BridgeAgentConstants {

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L45)

```solidity
File: src/CoreBranchRouter.sol

//@audit ICoreBranchRouter.addLocalToken(),  
18: contract CoreBranchRouter is ICoreBranchRouter, BaseBranchRouter {

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L18)

```solidity
File: src/MulticallRootRouter.sol

//@audit IRootRouter.executeDepositMultiple(),  
57: contract MulticallRootRouter is IRootRouter, Ownable {

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L57)

```solidity
File: src/RootBridgeAgent.sol

//@audit IRootBridgeAgent.lzReceive(),  
32: contract RootBridgeAgent is IRootBridgeAgent, BridgeAgentConstants {

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L32)

```solidity
File: src/RootPort.sol

//@audit IRootPort.isLocalToken(),  
16: contract RootPort is Ownable, IRootPort {

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L16)

```solidity
File: src/VirtualAccount.sol

//@audit IVirtualAccount.payableCall(),  
17: contract VirtualAccount is IVirtualAccount, ERC1155Receiver {

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L17)

```solidity
File: src/factories/BranchBridgeAgentFactory.sol

//@audit IBranchBridgeAgentFactory.createBridgeAgent(),  
19: contract BranchBridgeAgentFactory is Ownable, IBranchBridgeAgentFactory {

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/BranchBridgeAgentFactory.sol#L19)

```solidity
File: src/factories/ERC20hTokenBranchFactory.sol

//@audit IERC20hTokenBranchFactory.createToken(),  
12: contract ERC20hTokenBranchFactory is Ownable, IERC20hTokenBranchFactory {

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenBranchFactory.sol#L12)

```solidity
File: src/factories/ERC20hTokenRootFactory.sol

//@audit IERC20hTokenRootFactory.createToken(),  
12: contract ERC20hTokenRootFactory is Ownable, IERC20hTokenRootFactory {

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenRootFactory.sol#L12)

```solidity
File: src/factories/RootBridgeAgentFactory.sol

//@audit IRootBridgeAgentFactory.createBridgeAgent(),  
11: contract RootBridgeAgentFactory is IRootBridgeAgentFactory {

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/RootBridgeAgentFactory.sol#L11)

```solidity
File: src/token/ERC20hTokenRoot.sol

//@audit IERC20hTokenRoot.burn(),  
12: contract ERC20hTokenRoot is ERC20, Ownable, IERC20hTokenRoot {

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/token/ERC20hTokenRoot.sol#L12)

</details>

### <a name="NC-10-1693316207"></a>[NC-10] Inconsistent usage of `require`/`error`
Some parts of the codebase use `require` statements, while others use custom `error`s. Consider refactoring the code to use the same approach: the following findings represent the minority of `require` vs `error`, and they show the first occurance in each file, for brevity.

*Instances (54)*:
<details><summary> see instances </summary>

```solidity
File: src/ArbitrumBranchPort.sol

39:         require(_rootPortAddress != address(0), "Root Port Address cannot be 0");

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol#L39-L39)

```solidity
File: src/BaseBranchRouter.sol

61:         require(_localBridgeAgentAddress != address(0), "Bridge Agent address cannot be 0");

213:         require(_unlocked == 1);

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L213-L213)

```solidity
File: src/BranchBridgeAgent.sol

125:         require(_rootBridgeAgentAddress != address(0), "Root Bridge Agent Address cannot be the zero address.");

126:         require(

130:         require(_localRouterAddress != address(0), "Local Router Address cannot be the zero address.");

131:         require(_localPortAddress != address(0), "Local Port Address cannot be the zero address.");

923:         require(_unlocked == 1);

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L923-L923)

```solidity
File: src/BranchPort.sol

109:         require(_owner != address(0), "Owner is zero address");

123:         require(coreBranchRouterAddress == address(0), "Contract already initialized");

124:         require(!isBridgeAgentFactory[_bridgeAgentFactory], "Contract already initialized");

126:         require(_coreBranchRouter != address(0), "CoreBranchRouter is zero address");

127:         require(_bridgeAgentFactory != address(0), "BridgeAgentFactory is zero address");

181:         require(ERC20(_token).balanceOf(address(this)) - currBalance == _amount, "Port Strategy Withdraw Failed");

213:         require(

332:         require(coreBranchRouterAddress != address(0), "CoreRouter address is zero");

333:         require(_newCoreRouter != address(0), "New CoreRouter address is zero");

567:         require(_unlocked == 1);

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L567-L567)

```solidity
File: src/CoreRootRouter.sol

84:         require(_setup, "Contract is already initialized");

278:         require(msg.sender == rootPortAddress, "Only root port can call");

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L278-L278)

```solidity
File: src/MulticallRootRouter.sol

93:         require(_localPortAddress != address(0), "Local Port Address cannot be 0");

94:         require(_multicallAddress != address(0), "Multicall Address cannot be 0");

110:         require(_bridgeAgentAddress != address(0), "Bridge Agent Address cannot be 0");

591:         require(_unlocked == 1);

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L591-L591)

```solidity
File: src/RootBridgeAgent.sol

111:         require(_lzEndpointAddress != address(0), "Layerzero Enpoint Address cannot be zero address");

112:         require(_localPortAddress != address(0), "Port Address cannot be zero address");

113:         require(_localRouterAddress != address(0), "Router Address cannot be zero address");

1191:         require(_unlocked == 1);

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1191-L1191)

```solidity
File: src/RootPort.sol

130:         require(_bridgeAgentFactory != address(0), "Bridge Agent Factory cannot be 0 address.");

131:         require(_coreRootRouter != address(0), "Core Root Router cannot be 0 address.");

132:         require(_setup, "Setup ended.");

152:         require(_coreRootBridgeAgent != address(0), "Core Root Bridge Agent cannot be 0 address.");

153:         require(_coreLocalBranchBridgeAgent != address(0), "Core Local Branch Bridge Agent cannot be 0 address.");

154:         require(_localBranchPortAddress != address(0), "Local Branch Port Address cannot be 0 address.");

155:         require(isBridgeAgent[_coreRootBridgeAgent], "Core Bridge Agent doesn't exist.");

156:         require(_setupCore, "Core Setup ended.");

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L156-L156)

```solidity
File: src/factories/ArbitrumBranchBridgeAgentFactory.sol

57:         require(_coreRootBridgeAgent != address(0), "Core Root Bridge Agent Address cannot be 0");

84:         require(

87:         require(

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ArbitrumBranchBridgeAgentFactory.sol#L87-L87)

```solidity
File: src/factories/BranchBridgeAgentFactory.sol

61:         require(_rootBridgeAgentFactoryAddress != address(0), "Root Bridge Agent Factory Address cannot be 0");

62:         require(

66:         require(_localCoreBranchRouterAddress != address(0), "Core Branch Router Address cannot be 0");

67:         require(_localPortAddress != address(0), "Port Address cannot be 0");

68:         require(_owner != address(0), "Owner cannot be 0");

88:         require(_coreRootBridgeAgent != address(0), "Core Root Bridge Agent cannot be 0");

120:         require(

123:         require(

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/BranchBridgeAgentFactory.sol#L123-L123)

```solidity
File: src/factories/ERC20hTokenBranchFactory.sol

43:         require(_localPortAddress != address(0), "Port address cannot be 0");

61:         require(_coreRouter != address(0), "CoreRouter address cannot be 0");

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenBranchFactory.sol#L61-L61)

```solidity
File: src/factories/ERC20hTokenRootFactory.sol

35:         require(_rootPortAddress != address(0), "Root Port Address cannot be 0");

50:         require(_coreRouter != address(0), "CoreRouter address cannot be 0");

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenRootFactory.sol#L50-L50)

```solidity
File: src/factories/RootBridgeAgentFactory.sol

32:         require(_rootPortAddress != address(0), "Root Port Address cannot be 0");

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/RootBridgeAgentFactory.sol#L32-L32)

```solidity
File: src/token/ERC20hTokenRoot.sol

39:         require(_rootPortAddress != address(0), "Root Port Address cannot be 0");

40:         require(_factoryAddress != address(0), "Factory Address cannot be 0");

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/token/ERC20hTokenRoot.sol#L40-L40)

</details>

### <a name="NC-11-1693316644"></a>[NC-11] Some error strings are not descriptive
Consider adding more detail to these error strings

*Instances (1)*:
<details><summary> see instances </summary>

```solidity
File: src/RootPort.sol

//@audit This message need more details : Setup ended.
132:         require(_setup, "Setup ended.");

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L132)

</details>

### <a name="NC-12-1693317147"></a>[NC-12] State variables should include comments
Consider adding some comments on critical state variables to explain what they are supposed to do: this will help for future code reviews.

*Instances (22)*:
<details><summary> see instances </summary>

```solidity
File: src/BranchPort.sol

//@audit DIVISIONER need comments
97:     uint256 internal constant DIVISIONER = 1e4;

//@audit MIN_RESERVE_RATIO need comments
98:     uint256 internal constant MIN_RESERVE_RATIO = 3e3;

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L98)

```solidity
File: src/interfaces/BridgeAgentConstants.sol

//@audit STATUS_READY need comments
13:     uint8 internal constant STATUS_READY = 0;

//@audit STATUS_DONE need comments
15:     uint8 internal constant STATUS_DONE = 1;

//@audit STATUS_RETRIEVE need comments
17:     uint8 internal constant STATUS_RETRIEVE = 2;

//@audit STATUS_FAILED need comments
21:     uint8 internal constant STATUS_FAILED = 1;

//@audit STATUS_SUCCESS need comments
23:     uint8 internal constant STATUS_SUCCESS = 0;

//@audit PARAMS_START need comments
27:     uint256 internal constant PARAMS_START = 1;

//@audit PARAMS_START_SIGNED need comments
29:     uint256 internal constant PARAMS_START_SIGNED = 21;

//@audit PARAMS_TKN_START need comments
31:     uint256 internal constant PARAMS_TKN_START = 5;

//@audit PARAMS_TKN_START_SIGNED need comments
33:     uint256 internal constant PARAMS_TKN_START_SIGNED = 25;

//@audit PARAMS_ENTRY_SIZE need comments
35:     uint256 internal constant PARAMS_ENTRY_SIZE = 32;

//@audit PARAMS_ADDRESS_SIZE need comments
37:     uint256 internal constant PARAMS_ADDRESS_SIZE = 20;

//@audit PARAMS_TKN_SET_SIZE need comments
39:     uint256 internal constant PARAMS_TKN_SET_SIZE = 109;

//@audit PARAMS_TKN_SET_SIZE_MULTIPLE need comments
41:     uint256 internal constant PARAMS_TKN_SET_SIZE_MULTIPLE = 128;

//@audit ADDRESS_END_OFFSET need comments
43:     uint256 internal constant ADDRESS_END_OFFSET = 12;

//@audit PARAMS_AMT_OFFSET need comments
45:     uint256 internal constant PARAMS_AMT_OFFSET = 64;

//@audit PARAMS_DEPOSIT_OFFSET need comments
47:     uint256 internal constant PARAMS_DEPOSIT_OFFSET = 96;

//@audit PARAMS_END_OFFSET need comments
49:     uint256 internal constant PARAMS_END_OFFSET = 6;

//@audit PARAMS_END_SIGNED_OFFSET need comments
51:     uint256 internal constant PARAMS_END_SIGNED_OFFSET = 26;

//@audit PARAMS_SETTLEMENT_OFFSET need comments
53:     uint256 internal constant PARAMS_SETTLEMENT_OFFSET = 129;

//@audit MAX_TOKENS_LENGTH need comments
57:     uint256 internal constant MAX_TOKENS_LENGTH = 255;

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/BridgeAgentConstants.sol#L57)

</details>

### <a name="NC-13-1693317214"></a>[NC-13] Top level pragma declarations should be separated by two blank lines

*Instances (37)*:
<details><summary> see instances </summary>

```solidity
File: src/ArbitrumBranchBridgeAgent.sol

2: pragma solidity ^0.8.0;
   
   import {IArbitrumBranchPort as IArbPort} from "./interfaces/IArbitrumBranchPort.sol";

8: import {BranchBridgeAgent} from "./BranchBridgeAgent.sol";
   
   library DeployArbitrumBranchBridgeAgent {

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchBridgeAgent.sol#L8-L10)

```solidity
File: src/ArbitrumBranchPort.sol

3: pragma solidity ^0.8.0;
   
   import {SafeTransferLib} from "solady/utils/SafeTransferLib.sol";

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol#L3-L5)

```solidity
File: src/ArbitrumCoreBranchRouter.sol

2: pragma solidity ^0.8.0;
   
   import {ERC20} from "solmate/tokens/ERC20.sol";

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumCoreBranchRouter.sol#L2-L4)

```solidity
File: src/BaseBranchRouter.sol

2: pragma solidity ^0.8.0;
   
   import {Ownable} from "solady/auth/Ownable.sol";

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L2-L4)

```solidity
File: src/BranchBridgeAgent.sol

2: pragma solidity ^0.8.0;
   
   import {ExcessivelySafeCall} from "lib/ExcessivelySafeCall.sol";

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L2-L4)

```solidity
File: src/BranchBridgeAgentExecutor.sol

2: pragma solidity ^0.8.0;
   
   import {Ownable} from "solady/auth/Ownable.sol";

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgentExecutor.sol#L2-L4)

```solidity
File: src/BranchPort.sol

3: pragma solidity ^0.8.0;
   
   import {Ownable} from "solady/auth/Ownable.sol";

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L3-L5)

```solidity
File: src/CoreBranchRouter.sol

2: pragma solidity ^0.8.0;
   
   import {ERC20} from "solmate/tokens/ERC20.sol";

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L2-L4)

```solidity
File: src/CoreRootRouter.sol

2: pragma solidity ^0.8.0;
   
   import {Ownable} from "solady/auth/Ownable.sol";

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L2-L4)

```solidity
File: src/MulticallRootRouter.sol

2: pragma solidity ^0.8.0;
   
   import {Ownable} from "solady/auth/Ownable.sol";

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L2-L4)

```solidity
File: src/MulticallRootRouterLibZip.sol

2: pragma solidity ^0.8.0;
   
   import {LibZip} from "solady/utils/LibZip.sol";

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouterLibZip.sol#L2-L4)

```solidity
File: src/RootBridgeAgent.sol

2: pragma solidity ^0.8.0;
   
   import {SafeTransferLib} from "solady/utils/SafeTransferLib.sol";

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L2-L4)

```solidity
File: src/RootBridgeAgentExecutor.sol

2: pragma solidity ^0.8.0;
   
   import {Ownable} from "solady/auth/Ownable.sol";

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L2-L4)

```solidity
File: src/RootPort.sol

3: pragma solidity ^0.8.0;
   
   import {Ownable} from "solady/auth/Ownable.sol";

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L3-L5)

```solidity
File: src/VirtualAccount.sol

3: pragma solidity ^0.8.0;
   
   import {SafeTransferLib} from "solady/utils/SafeTransferLib.sol";

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L3-L5)

```solidity
File: src/factories/ArbitrumBranchBridgeAgentFactory.sol

2: pragma solidity ^0.8.0;
   
   import {IArbitrumBranchPort as IPort} from "../interfaces/IArbitrumBranchPort.sol";

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ArbitrumBranchBridgeAgentFactory.sol#L2-L4)

```solidity
File: src/factories/BranchBridgeAgentFactory.sol

2: pragma solidity ^0.8.0;
   
   import {Ownable} from "solady/auth/Ownable.sol";

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/BranchBridgeAgentFactory.sol#L2-L4)

```solidity
File: src/factories/ERC20hTokenBranchFactory.sol

2: pragma solidity ^0.8.0;
   
   import {Ownable} from "solady/auth/Ownable.sol";

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenBranchFactory.sol#L2-L4)

```solidity
File: src/factories/ERC20hTokenRootFactory.sol

2: pragma solidity ^0.8.0;
   
   import {Ownable} from "solady/auth/Ownable.sol";

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenRootFactory.sol#L2-L4)

```solidity
File: src/factories/RootBridgeAgentFactory.sol

2: pragma solidity ^0.8.0;
   
   import {IRootBridgeAgentFactory} from "../interfaces/IRootBridgeAgentFactory.sol";

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/RootBridgeAgentFactory.sol#L2-L4)

```solidity
File: src/interfaces/IArbitrumBranchPort.sol

3: pragma solidity ^0.8.0;
   
   import {IBranchPort} from "./IBranchPort.sol";

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/IArbitrumBranchPort.sol#L3-L5)

```solidity
File: src/interfaces/IBranchBridgeAgent.sol

2: pragma solidity ^0.8.0;
   
   import {ILayerZeroReceiver} from "./ILayerZeroReceiver.sol";

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/IBranchBridgeAgent.sol#L2-L4)

```solidity
File: src/interfaces/IBranchRouter.sol

2: pragma solidity ^0.8.0;
   
   import {

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/IBranchRouter.sol#L2-L4)

```solidity
File: src/interfaces/ICoreBranchRouter.sol

2: pragma solidity ^0.8.0;
   
   import {GasParams} from "./IBranchBridgeAgent.sol";

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/ICoreBranchRouter.sol#L2-L4)

```solidity
File: src/interfaces/IERC20hTokenBranchFactory.sol

2: pragma solidity ^0.8.0;
   
   import {ERC20hTokenBranch} from "../token/ERC20hTokenBranch.sol";

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/IERC20hTokenBranchFactory.sol#L2-L4)

```solidity
File: src/interfaces/IERC20hTokenRootFactory.sol

2: pragma solidity ^0.8.0;
   
   import {ERC20hTokenRoot} from "../token/ERC20hTokenRoot.sol";

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/IERC20hTokenRootFactory.sol#L2-L4)

```solidity
File: src/interfaces/ILayerZeroEndpoint.sol

3: pragma solidity >=0.5.0;
   
   import "./ILayerZeroUserApplicationConfig.sol";

5: import "./ILayerZeroUserApplicationConfig.sol";
   
   interface ILayerZeroEndpoint is ILayerZeroUserApplicationConfig {

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/ILayerZeroEndpoint.sol#L5-L7)

```solidity
File: src/interfaces/ILayerZeroReceiver.sol

3: pragma solidity >=0.5.0;
   
   interface ILayerZeroReceiver {

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/ILayerZeroReceiver.sol#L3-L5)

```solidity
File: src/interfaces/ILayerZeroUserApplicationConfig.sol

3: pragma solidity >=0.5.0;
   
   interface ILayerZeroUserApplicationConfig {

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/ILayerZeroUserApplicationConfig.sol#L3-L5)

```solidity
File: src/interfaces/IRootBridgeAgent.sol

2: pragma solidity ^0.8.0;
   
   import {ILayerZeroReceiver} from "./ILayerZeroReceiver.sol";

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/IRootBridgeAgent.sol#L2-L4)

```solidity
File: src/interfaces/IRootPort.sol

3: pragma solidity ^0.8.0;
   
   import {GasParams} from "../interfaces/IRootBridgeAgent.sol";

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/IRootPort.sol#L3-L5)

```solidity
File: src/interfaces/IRootRouter.sol

2: pragma solidity ^0.8.0;
   
   import {DepositParams, DepositMultipleParams} from "../interfaces/IRootBridgeAgent.sol";

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/IRootRouter.sol#L2-L4)

```solidity
File: src/interfaces/IVirtualAccount.sol

2: pragma solidity ^0.8.0;
   
   import {IERC721Receiver} from "@openzeppelin/contracts/token/ERC721/IERC721Receiver.sol";

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/IVirtualAccount.sol#L2-L4)

```solidity
File: src/token/ERC20hTokenBranch.sol

2: pragma solidity ^0.8.0;
   
   import {Ownable} from "solady/auth/Ownable.sol";

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/token/ERC20hTokenBranch.sol#L2-L4)

```solidity
File: src/token/ERC20hTokenRoot.sol

2: pragma solidity ^0.8.0;
   
   import {Ownable} from "solady/auth/Ownable.sol";

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/token/ERC20hTokenRoot.sol#L2-L4)

</details>

### <a name="NC-14-1693317342"></a>[NC-14] Uncommented fields in a struct
Consider adding comments for all the fields in a struct to improve the readability of the codebase.

*Instances (7)*:
<details><summary> see instances </summary>

```solidity
File: src/MulticallRootRouter.sol

//@audit Add explanational comments to the following items `recipient`, `outputToken`, `amountOut`, `depositOut`, 
17: struct OutputParams {
        // Address to receive the output assets.
        address recipient;
        // Address of the output hToken.
        address outputToken;
        // Amount of output hTokens to send.
        uint256 amountOut;
        // Amount of output underlying token to send.
        uint256 depositOut;
    }

//@audit Add explanational comments to the following items `recipient`, `outputTokens`, `amountsOut`, `depositsOut`, 
28: struct OutputMultipleParams {
        // Address to receive the output assets.
        address recipient;
        // Addresses of the output hTokens.
        address[] outputTokens;
        // Amounts of output hTokens to send.
        uint256[] amountsOut;
        // Amounts of output underlying tokens to send.
        uint256[] depositsOut;
    }

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L28-L37)

```solidity
File: src/interfaces/BridgeAgentStructs.sol

//@audit Add explanational comments to the following items `status`, `owner`, `hTokens`, `tokens`, `amounts`, `deposits`, 
13: struct Deposit {
        uint8 status;
        address owner;
        address[] hTokens;
        address[] tokens;
        uint256[] amounts;
        uint256[] deposits;
    }

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/BridgeAgentStructs.sol#L13-L20)

```solidity
File: src/interfaces/IMulticall2.sol

//@audit Add explanational comments to the following items `target`, `callData`, 
10:     struct Call {
            address target;
            bytes callData;
        }

//@audit Add explanational comments to the following items `success`, `returnData`, 
15:     struct Result {
            bool success;
            bytes returnData;
        }

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/IMulticall2.sol#L15-L18)

```solidity
File: src/interfaces/IVirtualAccount.sol

//@audit Add explanational comments to the following items `target`, `callData`, 
7: struct Call {
       address target;
       bytes callData;
   }

//@audit Add explanational comments to the following items `target`, `callData`, `value`, 
13: struct PayableCall {
        address target;
        bytes callData;
        uint256 value;
    }

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/IVirtualAccount.sol#L13-L17)

</details>

### <a name="NC-15-1693317551"></a>[NC-15] Missing upgradability functionality
At the begining of a project, there is always the need to modify of add something to the source code especialy if any vulnerability is discovered. Therefore, having such system is crusial at least at the first stages of the project

*Instances (1)*:
<details><summary> see instances </summary>

```solidity
File: src/ArbitrumBranchBridgeAgent.sol

1: // SPDX-License-Identifier: MIT

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchBridgeAgent.sol#L1)

</details>

### <a name="NC-16-1693317704"></a>[NC-16] Use a single file for system wide constants
Consider grouping all the system constants under a single file. This finding shows only the first constant for each file.

*Instances (2)*:
<details><summary> see instances </summary>

```solidity
File: src/BranchPort.sol

97:     uint256 internal constant DIVISIONER = 1e4;

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L97)

```solidity
File: src/interfaces/BridgeAgentConstants.sol

13:     uint8 internal constant STATUS_READY = 0;

```
[Link to code](https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/BridgeAgentConstants.sol#L13)

</details>

