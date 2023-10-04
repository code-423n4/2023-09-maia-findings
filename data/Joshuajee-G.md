## G-01: Duplicate arithmetic operation;
 “_amount - _deposit” was repeated 2 - 3 times in the following transactions costing more gas.

```sol
if (_amount - _deposit > 0) {
   unchecked {
       _hToken.safeTransferFrom(msg.sender, address(this), _amount - _deposit);
       ERC20(_hToken).approve(_localPortAddress, _amount - _deposit);
   }
}
```


Fix: 
Cache the difference between   “_amount - _deposit”, and use the cache value instead.

```sol
uint256 _hTokenAmount = _amount - _deposit;

if (_hTokenAmount > 0) {
     unchecked {
       _hToken.safeTransferFrom(msg.sender, address(this), _hTokenAmount);
       ERC20(_hToken).approve(_localPortAddress, _hTokenAmount);
     }
}
```



Affected codes
 https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L164C2-L170C10

https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol#L132C1-L136C10

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L906C1-L910C10

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1149C1-L1153C10

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L284C1-L288C10

## G-02: Use customer errors instead of revert to save gas;

Affected codes:
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol#L39

https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L61

https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L213

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L125C1-L131C100

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L923

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L109

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L123C1-L127C90

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L181
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L213

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L332C1-L333C81

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L567

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L84

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L278

https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L93C1-L94C83

https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L591

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L111C1-L113C93

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1191






