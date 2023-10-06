** Issue #1 **
BranchBridgeAgent.sol : line: 447 
Switch for loop to while loop
Example:
``		
        uint256 i = 0;
        uint256 len =  deposit.tokens.length;
        while (i < len) {
           
            // execute code
            unchecked {
                ++i;
            }
        }

``

**Issue #2**
BranchBridgeAgent.sol | line: 513 
Switch for loop to while loop
Example:
```
        uint256 i = 0;
        uint256 len =  numOfAssets;
        while (i < len) {
           
            // execute code
            unchecked {
                ++i;
            }
        }
```

**Issue #3**
BaseBranchRouter.sol | line: 192
Switch for loop to while loop
Example:
```
        uint256 i = 0;
        uint256 len =  _hTokens.length;
        while (i < len) {
           
            // execute code
            unchecked {
                ++i;
            }
        }
```

**Issue #4**

RootBridgeAgent.sol | line: 318 
Switch for loop to while loop
Example:
```
        uint256 i = 0;
        uint256 len =  settlement.hTokens.length;
        while (i < len) {
           
            // execute code
            unchecked {
                ++i;
            }
        }
```

**Issue #4**
RootBridgeAgent.sol | line: 399 
Switch for loop to while loop
Example:
```
        uint256 i = 0;
        uint256 len =  _dParams.hTokens.length;
        while (i < len) {
           
            // execute code
            unchecked {
                ++i;
            }
        }
```

**Issue #5**
RootBridgeAgentExecutor.sol | line: 281 
Switch for loop to while loop
Example:
```
        uint256 i = 0;
        uint256 len =  uint256(uint8(numOfAssets));
        while (i < len) {
           
            // execute code
            unchecked {
                ++i;
            }
        }
```

**Issue #6**
VirtualAccount.sol | line: 70
Switch for loop to while loop
Example:
```
        uint256 i = 0;
        uint256 len =  calls.length;
        while (i < len) {
           
            // execute code
            unchecked {
                ++i;
            }
        }
```

**Issue #7**
VirtualAccount.sol | line: 90 
Switch for loop to while loop
Example:
```
        uint256 i = 0;
        uint256 len =  calls.length;
        while (i < len) {
           
            // execute code
            unchecked {
                ++i;
            }
        }
```
**Issue #8**
**ERC20hTokenBranchFactory.createToken**
Modifier **requiresCoreRouterOrPort** only called once.
Cheaper to include the code inside the function. In addition, makes code more readable for developers.

**Issue #9** 
**ERC20hTokenRootFactory.createToken**
Modifier **requiresCoreRouter** only called once.
Cheaper to include the code inside the function. In addition, makes code more readable for developers. 

**Issue #10**
**BranchBridgeAgent.callOutSystem**
Modifier **requiresRouteronly** called once.
Cheaper to include the code inside the function. In addition, makes code more readable for developers. 

**Issue #11**
**BranchPort.addBridgeAgent** 
Modifier **requiresBridgeAgentFactory** only called once
Cheaper to include the code inside the function. In addition, makes code more readable for developers

**Issue #12**
**BranchPort.manage**
Modifier **requiresPortStrategey** only called once
Cheaper to include the code inside the function. In addition, makes code more readable for developers

**Issue #13**
**RootBridgeAgent.approveBranchBridgeAgent**
Modifier **requireManager** only called once
Cheaper to include the code inside the function. In addition, makes code more readable for developers

**Issue #14**
**RootBridgeAgent.syncBranchBridgeAgent**
Modifier **requirePort** only called once
Cheaper to include the code inside the function. In addition, makes code more readable for developers

**Issue #15**
**RootBridgeAgent.lsRecieveNonBlocking**
Modifier **requiresEndpoint** only called once
Cheaper to include the code inside the function. In addition, makes code more readable for developers

**Issue #16**
**RootPort.addBridgeAgent**
Modifier **requiresBridgeAgentFactory** only called once
Cheaper to include the code inside the function. In addition, makes code more readable for developers

















