## [L-01] VirtualAccount#withdrawNative should use ```forceSafeTransferETH``` for DoS Protection 

#### Proof of Concept 
It is stated in ```safeTransferLib``` by solady : 
``` 
/// @dev Note:
/// - For ETH transfers, please use `forceSafeTransferETH` for DoS protection.
```
But this recommendation is not implemented in ```VirtualAccount#withdrawNative```.
``` solidity 
function withdrawNative(uint256 _amount) external override requiresApprovedCaller {
        msg.sender.safeTransferETH(_amount);    //@audit-issue not used forceSafeTransferEth for DoS Protection
    }
```
#### Recommended Mitigation 
```forceSafeTransferEth``` should be used instead of safeTransferEth for native tokens.
Mitigated Code : 
``` solidity 
function withdrawNative(uint256 _amount) external override requiresApprovedCaller {
        -msg.sender.safeTransferETH(_amount); 
        +msg.sender.forceSafeTransferETH(_amount);
    }
```
#### Relevant Github Link 
[VirtualAccount.sol#L52](https://github.com/code-423n4/2023-09-maia/blob/c0dc3550e0754571b82d7bfd8f0282ac8fa5e42f/src/VirtualAccount.sol#L52)
  