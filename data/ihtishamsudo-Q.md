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

## [NC-01] BranchPort#_checkTimeLimit lastmanage could be simplified 
#### Proof of Concept 
```solidity 
function _checkTimeLimit(address _token, uint256 _amount) internal {
        uint256 dailyLimit = strategyDailyLimitRemaining[msg.sender][_token];
        if (block.timestamp - lastManaged[msg.sender][_token] >= 1 days) {
            dailyLimit = strategyDailyLimitAmount[msg.sender][_token];
            unchecked {
                lastManaged[msg.sender][_token] = (block.timestamp / 1 days) * 1 days;
            }
        }
        strategyDailyLimitRemaining[msg.sender][_token] = dailyLimit - _amount;
    }
```
Above function has lastManaged = block.timestamp which is dividing and multiplying by 1 days. Dividing by 1 day and then Multiplying by 1 day does not have any effect on the value, as it cancels out and leaves the timestamp unchanged. So, this line of code could be simplified to :
``` lastManaged[msg.sender][_token] = block.timestamp; ```

## [NC-02] Incorrect or Mismatch Comments 

 /**  
-     * @notice Internal function performs call to Layerzero Enpoint Contract for cross-chain messaging.
[This comment](https://github.com/code-423n4/2023-09-maia/blob/c0dc3550e0754571b82d7bfd8f0282ac8fa5e42f/src/interfaces/IBranchBridgeAgent.sol#L130) is supposed to be incorrect because the function itself is external.
``` solidity 
function callOutSystem(address payable gasRefundee, bytes calldata params, GasParams calldata gasParams)
        external
        payable;
```

/**
-     * @notice External function to retry a failed Deposit entry on this branch chain.
This is wrong [dev comment](https://github.com/code-423n4/2023-09-maia/blob/c0dc3550e0754571b82d7bfd8f0282ac8fa5e42f/src/interfaces/IBranchBridgeAgent.sol#L266) as this function is doing what the name of this comment suggests i.e, clearing the tokens to the recipient.
```solidity
function redeemDeposit(uint32 depositNonce) external;
```
## [NC-03] Typo Mistakes 
 /**
-     * @notice Reverts thfe toggle on the given bridge agent  If it's active, it will de-activate it and vice-versa.
[This comment](https://github.com/code-423n4/2023-09-maia/blob/c0dc3550e0754571b82d7bfd8f0282ac8fa5e42f/src/interfaces/IBranchPort.sol#L173) has typo mistake which can be corrected like : 
* @notice Reverts the toggle on the given bridge agent  If it's active, it will de-activate it and vice-versa.

-     *  @param refundee settlement owner adn excess gas receiver.
[This comment](https://github.com/code-423n4/2023-09-maia/blob/c0dc3550e0754571b82d7bfd8f0282ac8fa5e42f/src/MulticallRootRouter.sol#L501) has typo mistake which can be corrected like : 
*  @param refundee settlement owner and excess gas receiver.
[Second Instance](https://github.com/code-423n4/2023-09-maia/blob/c0dc3550e0754571b82d7bfd8f0282ac8fa5e42f/src/MulticallRootRouter.sol#L537)
  