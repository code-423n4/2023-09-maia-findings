# Virtual Account cannot withdraw ERC1155 directly

In [virtual account](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/VirtualAccount.sol#L56)

the owner of virtual account is capable of withdraw ERC20 token, and ERC721 token and native ETH

but there is no funciton to withdraw ERC1155 token directly

technically speaking the user can trigger the withdaw of ERC1155 token via multicall

but there is two multicall function: 

for the function below

```solidity
  function call(Call[] calldata calls) external override requiresApprovedCaller 
```

user can only trigger this function via the router and cross-chain request, for normal user that just want to withdraw ERC1155, this is too complicated and impact protocol useability

for the function 

```solidity
function payableCall(PayableCall[] calldata calls) public payable 
```

anyone can trigger this function, which lacks acecss control.

to summarize it, the virtual account lacks the function to let user withdraw ERC1155 NFT

# the protocol always pay the layerzero fee in ETH instead of Layerzero token

According to:

https://layerzero.gitbook.io/docs/evm-guides/layerzero-integration-checklist

> Do not hardcode useZro to false when estimating fees and sending messages. Pass it as a parameter instead.

> Do not hardcode address zero (address(0)) as zroPaymentAddress when estimating fees and sending messages. Pass it as a parameter instead.

however, in the current implementation, the protocol always payes the layerzero fee in ETH and offer no option to pay the fee in layerzero token

maybe in the future, paying the fee in layerzero token can have discount or on layerzero side they enforce the fee to be paid by layerzero token, then the usabilty of the maia dao agent contract will be impacted