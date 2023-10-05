## Summary

|                        | Issues                                                           | Instances |
| ---------------------- | ---------------------------------------------------------------- | --------- |
| [[N-0](#nonCritical1)] | Typos                                                            | 2         |
| [[N-1](#nonCritical2)] | Incorrect comment in CoreRootRouter                              | 6         |
| [[N-2](#nonCritical3)] | Solidity best practice for naming internal function not followed | 1         |

## Non Critical

---

### [N-0] Typos <a id="nonCritical1"></a>

_Instance 1:_

`is is` Used twice in the `ArbitrumCoreBranchRouter` Natspac

```Javascript
File: src/ArbitrumCoreBranchRouter.sol

19    * @dev    The function `addGlobalToken` is is not available since it's used
```

GitHub: [19](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/ArbitrumCoreBranchRouter.sol#L19)

_Instance 2:_

`is` is used in the following line of code that is grammatically incorrect

```Javascript
File: src/interfaces/ICoreBranchRouter.sol

10    *         This contract is allows users to permissionlessly add new tokens
```

GitHub: [10](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/interfaces/ICoreBranchRouter.sol#L10)

### [N-1] Incorrect/Unnecessary comment in following CoreRootRouter functions <a id="nonCritical2"></a>

There is no use for this comment. Either change it according to the function or remove it

_Instances [Total 6]_

```Javascript
File: src/CoreRootRouter.sol

138         //Add new global token to branch chain

173         //Add new global token to branch chain

198         //Add new global token to branch chain

225         //Add new global token to branch chain

256         //Add new global token to branch chain

286         //Add new global token to branch chain
```

Github: [138](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/CoreRootRouter.sol#L138), [173](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/CoreRootRouter.sol#L173), [198](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/CoreRootRouter.sol#L198), [225](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/CoreRootRouter.sol#L225), [256](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/CoreRootRouter.sol#L256), [286](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/CoreRootRouter.sol#L286)

### [N-2] Solidity best practice for naming internal function not followed <a id="nonCritical3"></a>

According to solidity guidelines, the name of the internal function should start with and underscore (`_`). This is not followed in `RooPort::addVirtualAccount(...)` function

_instance:_

```Javascript
359    function addVirtualAccount(address _user) internal returns (VirtualAccount newAccount) {
```

GitHub: [359](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L359-L359)