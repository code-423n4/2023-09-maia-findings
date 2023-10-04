# MAIA DAO - ULYSSES GAS OPTIMIZATIONS


## INTRODUCTION 
The majority of optimizations underwent benchmarking using the protocol's tests, specifically employing the following configuration: `solc version 0.8.19`, `optimizer enabled,` and `200` runs. For optimizations that were not subjected to benchmarking, their rationale is elucidated through EVM gas expenses and opcodes.

Highlighted below are optimizations exclusively targeting state-mutating functions and view/pure functions invoked by state-mutating functions. In the discussion that follows, only runtime gas is emphasized, given its inevitable dominance over deployment gas costs throughout the protocol's lifetime. 

Please be aware that some code snippets may be shortened to conserve space, and certain code snippets may include @audit tags in comments to facilitate issue explanations."


## TABLE OF FINDINGS

| Number |Issue|Instances|Total Gas Saved|
|-|:-|:-:|:-:|
| G-01| Add unchecked blocks for subtractions where the operands cannot underflow | 1 | 40 |
| G-02| Cache state/storage variables outside of loop to avoid reading storage on every iteration | 1 | 101 |
| G-03| Unnecessary require statements | 1 | - |
| G-04| Declaring Unnecessary variables | 1 | 3 |
| G-05| Redundant state variable getters | 2 |  - |
| G-06| Calculations should be memoized rather than re-calculating them | 3 | - |
| G-07| State/Storage variables should be cached in stack variables rather than re-reading them from storage. | 1 |  6 |
| G-08| Initializers can be marked as payable to save deployment gas | 9 | 120 |
| G-09| Unbounded Gas Consumption Risk Due to External Call Recipients | 6 | -  |
| G-10| Multiplication by two should use bit shifting | 22 | 440  |




## [G-01] Add unchecked blocks for subtractions where the operands cannot underflow
There are some checks to avoid an underflow, but in some scenarios where it is impossible for underflow to occur we can use unchecked blocks to have some gas savings.

### 1 Instance
1. #### Put `getPortStrategyTokenDebt[_strategy][_token] = portStrategyTokenDebt - amountToWithdraw` statement in an unchecked block.
- https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L205
```solidity
file: src/BranchPort.sol

188:    function replenishReserves(address _strategy, address _token) external override lock {
189:        // Cache Strategy Token Global Debt
190:        uint256 strategyTokenDebt = getStrategyTokenDebt[_token];
191:
192:        // Get current balance of _token
193:        uint256 currBalance = ERC20(_token).balanceOf(address(this));
194:
195:        // Get reserves lacking
196:        uint256 reservesLacking = _reservesLacking(strategyTokenDebt, _token, currBalance);
197:
198:        // Cache Port Strategy Token Debt
199:        uint256 portStrategyTokenDebt = getPortStrategyTokenDebt[_strategy][_token];
200:
201:        // Calculate amount to withdraw. The lesser of reserves lacking or Strategy Token Global Debt.
202:        uint256 amountToWithdraw = portStrategyTokenDebt < reservesLacking ? portStrategyTokenDebt : reservesLacking;
203:
204:        // Update Port Strategy Token Debt
205:        getPortStrategyTokenDebt[_strategy][_token] = portStrategyTokenDebt - amountToWithdraw; //@audit arithmetic can be in an unchecked block
.
.
.
219:    }
```
In the `replenishReserves()` function above the `getPortStrategyTokenDebt[_strategy][_token] = portStrategyTokenDebt - amountToWithdraw` statement should be in an unchecked block this is because the `uint256 amountToWithdraw = portStrategyTokenDebt < reservesLacking ? portStrategyTokenDebt : reservesLacking` statement ensures that whatever value the `amountToWithdraw` variable results to it would not be greater than `portStrategyTokenDebt` at most they variables would be equal so there is no possiblity of an underflow. Implementing this change would save between `20` to `40` gas units.  The diff below shows how the code could be refactored:
```diff
diff --git a/src/BranchPort.sol b/src/BranchPort.sol
index ba60acc..a87ad27 100644
--- a/src/BranchPort.sol
+++ b/src/BranchPort.sol
@@ -201,8 +201,11 @@ contract BranchPort is Ownable, IBranchPort {
         // Calculate amount to withdraw. The lesser of reserves lacking or Strategy Token Global Debt.
         uint256 amountToWithdraw = portStrategyTokenDebt < reservesLacking ? portStrategyTokenDebt : reservesLacking;

-        // Update Port Strategy Token Debt
-        getPortStrategyTokenDebt[_strategy][_token] = portStrategyTokenDebt - amountToWithdraw;
+        unchecked {
+            // Update Port Strategy Token Debt
+            getPortStrategyTokenDebt[_strategy][_token] = portStrategyTokenDebt - amountToWithdraw;
+        }
+
         // Update Strategy Token Global Debt
         getStrategyTokenDebt[_token] = strategyTokenDebt - amountToWithdraw;

```
```
Estimated gas saved: 40 gas units
```



## [G-02] Cache state/storage variables outside of loop to avoid reading storage on every iteration
Reading from storage should always try to be avoided within loops. In the following instances, we are able to cache state variables outside of the loop to save a Gwarmaccess (100 gas) per loop iteration.

### 1 Instance
1. #### Move `int24 _dstChainId = settlement.dstChainId` outside of the loop
- https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L325
```solidity
file: /src/RootBridgeAgent.sol

299:    function redeemSettlement(uint32 _settlementNonce) external override lock {
.
.
.
318:        // Clear Global hTokens To Recipient on Root Chain cancelling Settlement to Branch
319:        for (uint256 i = 0; i < settlement.hTokens.length;) {
320:            // Save to memory
321:            address _hToken = settlement.hTokens[i];
322:
323:            // Check if asset
324:            if (_hToken != address(0)) {
325:                // Save to memory
326:                uint24 _dstChainId = settlement.dstChainId; //@audit cache storage variable out of loop
327:
328:                // Move hTokens from Branch to Root + Mint Sufficient hTokens to match new port deposit
329:                IPort(localPortAddress).bridgeToRoot(
330:                    msg.sender,
331:                    IPort(localPortAddress).getGlobalTokenFromLocal(_hToken, _dstChainId),
332:                    settlement.amounts[i],
333:                    settlement.deposits[i],
334:                    _dstChainId
335:                );
336:            }
337:
338:            unchecked {
339:                ++i;
340:            }
341:        }
.
.
.
    }
```
The `uint24 _dstChainId = settlement.dstChainId` statement should be declared before the loop as its value does not change per iteration of the loop. Implementing this change would help avoid reading the value `settlement.dstChainId` anytime the `_hToken != address(0)` condition results to `true` during the loop iterations. The code should be refactored as shown in the diff below:
```diff
diff --git a/src/RootBridgeAgent.sol b/src/RootBridgeAgent.sol
index a6ad0ef..5579460 100644
--- a/src/RootBridgeAgent.sol
+++ b/src/RootBridgeAgent.sol
@@ -314,6 +314,9 @@ contract RootBridgeAgent is IRootBridgeAgent, BridgeAgentConstants {
             }
         }

+        // Save to memory
+        uint24 _dstChainId = settlement.dstChainId;
+
         // Clear Global hTokens To Recipient on Root Chain cancelling Settlement to Branch
         for (uint256 i = 0; i < settlement.hTokens.length;) {
             // Save to memory
@@ -321,8 +324,7 @@ contract RootBridgeAgent is IRootBridgeAgent, BridgeAgentConstants {

             // Check if asset
             if (_hToken != address(0)) {
-                // Save to memory
-                uint24 _dstChainId = settlement.dstChainId;
+

                 // Move hTokens from Branch to Root + Mint Sufficient hTokens to match new port deposit
                 IPort(localPortAddress).bridgeToRoot(
```

#### Gas saving for `redeemSettlement()` function obtained via protocol test: Avg 101 gas
|  | Min | Max | Avg | # Calls |
|----------|----------|----------|----------|----------|
| Before | 11045 | 28464 | 22725 | 3 |
| After | 10944 | 28464 | 22624 | 3 |



## [G-03]  Unnecessary require statements

### 1 Instance
1. #### The require statement `require(coreBranchRouterAddress != address(0), "CoreRouter address is zero")` of `setCoreRouter()` is unnecessary as its condition has been covered by the `requiresCoreRouter` modifier of the `setCoreRouter()` function.
- https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L331
```solidity
file: src/BranchPort.sol

331:    function setCoreRouter(address _newCoreRouter) external override requiresCoreRouter {
332:        require(coreBranchRouterAddress != address(0), "CoreRouter address is zero");   //@audit require statement is unnecessary
333:        require(_newCoreRouter != address(0), "New CoreRouter address is zero");
334:        coreBranchRouterAddress = _newCoreRouter;
335:    }
```
The `requiresCoreRouter` modifier ensures that `msg.sender` must be equal to `coreBranchRouterAddress` else the transaction would be reverted. Since it is not possible for the `msg.sender` to be `address(0)` (as there is no known private key for the zero address) in a scenario where the value of `coreBranchRouterAddress` is the zero address the `requiresCoreRouter` modifier would ensure that the transaction reverts since as explained above `msg.sender` cannot be zero address thereby making the `require(coreBranchRouterAddress != address(0), "CoreRouter address is zero")` statement unnecessary. The code could be refactored as shown in the diff below:
```diff
diff --git a/src/BranchPort.sol b/src/BranchPort.sol
index ba60acc..44216dc 100644
--- a/src/BranchPort.sol
+++ b/src/BranchPort.sol
@@ -329,7 +329,6 @@ contract BranchPort is Ownable, IBranchPort {

     /// @inheritdoc IBranchPort
     function setCoreRouter(address _newCoreRouter) external override requiresCoreRouter {
-        require(coreBranchRouterAddress != address(0), "CoreRouter address is zero");
         require(_newCoreRouter != address(0), "New CoreRouter address is zero");
         coreBranchRouterAddress = _newCoreRouter;
     }
```


## [G-04] Declaring Unnecessary variables
some varibles were defined even though they are used once. Not defining variables can reduce gas cost and contract size.

### 1 Instances
1. #### The `length` variable is unnecessary
- https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L131
```solidity
file: src/RootBridgeAgentExecutor.sol

115:    function executeWithDepositMultiple(address _router, bytes calldata _payload, uint16 _srcChainId)
116:        external
117:        payable
118:        onlyOwner
119:    {
120:        //Bridge In Assets and Save Deposit Params
121:        DepositMultipleParams memory dParams = _bridgeInMultiple(
122:            _router,
123:            _payload[
124:                PARAMS_START:
125:                    PARAMS_END_OFFSET + uint256(uint8(bytes1(_payload[PARAMS_START]))) * PARAMS_TKN_SET_SIZE_MULTIPLE
126:            ],
127:            _srcChainId
128:        );
129:
130:        uint256 numOfAssets = uint8(bytes1(_payload[PARAMS_START]));
131:        uint256 length = _payload.length;  //@audit length variable unnecessary
132:
133:        // Check if there is additional calldata in the payload
134:        if (length > PARAMS_END_OFFSET + (numOfAssets * PARAMS_TKN_SET_SIZE_MULTIPLE)) {
135:            //Try to execute remote request
136:            IRouter(_router).executeDepositMultiple{value: msg.value}(
137:                _payload[PARAMS_END_OFFSET + uint256(numOfAssets) * PARAMS_TKN_SET_SIZE_MULTIPLE:], dParams, _srcChainId
138:            );
139:        }
140:    }
```
In the `executeWithDepositMultiple()` function above `length` variable was declared on [line 131](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L131) but only used once in the function thereby making its declaration redundant and unnecessary. `3` units of gas can be saved if we remove this declaration and use `_payload.length` directly. The code could be refactored as shown in the diff below:
```diff
diff --git a/src/RootBridgeAgentExecutor.sol b/src/RootBridgeAgentExecutor.sol
index b23cc98..18c9863 100644
--- a/src/RootBridgeAgentExecutor.sol
+++ b/src/RootBridgeAgentExecutor.sol
@@ -128,10 +128,10 @@ contract RootBridgeAgentExecutor is Ownable, BridgeAgentConstants {
         );

         uint256 numOfAssets = uint8(bytes1(_payload[PARAMS_START]));
-        uint256 length = _payload.length;
+

         // Check if there is additional calldata in the payload
-        if (length > PARAMS_END_OFFSET + (numOfAssets * PARAMS_TKN_SET_SIZE_MULTIPLE)) {
+        if (_payload.length > PARAMS_END_OFFSET + (numOfAssets * PARAMS_TKN_SET_SIZE_MULTIPLE)) {
             //Try to execute remote request
             IRouter(_router).executeDepositMultiple{value: msg.value}(
                 _payload[PARAMS_END_OFFSET + uint256(numOfAssets) * PARAMS_TKN_SET_SIZE_MULTIPLE:], dParams, _srcChainId
```
```
Estimated gas saved: 3 gas units
```


## [G-05] Redundant state variable getters
Getters for public state variables are automatically generated by the solidity compiler so there is no need to code them manually as this increases deployment cost.

### 2 Instances
1. #### Make the `getDeposit` state variable private/internal since a getter function `getDepositEntry()` was defined.
- https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L87
```solidity
file: src/BranchBridgeAgent.sol

87:    mapping(uint256 depositNonce => Deposit depositInfo) public getDeposit;  
.
.
.
156:    function getDepositEntry(uint32 _depositNonce) external view override returns (Deposit memory) { //@audit getter function for public state variable getDeposit
157:        return getDeposit[_depositNonce];
158:    }
```
Getter functions are automatically generated by the solidity compiler for public state variables so it is redundant and unnecessary for you to manually define a getter function for a public state variable as this would increase deployment cost. You could either make the state variable `getDeposit` private (or internal if its to be inherited) or refactor the code to remove the getter function that was defined. The code could be defined as shown in the diff below:
```diff
diff --git a/src/BranchBridgeAgent.sol b/src/BranchBridgeAgent.sol
index b076d2d..f767ea6 100644
--- a/src/BranchBridgeAgent.sol
+++ b/src/BranchBridgeAgent.sol
@@ -84,7 +84,7 @@ contract BranchBridgeAgent is IBranchBridgeAgent, BridgeAgentConstants {
     uint32 public depositNonce;

     /// @notice Mapping from Pending deposits hash to Deposit Struct.
-    mapping(uint256 depositNonce => Deposit depositInfo) public getDeposit;
+    mapping(uint256 depositNonce => Deposit depositInfo) internal getDeposit;
```

2. #### Make the `getSettlement` state variable private/internal since a getter function `getSettlementEntry()` was defined.
- https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L135
```solidity
file: src/RootBridgeAgent.sol

78:    mapping(uint256 nonce => Settlement settlementInfo) public getSettlement;
.
.
.
135:    function getSettlementEntry(uint32 _settlementNonce) external view override returns (Settlement memory) { //@audit getter function for public state variable getSettlement
136:        return getSettlement[_settlementNonce];
137:    }
```
Getter functions are automatically generated by the solidity compiler for public state variables so it is redundant and unnecessary for you to manually define a getter function for a public state variable as this would increase deployment cost. You could either make the state variable `getSettlement` private (or internal if its to be inherited) or refactor the code to remove the getter function that was defined. The code could be defined as shown in the diff below:
```diff
diff --git a/src/RootBridgeAgent.sol b/src/RootBridgeAgent.sol
index a6ad0ef..4cbd3eb 100644
--- a/src/RootBridgeAgent.sol
+++ b/src/RootBridgeAgent.sol
@@ -75,7 +75,7 @@ contract RootBridgeAgent is IRootBridgeAgent, BridgeAgentConstants {
     uint32 public settlementNonce;

     /// @notice Mapping from Settlement nonce to Settlement Struct.
-    mapping(uint256 nonce => Settlement settlementInfo) public getSettlement;
+    mapping(uint256 nonce => Settlement settlementInfo) internal getSettlement;
```


## [G-06] Calculations should be memoized rather than re-calculating them
In computing, memoization or memoisation is an optimization technique used primarily to speed up computer programs by storing the results of expensive function calls to pure functions and returning the cached result when the same inputs occur again.

### 3 Instances
1. #### We can memoize the calculation `PARAMS_END_SIGNED_OFFSET + uint256(uint8(bytes1(_payload[PARAMS_START_SIGNED]))) * PARAMS_TKN_SET_SIZE_MULTIPLE`.
- https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L212
```solidity
file: src/RootBridgeAgentExecutor.sol

201:    function executeSignedWithDepositMultiple(
202:        address _account,
203:        address _router,
204:        bytes calldata _payload,
205:        uint16 _srcChainId
206:    ) external payable onlyOwner {
207:        //Bridge In Assets
208:        DepositMultipleParams memory dParams = _bridgeInMultiple(
209:            _account,
210:            _payload[
211:                PARAMS_START_SIGNED:
212:                    PARAMS_END_SIGNED_OFFSET    //@audit memoize calculation
213:                        + uint256(uint8(bytes1(_payload[PARAMS_START_SIGNED]))) * PARAMS_TKN_SET_SIZE_MULTIPLE
214:            ],
215:            _srcChainId
216:        );
217:
218:        // Check if there is additional calldata in the payload
219:        if (
220:            _payload.length
221:                > PARAMS_END_SIGNED_OFFSET  //@audit same calculation
222:                    + uint256(uint8(bytes1(_payload[PARAMS_START_SIGNED]))) * PARAMS_TKN_SET_SIZE_MULTIPLE
223:        ) {
224:            //Execute remote request
225:            IRouter(_router).executeSignedDepositMultiple{value: msg.value}(
226:                _payload[
227:                    PARAMS_END_SIGNED_OFFSET  //@audit same calculation
228:                        + uint256(uint8(bytes1(_payload[PARAMS_START_SIGNED]))) * PARAMS_TKN_SET_SIZE_MULTIPLE:
229:                ],
230:                dParams,
231:                _account,
232:                _srcChainId
233:            );
234:        }
235:    }
```
In the `executeSignedWithDepositMultiple()` function above the arithmetic `PARAMS_END_SIGNED_OFFSET + uint256(uint8(bytes1(_payload[PARAMS_START_SIGNED]))) * PARAMS_TKN_SET_SIZE_MULTIPLE` was repeated multiple times. We could save the gas used in the subsequent calculations if we memoize the calculation i.e we cache the result of the calcultion the first time in a variable and use the variable in place of the subsequent calculations. The diff below shows how the code could be refactored: 
```diff
diff --git a/src/RootBridgeAgentExecutor.sol b/src/RootBridgeAgentExecutor.sol
index b23cc98..640139d 100644
--- a/src/RootBridgeAgentExecutor.sol
+++ b/src/RootBridgeAgentExecutor.sol
@@ -205,12 +205,15 @@ contract RootBridgeAgentExecutor is Ownable, BridgeAgentConstants {
         uint16 _srcChainId
     ) external payable onlyOwner {
         //Bridge In Assets
+
+        uint256 payloadIndex = PARAMS_END_SIGNED_OFFSET +
+                                    uint256(uint8(bytes1(_payload[PARAMS_START_SIGNED]))) * PARAMS_TKN_SET_SIZE_MULTIPLE;
+
         DepositMultipleParams memory dParams = _bridgeInMultiple(
             _account,
             _payload[
                 PARAMS_START_SIGNED:
-                    PARAMS_END_SIGNED_OFFSET
-                        + uint256(uint8(bytes1(_payload[PARAMS_START_SIGNED]))) * PARAMS_TKN_SET_SIZE_MULTIPLE
+                    payloadIndex
             ],
             _srcChainId
         );
@@ -218,14 +221,12 @@ contract RootBridgeAgentExecutor is Ownable, BridgeAgentConstants {
         // Check if there is additional calldata in the payload
         if (
             _payload.length
-                > PARAMS_END_SIGNED_OFFSET
-                    + uint256(uint8(bytes1(_payload[PARAMS_START_SIGNED]))) * PARAMS_TKN_SET_SIZE_MULTIPLE
+                > payloadIndex
         ) {
             //Execute remote request
             IRouter(_router).executeSignedDepositMultiple{value: msg.value}(
                 _payload[
-                    PARAMS_END_SIGNED_OFFSET
-                        + uint256(uint8(bytes1(_payload[PARAMS_START_SIGNED]))) * PARAMS_TKN_SET_SIZE_MULTIPLE:
+                    payloadIndex:
                 ],
                 dParams,
                 _account,
```


2. #### We can memoize the `_hTokens.length` computation.
- https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L871
```solidity
file: src/RootBridgeAgent.sol

856:    function _performRetrySettlementCall(
857:        bool _hasFallbackToggled,
858:        address[] memory _hTokens,
859:        address[] memory _tokens,
860:        uint256[] memory _amounts,
861:        uint256[] memory _deposits,
862:        bytes memory _params,
863:        uint32 _settlementNonce,
864:        address payable _refundee,
865:        address _recipient,
866:        uint16 _dstChainId,
867:        GasParams memory _gParams,
868:        uint256 _value
869:    ) internal {
870:        // Check if payload is ready for message
871:        if (_hTokens.length == 0) revert SettlementRetryUnavailableUseCallout();   // @audit memoize computation
872:
873:        // Get packed data
874:        bytes memory payload;
875:
876:        // Check if it's a single asset settlement
877:        if (_hTokens.length == 1) {    // @audit same computation
878:            //Pack new Data
879:            payload = abi.encodePacked(
880:                _hasFallbackToggled ? bytes1(0x81) : bytes1(0x01),
881:                _recipient,
882:                _settlementNonce,
883:                _hTokens[0],
884:                _tokens[0],
885:                _amounts[0],
886:                _deposits[0],
887:                _params
888:            );
889:
890:            // Check if it's mulitple asset settlement
891:        } else if (_hTokens.length > 1) { // @audit same computation
892:            //Pack new Data
893:            payload = abi.encodePacked(
894:                _hasFallbackToggled ? bytes1(0x82) : bytes1(0x02),
895:                _recipient,
896:                uint8(_hTokens.length),
897:                _settlementNonce,
898:                _hTokens,
899:                _tokens,
900:                _amounts,
901:                _deposits,
902:                _params
903:            );
904:        }
.
.
.
931:    }
```
In the `_performRetrySettlementCall()` function above the computation `_hTokens.length` was repeated multiple times. We could save the gas used in the subsequent computation if we memoize the computation i.e we cache the result of the calcultion the first time in a variable and use the variable in place of the subsequent calculations so that in scenarios where the `_hTokens.length` computation would be calculated we would have reduce gas cost for the computation to just a stack read. The diff below shows how the code could be refactored: 
```diff
diff --git a/src/RootBridgeAgent.sol b/src/RootBridgeAgent.sol                            
index a6ad0ef..eadd8d5 100644                                                             
--- a/src/RootBridgeAgent.sol                                                             
+++ b/src/RootBridgeAgent.sol                                                             
@@ -867,14 +867,15 @@ contract RootBridgeAgent is IRootBridgeAgent, BridgeAgentConstants {
         GasParams memory _gParams,                                                       
         uint256 _value                                                                   
     ) internal {                                                                         
+        uint256 length = _hTokens.length;                                                
         // Check if payload is ready for message                                         
-        if (_hTokens.length == 0) revert SettlementRetryUnavailableUseCallout();         
+        if (length == 0) revert SettlementRetryUnavailableUseCallout();                  
                                                                                          
         // Get packed data                                                               
         bytes memory payload;                                                            
                                                                                          
         // Check if it's a single asset settlement                                       
-        if (_hTokens.length == 1) {                                                      
+        if (length == 1) {                                                               
             //Pack new Data                                                              
             payload = abi.encodePacked(                                                  
                 _hasFallbackToggled ? bytes1(0x81) : bytes1(0x01),                       
@@ -888,12 +889,12 @@ contract RootBridgeAgent is IRootBridgeAgent, BridgeAgentConstants {
             );                                                                           
                                                                                          
             // Check if it's mulitple asset settlement                                   
-        } else if (_hTokens.length > 1) {                                                
+        } else if (length > 1) {                                                         
             //Pack new Data                                                              
             payload = abi.encodePacked(                                                  
                 _hasFallbackToggled ? bytes1(0x82) : bytes1(0x02),                       
                 _recipient,                                                              
-                uint8(_hTokens.length),                                                  
+                uint8(length),                                                           
                 _settlementNonce,                                                        
                 _hTokens,                                                                
                 _tokens,                                                                 
```


3. #### We can memoize the `uint8(deposit.hTokens.length)` computation.
- https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L359
```solidity
file: src/BranchBridgeAgent.sol

343:    function retryDeposit(
344:        bool _isSigned,
345:        uint32 _depositNonce,
346:        bytes calldata _params,
347:        GasParams calldata _gParams,
348:        bool _hasFallbackToggled
349:    ) external payable override lock {
.
.
.
359:        if (uint8(deposit.hTokens.length) == 1) {   //@audit memoize computation
360:            if (_isSigned) {
361:                //Pack new Data
362:                payload = abi.encodePacked(
363:                    _hasFallbackToggled ? bytes1(0x85) : bytes1(0x05),
364:                    msg.sender,
365:                    _depositNonce,
366:                    deposit.hTokens[0],
367:                    deposit.tokens[0],
368:                    deposit.amounts[0],
369:                    deposit.deposits[0],
370:                    _params
371:                );
372:            } else {
373:                payload = abi.encodePacked(
374:                    bytes1(0x02),
375:                    _depositNonce,
376:                    deposit.hTokens[0],
377:                    deposit.tokens[0],
378:                    deposit.amounts[0],
379:                    deposit.deposits[0],
380:                    _params
381:                );
382:            }
383:        } else if (uint8(deposit.hTokens.length) > 1) { //@audit same computation
384:            if (_isSigned) {
385:                //Pack new Data
386:                payload = abi.encodePacked(
387:                    _hasFallbackToggled ? bytes1(0x86) : bytes1(0x06),
388:                    msg.sender,
389:                    uint8(deposit.hTokens.length),  //@audit same computation
390:                    _depositNonce,
391:                    deposit.hTokens,
392:                    deposit.tokens,
393:                    deposit.amounts,
394:                    deposit.deposits,
395:                    _params
396:                );
397:            } else {
398:                payload = abi.encodePacked(
399:                    bytes1(0x03),
400:                    uint8(deposit.hTokens.length),  //@audit same computation
401:                    _depositNonce,
402:                    deposit.hTokens,
403:                    deposit.tokens,
404:                    deposit.amounts,
405:                    deposit.deposits,
406:                    _params
407:                );
408:            }
409:        }
.
.
.
419:    }
```
In the `retryDeposit()` function above the computation `uint8(deposit.hTokens.length)` was repeated multiple times. We could save the gas used in the subsequent computation if we memoize the computation i.e we cache the result of the calcultion the first time in a variable and use the variable in place of the subsequent calculations so that in scenarios where the `uint8(deposit.hTokens.length)` computation would be calculated we would have reduce gas cost for the computation to just a stack read. The diff below shows how the code could be refactored:
```diff
diff --git a/src/BranchBridgeAgent.sol b/src/BranchBridgeAgent.sol
index b076d2d..bd6cc7c 100644
--- a/src/BranchBridgeAgent.sol
+++ b/src/BranchBridgeAgent.sol
@@ -356,7 +356,9 @@ contract BranchBridgeAgent is IBranchBridgeAgent, BridgeAgentConstants {
         //Encode Data for cross-chain call.
         bytes memory payload;

-        if (uint8(deposit.hTokens.length) == 1) {
+        uint8 length = uint8(deposit.hTokens.length);
+
+        if (length == 1) {
             if (_isSigned) {
                 //Pack new Data
                 payload = abi.encodePacked(
@@ -380,13 +382,13 @@ contract BranchBridgeAgent is IBranchBridgeAgent, BridgeAgentConstants {
                     _params
                 );
             }
-        } else if (uint8(deposit.hTokens.length) > 1) {
+        } else if (length > 1) {
             if (_isSigned) {
                 //Pack new Data
                 payload = abi.encodePacked(
                     _hasFallbackToggled ? bytes1(0x86) : bytes1(0x06),
                     msg.sender,
-                    uint8(deposit.hTokens.length),
+                    length,
                     _depositNonce,
                     deposit.hTokens,
                     deposit.tokens,
@@ -397,7 +399,7 @@ contract BranchBridgeAgent is IBranchBridgeAgent, BridgeAgentConstants {
             } else {
                 payload = abi.encodePacked(
                     bytes1(0x03),
-                    uint8(deposit.hTokens.length),
+                    length,
                     _depositNonce,
                     deposit.hTokens,
                     deposit.tokens,
```


## [G-07]  State/Storage variables should be cached in stack variables rather than re-reading them from storage.
The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

### 1 Instance
1. #### Cache `settlementReference.owner` to avoid multiple storage reads
- https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L247
```solidity
file: src/RootBridgeAgent.sol

233:    function retrySettlement(
234:        uint32 _settlementNonce,
235:        address _recipient,
236:        bytes calldata _params,
237:        GasParams calldata _gParams,
238:        bool _hasFallbackToggled
239:    ) external payable override lock {
240:        // Get storage reference
241:        Settlement storage settlementReference = getSettlement[_settlementNonce];
242:
243:        // Check if Settlement hasn't been redeemed.
244:        if (settlementReference.owner == address(0)) revert SettlementRetryUnavailable();
245:
246:        // Check if caller is Settlement owner
247:        if (msg.sender != settlementReference.owner) {
248:            if (msg.sender != address(IPort(localPortAddress).getUserAccount(settlementReference.owner))) {
249:                revert NotSettlementOwner();
250:            }
251:        }
252:
253:        // Update Settlement Status
254:        settlementReference.status = STATUS_SUCCESS;
255:
256:        // Perform Settlement Retry
257:        _performRetrySettlementCall(
258:            _hasFallbackToggled,
259:            settlementReference.hTokens,
260:            settlementReference.tokens,
261:            settlementReference.amounts,
262:            settlementReference.deposits,
263:            _params,
264:            _settlementNonce,
265:            payable(settlementReference.owner),
266:            _recipient,
267:            settlementReference.dstChainId,
268:            _gParams,
269:            msg.value
270:        );
271:    }
```

```diff
diff --git a/src/RootBridgeAgent.sol b/src/RootBridgeAgent.sol                                               
index a6ad0ef..031ea0a 100644                                                                                
--- a/src/RootBridgeAgent.sol                                                                                
+++ b/src/RootBridgeAgent.sol                                                                                
@@ -240,12 +240,14 @@ contract RootBridgeAgent is IRootBridgeAgent, BridgeAgentConstants {                   
         // Get storage reference                                                                            
         Settlement storage settlementReference = getSettlement[_settlementNonce];                           
                                                                                                             
+        address settlementReferenceOwner = settlementReference.owner;                                       
+                                                                                                            
         // Check if Settlement hasn't been redeemed.                                                        
-        if (settlementReference.owner == address(0)) revert SettlementRetryUnavailable();                   
+        if (settlementReferenceOwner == address(0)) revert SettlementRetryUnavailable();                    
                                                                                                             
         // Check if caller is Settlement owner                                                              
-        if (msg.sender != settlementReference.owner) {                                                      
-            if (msg.sender != address(IPort(localPortAddress).getUserAccount(settlementReference.owner))) { 
+        if (msg.sender != settlementReferenceOwner) {                                                       
+            if (msg.sender != address(IPort(localPortAddress).getUserAccount(settlementReferenceOwner))) {  
                 revert NotSettlementOwner();                                                                
             }                                                                                               
         }                                                                                                   
@@ -262,7 +264,7 @@ contract RootBridgeAgent is IRootBridgeAgent, BridgeAgentConstants {                     
             settlementReference.deposits,                                                                   
             _params,                                                                                        
             _settlementNonce,                                                                               
-            payable(settlementReference.owner),                                                             
+            payable(settlementReferenceOwner),                                                              
             _recipient,                                                                                     
             settlementReference.dstChainId,                                                                 
             _gParams,                                                                                       
```
#### Gas saving for `retrySettlement()` function obtained via protocol test: Avg 6 gas
|  | Min | Max | Avg | # Calls |
|----------|----------|----------|----------|----------|
| Before | 83561 | 127861 | 113094 | 3 |
| After | 83555 | 127855 | 113088 | 3 |

#### Please note these Instances were not included in the bot report.




## [G-08] Initializers can be marked as payable to save deployment gas
Payable functions cost less gas to execute, because the compiler does not have to add extra checks to ensure that no payment is provided. Initializers can be safely marked as payable, because only the deployer or the factory contract would call the function without carrying any funds.

### 9 Instances
1. #### Make `MulticallRootRouter.initialize()` function a `payable` function
- https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L109
```solidity
file: src/MulticallRootRouter.sol

109:    function initialize(address _bridgeAgentAddress) external onlyOwner {  // @audit can be made payable
110:        require(_bridgeAgentAddress != address(0), "Bridge Agent Address cannot be 0");
111:
112:        bridgeAgentAddress = payable(_bridgeAgentAddress);
113:        bridgeAgentExecutorAddress = IBridgeAgent(_bridgeAgentAddress).bridgeAgentExecutorAddress();
114:        renounceOwnership();
115:    }
```
We could reduce the gas usage of the `MulticallRootRouter.initialize()` function by making the function a payable function. This can be done as shown in the diff below:
```diff
diff --git a/src/MulticallRootRouter.sol b/src/MulticallRootRouter.sol
index 0621f81..52153fc 100644
--- a/src/MulticallRootRouter.sol
+++ b/src/MulticallRootRouter.sol
@@ -106,7 +106,7 @@ contract MulticallRootRouter is IRootRouter, Ownable {
      * @notice Initializes the Multicall Root Router.
      * @param _bridgeAgentAddress The address of the Bridge Agent.
      */
-    function initialize(address _bridgeAgentAddress) external onlyOwner {
+    function initialize(address _bridgeAgentAddress) external payable onlyOwner {
         require(_bridgeAgentAddress != address(0), "Bridge Agent Address cannot be 0");

         bridgeAgentAddress = payable(_bridgeAgentAddress);
```
#### Gas saving for `MulticallRootRouter.initialize()` function obtained via protocol test: Avg 12 gas
|  | Min | Max | Avg | # Calls |
|----------|----------|----------|----------|----------|
| Before | 38147 | 38147 | 38147 | 72 |
| After | 38135 | 38135 | 38135 | 72 |


2. #### Make `CoreRootRouter.initialize()` function a `payable` function
- https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L83
```solidity
file: src/CoreRootRouter.sol

83:    function initialize(address _bridgeAgentAddress, address _hTokenFactory) external onlyOwner { // @audit can be made payable
84:        require(_setup, "Contract is already initialized");
85:        _setup = false;
86:        bridgeAgentAddress = payable(_bridgeAgentAddress);
87:        bridgeAgentExecutorAddress = IBridgeAgent(_bridgeAgentAddress).bridgeAgentExecutorAddress();
88:        hTokenFactoryAddress = _hTokenFactory;
89:    }
```
We could reduce the gas usage of the `CoreRootRouter.initialize()` function by making the function a payable function. This can be done as shown in the diff below:
```diff
diff --git a/src/CoreRootRouter.sol b/src/CoreRootRouter.sol
index 3104a2a..39c48b8 100644
--- a/src/CoreRootRouter.sol
+++ b/src/CoreRootRouter.sol
@@ -80,7 +80,7 @@ contract CoreRootRouter is IRootRouter, Ownable {
                     INITIALIZATION FUNCTIONS
     //////////////////////////////////////////////////////////////*/

-    function initialize(address _bridgeAgentAddress, address _hTokenFactory) external onlyOwner {
+    function initialize(address _bridgeAgentAddress, address _hTokenFactory) external payable onlyOwner {
         require(_setup, "Contract is already initialized");
         _setup = false;
         bridgeAgentAddress = payable(_bridgeAgentAddress);
```
#### Gas saving for `CoreRootRouter.initialize()` function obtained via protocol test: Avg 15 gas
|  | Min | Max | Avg | # Calls |
|----------|----------|----------|----------|----------|
| Before | 45997 | 45997 | 45997 | 76 |
| After | 45982 | 45982 | 45982 | 76 |


3. #### Make `BranchPort.initialize()` function a `payable` function
- https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L122
```solidity
file: src/BranchPort.sol

122:    function initialize(address _coreBranchRouter, address _bridgeAgentFactory) external virtual onlyOwner {  //@audit can be made payable
123:        require(coreBranchRouterAddress == address(0), "Contract already initialized");
124:        require(!isBridgeAgentFactory[_bridgeAgentFactory], "Contract already initialized");
125:
126:        require(_coreBranchRouter != address(0), "CoreBranchRouter is zero address");
127:        require(_bridgeAgentFactory != address(0), "BridgeAgentFactory is zero address");
128:
129:        coreBranchRouterAddress = _coreBranchRouter;
130:        isBridgeAgentFactory[_bridgeAgentFactory] = true;
131:        bridgeAgentFactories.push(_bridgeAgentFactory);
132:    }
```
We could reduce the gas usage of the `BranchPort.initialize()` function by making the function a payable function. This can be done as shown in the diff below:
```diff
diff --git a/src/BranchPort.sol b/src/BranchPort.sol
index ba60acc..0485a56 100644
--- a/src/BranchPort.sol
+++ b/src/BranchPort.sol
@@ -119,7 +119,7 @@ contract BranchPort is Ownable, IBranchPort {
      *   @param _coreBranchRouter Address of the Core Branch Router.
      *   @param _bridgeAgentFactory Address of the Bridge Agent Factory.
      */
-    function initialize(address _coreBranchRouter, address _bridgeAgentFactory) external virtual onlyOwner {
+    function initialize(address _coreBranchRouter, address _bridgeAgentFactory) external payable virtual onlyOwner {
         require(coreBranchRouterAddress == address(0), "Contract already initialized");
         require(!isBridgeAgentFactory[_bridgeAgentFactory], "Contract already initialized");
```
#### Gas saving for `BranchPort.initialize()` function obtained via protocol test: Avg 13 gas
|  | Min | Max | Avg | # Calls |
|----------|----------|----------|----------|----------|
| Before | 89623 | 89623 | 89623 | 70 |
| After | 89610 | 89610 | 89610 | 70 |


4. #### Make `BaseBranchRouter.initialize()` function a `payable` function
- https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L60
```solidity
file: src/BaseBranchRouter.sol

60:    function initialize(address _localBridgeAgentAddress) external onlyOwner {  //@audit can be made payable
61:        require(_localBridgeAgentAddress != address(0), "Bridge Agent address cannot be 0");
62:        localBridgeAgentAddress = _localBridgeAgentAddress;
63:        localPortAddress = IBridgeAgent(_localBridgeAgentAddress).localPortAddress();
64:        bridgeAgentExecutorAddress = IBridgeAgent(_localBridgeAgentAddress).bridgeAgentExecutorAddress();
65:        
66:        renounceOwnership();
67:    }
```
We could reduce the gas usage of the `BaseBranchRouter.initialize()` function by making the function a payable function. This can be done as shown in the diff below:
```diff
diff --git a/src/BaseBranchRouter.sol b/src/BaseBranchRouter.sol
index 62bc287..0944f8a 100644
--- a/src/BaseBranchRouter.sol
+++ b/src/BaseBranchRouter.sol
@@ -57,7 +57,7 @@ contract BaseBranchRouter is IBranchRouter, Ownable {
      * @notice Initializes the Base Branch Router.
      * @param _localBridgeAgentAddress The address of the local Bridge Agent.
      */
-    function initialize(address _localBridgeAgentAddress) external onlyOwner {
+    function initialize(address _localBridgeAgentAddress) external payable onlyOwner {
         require(_localBridgeAgentAddress != address(0), "Bridge Agent address cannot be 0");
         localBridgeAgentAddress = _localBridgeAgentAddress;
         localPortAddress = IBridgeAgent(_localBridgeAgentAddress).localPortAddress();
```
#### Gas saving for `BaseBranchRouter.initialize()` function obtained via protocol test: Avg 12 gas
|  | Min | Max | Avg | # Calls |
|----------|----------|----------|----------|----------|
| Before | 56580 | 56580 | 56580 | 150 |
| After | 56568 | 56568 | 56568 | 150 |


5. #### Make `RootPort.initialize()` function a `payable` function
- https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L129
```solidity
file: src/RootPort.sol

129:    function initialize(address _bridgeAgentFactory, address _coreRootRouter) external onlyOwner { // @audit can be made payable
130:        require(_bridgeAgentFactory != address(0), "Bridge Agent Factory cannot be 0 address.");
131:        require(_coreRootRouter != address(0), "Core Root Router cannot be 0 address.");
132:        require(_setup, "Setup ended.");
133:        _setup = false;
134:
135:        isBridgeAgentFactory[_bridgeAgentFactory] = true;
136:        bridgeAgentFactories.push(_bridgeAgentFactory);
137:
138:        coreRootRouterAddress = _coreRootRouter;
139:    }
```
We could reduce the gas usage of the `RootPort.initialize()` function by making the function a payable function. This can be done as shown in the diff below:
```diff
diff --git a/src/RootPort.sol b/src/RootPort.sol
index c379b6e..665b646 100644
--- a/src/RootPort.sol
+++ b/src/RootPort.sol
@@ -126,7 +126,7 @@ contract RootPort is Ownable, IRootPort {
      *   @param _bridgeAgentFactory The address of the Bridge Agent Factory.
      *   @param _coreRootRouter The address of the Core Root Router.
      */
-    function initialize(address _bridgeAgentFactory, address _coreRootRouter) external onlyOwner {
+    function initialize(address _bridgeAgentFactory, address _coreRootRouter) external payable onlyOwner {
         require(_bridgeAgentFactory != address(0), "Bridge Agent Factory cannot be 0 address.");
         require(_coreRootRouter != address(0), "Core Root Router cannot be 0 address.");
         require(_setup, "Setup ended.");
```
#### Gas saving for `RootPort.initialize()` function obtained via protocol test: Avg 14 gas
|  | Min | Max | Avg | # Calls |
|----------|----------|----------|----------|----------|
| Before | 89714 | 89714 | 89714 | 76 |
| After | 89700 | 89700 | 89700 | 76 |


6. #### Make `BranchBridgeAgentFactory.initialize()` function a `payable` function
- https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/BranchBridgeAgentFactory.sol#L87
```solidity
file: src/factories/BranchBridgeAgentFactory.sol

87:     function initialize(address _coreRootBridgeAgent) external virtual onlyOwner { // @audit can be made payable
88:         require(_coreRootBridgeAgent != address(0), "Core Root Bridge Agent cannot be 0");
89: 
90:         address newCoreBridgeAgent = address(
91:             DeployBranchBridgeAgent.deploy(
92:                 rootChainId,
93:                 localChainId,
94:                 _coreRootBridgeAgent,
95:                 lzEndpointAddress,
96:                 localCoreBranchRouterAddress,
97:                 localPortAddress
98:             )
99:         );
100: 
101:        IPort(localPortAddress).addBridgeAgent(newCoreBridgeAgent);
102:
103:        renounceOwnership();
104:    }
```
We could reduce the gas usage of the `BranchBridgeAgentFactory.initialize()` function by making the function a payable function. This can be done as shown in the diff below:
```diff
diff --git a/src/factories/BranchBridgeAgentFactory.sol b/src/factories/BranchBridgeAgentFactory.sol
index c789a27..0ba4396 100644
--- a/src/factories/BranchBridgeAgentFactory.sol
+++ b/src/factories/BranchBridgeAgentFactory.sol
@@ -84,7 +84,7 @@ contract BranchBridgeAgentFactory is Ownable, IBranchBridgeAgentFactory {
      * @notice Function to initialize the contract.
      * @param _coreRootBridgeAgent Address of the Root Chain's Core Root Bridge Agent.
      */
-    function initialize(address _coreRootBridgeAgent) external virtual onlyOwner {
+    function initialize(address _coreRootBridgeAgent) external payable virtual onlyOwner {
         require(_coreRootBridgeAgent != address(0), "Core Root Bridge Agent cannot be 0");

         address newCoreBridgeAgent = address(
```
```
Estimated gas saved: 14 gas units
```


7. #### Make `ERC20hTokenBranchFactory.initialize()` function a `payable` function
- https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenBranchFactory.sol#L60
```solidity
file: src/factories/ERC20hTokenBranchFactory.sol

60:    function initialize(address _wrappedNativeTokenAddress, address _coreRouter) external onlyOwner { // @audit can be made payable
61:        require(_coreRouter != address(0), "CoreRouter address cannot be 0");
62:
63:        ERC20hTokenBranch newToken = new ERC20hTokenBranch(
64:            chainName,
65:            chainSymbol,
66:            ERC20(_wrappedNativeTokenAddress).name(),
67:            ERC20(_wrappedNativeTokenAddress).symbol(),
68:            ERC20(_wrappedNativeTokenAddress).decimals(),
69:            localPortAddress
70:        );
71:
72:        hTokens.push(newToken);
73:
74:        localCoreRouterAddress = _coreRouter;
75:
76:        renounceOwnership();
77:    }
```
We could reduce the gas usage of the `ERC20hTokenBranchFactory.initialize()` function by making the function a payable function. This can be done as shown in the diff below:
```diff
diff --git a/src/factories/ERC20hTokenBranchFactory.sol b/src/factories/ERC20hTokenBranchFactory.sol
index abcccb7..dab9732 100644
--- a/src/factories/ERC20hTokenBranchFactory.sol
+++ b/src/factories/ERC20hTokenBranchFactory.sol
@@ -57,7 +57,7 @@ contract ERC20hTokenBranchFactory is Ownable, IERC20hTokenBranchFactory {
      * @param _wrappedNativeTokenAddress Address of the Local Chain's wrapped native token.
      * @param _coreRouter Address of the Local Chain's Core Router.
      */
-    function initialize(address _wrappedNativeTokenAddress, address _coreRouter) external onlyOwner {
+    function initialize(address _wrappedNativeTokenAddress, address _coreRouter) external payable onlyOwner {
         require(_coreRouter != address(0), "CoreRouter address cannot be 0");

         ERC20hTokenBranch newToken = new ERC20hTokenBranch(
```
#### Gas saving for `ERC20hTokenBranchFactory.initialize()` function obtained via protocol test: Avg 13 gas
|  | Min | Max | Avg | # Calls |
|----------|----------|----------|----------|----------|
| Before | 918378 | 918378 | 918378 | 52 |
| After | 918365 | 918365 | 918365 | 52 |


8. #### Make `ERC20hTokenRootFactory.initialize()` function a `payable` function
- https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenRootFactory.sol#L49
```solidity
file: src/factories/ERC20hTokenRootFactory.sol

49:    function initialize(address _coreRouter) external onlyOwner { //  @audit can be made payable
50:        require(_coreRouter != address(0), "CoreRouter address cannot be 0");
51:        coreRootRouterAddress = _coreRouter;
52:        renounceOwnership();
53:    }
```
We could reduce the gas usage of the `ERC20hTokenRootFactory.initialize()` function by making the function a payable function. This can be done as shown in the diff below:
```diff
diff --git a/src/factories/ERC20hTokenRootFactory.sol b/src/factories/ERC20hTokenRootFactory.sol
index b3c6c50..741b141 100644
--- a/src/factories/ERC20hTokenRootFactory.sol
+++ b/src/factories/ERC20hTokenRootFactory.sol
@@ -46,7 +46,7 @@ contract ERC20hTokenRootFactory is Ownable, IERC20hTokenRootFactory {
      * @notice Function to initialize the contract.
      * @param _coreRouter Address of the Root Chain's Core Router.
      */
-    function initialize(address _coreRouter) external onlyOwner {
+    function initialize(address _coreRouter) external payable onlyOwner {
         require(_coreRouter != address(0), "CoreRouter address cannot be 0");
         coreRootRouterAddress = _coreRouter;
         renounceOwnership();
```
#### Gas saving for `ERC20hTokenRootFactory.initialize()` function obtained via protocol test: Avg 12 gas
|  | Min | Max | Avg | # Calls |
|----------|----------|----------|----------|----------|
| Before | 19638 | 19638 | 19638 | 76 |
| After | 19626 | 19626 | 19626 | 76 |


9. #### Make `ArbitrumBranchBridgeAgentFactory.initialize()` function a `payable` function
- https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ArbitrumBranchBridgeAgentFactory.sol#L56
```solidity
file: src/factories/ArbitrumBranchBridgeAgentFactory.sol

56:    function initialize(address _coreRootBridgeAgent) external override onlyOwner { //  @audit can be made payable
57:        require(_coreRootBridgeAgent != address(0), "Core Root Bridge Agent Address cannot be 0");
58:
59:        address newCoreBridgeAgent = address(
60:            DeployArbitrumBranchBridgeAgent.deploy(
61:                rootChainId, _coreRootBridgeAgent, localCoreBranchRouterAddress, localPortAddress
62:            )
63:        );
64:
65:        IPort(localPortAddress).addBridgeAgent(newCoreBridgeAgent);
66:
67:        renounceOwnership();
68:    }
```
We could reduce the gas usage of the `ArbitrumBranchBridgeAgentFactory.initialize()` function by making the function a payable function. This can be done as shown in the diff below:
```diff
diff --git a/src/factories/ArbitrumBranchBridgeAgentFactory.sol b/src/factories/ArbitrumBranchBridgeAgentFactory.sol
index 3f07a7d..fbe8c1c 100644
--- a/src/factories/ArbitrumBranchBridgeAgentFactory.sol
+++ b/src/factories/ArbitrumBranchBridgeAgentFactory.sol
@@ -53,7 +53,7 @@ contract ArbitrumBranchBridgeAgentFactory is BranchBridgeAgentFactory {
      * @notice Initializes the Bridge Agent Factory Contract.
      * @param _coreRootBridgeAgent Address of the Core Root Bridge Agent.
      */
-    function initialize(address _coreRootBridgeAgent) external override onlyOwner {
+    function initialize(address _coreRootBridgeAgent) external payable override onlyOwner {
         require(_coreRootBridgeAgent != address(0), "Core Root Bridge Agent Address cannot be 0");

         address newCoreBridgeAgent = address(
```
#### Gas saving for `ArbitrumBranchBridgeAgentFactory.initialize()` function obtained via protocol test: Avg 15 gas
|  | Min | Max | Avg | # Calls |
|----------|----------|----------|----------|----------|
| Before | 3406778 | 3406778 | 3406778 | 26 |
| After | 3406763 | 3406763 | 3406763 | 26 |




## [G-09] Unbounded Gas Consumption Risk Due to External Call Recipients
In the context of Solidity, external function calls without a specified gas limit present a significant risk. The callee contract has the potential to consume all the gas allocated to the transaction, causing an undesired revert and disrupt the function's execution. To mitigate this, it's recommended to explicitly set a gas limit when performing external calls using addr.call{gas: }. This limits the gas forwarded to the callee, preventing potential pitfalls and offering better control over transaction execution.

### 6 Instances
- https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchBridgeAgent.sol#L103
- https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L754
- https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L779
- https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L927
- https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L717
- https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L737




## [G-10] Multiplication by two should use bit shifting
x * 2 is the same as x << 1. While the compiler uses the SHL opcode to accomplish both, the version that uses multiplication incurs an overhead of 20 gas due to JUMPs to and from a compiler utility function that introduces checks which can be avoided by using unchecked {} around the division by two.

### 22 Instances
- https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L125
- https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L134
- https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L137
- https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L213
- https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L222
- https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L228
- https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L291
- https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L292
- https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L304
- https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L305
- https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L315
- https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L316
- https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L317
- https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L325
- https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L523
- https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L524
- https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L536
- https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L537
- https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L547
- https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#548
- https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L556
- https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L




## CONCLUSION
As you embark on incorporating the recommended optimizations, we want to emphasize the utmost importance of proceeding with vigilance and dedicating thorough efforts to comprehensive testing. It is of paramount significance to ensure that the proposed alterations do not inadvertently introduce fresh vulnerabilities, while also successfully achieving the anticipated enhancements in performance.

We strongly advise conducting a meticulous and exhaustive evaluation of the modifications made to the codebase. This rigorous scrutiny and exhaustive assessment will play a pivotal role in affirming both the security and efficacy of the refactored code. Your careful attention to detail, coupled with the implementation of a robust testing framework, will provide the necessary assurance that the refined code aligns with your security objectives and effectively fulfills the intended performance optimizations.