Many functions uses the logic of (int - int > 0) in order the unchecked call does not overflow, e,g :
* https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L165
* https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L284
* https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L906
* https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L259
* https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol#L132

The if condition in that case will reverts if does not meet with no error given and the function call will fails, with no error given, this will results for confusing on users end, and will succeed (the call) only when the if condition meets, it's better to use another logic where if the "if" condition does not meet the call will succeed and not revert, and even if it does it will inform users why it has been revereted, example :

```
 if (_amount - _deposit > 0) {
            unchecked {
                IRootPort(rootPortAddress).bridgeToRootFromLocalBranch(_depositor, _localAddress, _amount - _deposit);
            }
        }
```

when _deposit > amount, the call will fails with no revert message, for it to be used properly, better user one of the these logics :

* if it will underflow does not pass to unchecked

```
if (_amount > _deposit ) { // this will never make "_deposit - _amount" underflow
            unchecked {
                IRootPort(rootPortAddress).bridgeToRootFromLocalBranch(_depositor, _localAddress, _amount - _deposit);
            }
        }
``` 

* if it will underflow, revert with error, so users know why it has been failed

```
require(_amount - _deposit > 0,"deposit is higher that amount");
unchecked {
                IRootPort(rootPortAddress).bridgeToRootFromLocalBranch(_depositor, _localAddress, _amount - _deposit);
            }
       
```
