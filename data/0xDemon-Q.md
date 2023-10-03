# Updating the state after an external call can be a loophole for reentrancy

In several internal functions in the `BranchBridgeAgent` and `RootBridgeAgent` contracts, there is a contract structure that is not in accordance with best practice Check-Effect-Interactions where after making an external call there is a state update, this could be a gap for reentrancy in this function.

*There are four instances of this issue* :

```solidity
File : src/BranchBridgeAgent.sol 

831	  Deposit storage deposit = getDeposit[_depositNonce];
832       deposit.owner = _refundee;
833
834       addressArray[0] = _hToken;
835       deposit.hTokens = addressArray;
836
837       addressArray[0] = _token;
838       deposit.tokens = addressArray;
839
840       uintArray[0] = _amount;
841       deposit.amounts = uintArray;
842
843       uintArray[0] = _deposit;
844       deposit.deposits = uintArray;
845
846       deposit.status = STATUS_SUCCESS;

881	  Deposit storage deposit = getDeposit[_depositNonce];
882       deposit.owner = _refundee;
883       deposit.hTokens = _hTokens;
884       deposit.tokens = _tokens;
885       deposit.amounts = _amounts;
886       deposit.deposits = _deposits;
887       deposit.status = STATUS_SUCCESS;
```

*GitHub : [831 - 846](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L831-L846), [881 - 887](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L881-L887)*

```solidity
File : src/RootBridgeAgent.sol 

1014        settlement.owner = _refundee;
1015        settlement.recipient = _recipient;
1016
1017        addressArray[0] = localAddress;
1018        settlement.hTokens = addressArray;
1019
1020        addressArray[0] = underlyingAddress;
1021        settlement.tokens = addressArray;
1022
1023        uintArray[0] = _amount;
1024        settlement.amounts = uintArray;
1025
1026        uintArray[0] = _deposit;
1027        settlement.deposits = uintArray;
1028
1029        settlement.dstChainId = _dstChainId;
1030        settlement.status = STATUS_SUCCESS;

1106        settlement.owner = _refundee;
1107        settlement.recipient = _recipient;
1108        settlement.hTokens = hTokens;
1109        settlement.tokens = tokens;
1110        settlement.amounts = _amounts;
1111        settlement.deposits = _deposits;
1112        settlement.dstChainId = _dstChainId;
1113        settlement.status = STATUS_SUCCESS;
```

*GitHub : [1014 - 1030](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L1014-L1030), [1106 - 1113](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L1106-L1113)*