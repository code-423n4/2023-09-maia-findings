# Ulysses Protocol from Maia DAO

## Description  

The contest focus was on the Ulysses protocol contracts from Maia DAO.

The protocol uses Layer Zero as an inter-blockchain communication layer. This allows virtualization of liquidity which is spread across multiple chains.

## Approach  

The focus for this reviewer was the contract integration with the Layer Zero contracts to allow multichain communication. 

Manual review of the code and the provided protocol documents was done. Approximately 4 hours per day over the course of 8 days were spent reviewing the code.  

## Architectural recommendations  

The protocol consists of permissioned and non-permissioned components. The components of the protocol are spread in a hub-and-spoke pattern, with Arbitrum being the `root` chain and all the other chains being `branch` chains.

Users can permissionlessly create `hTokens` (which are representations of underlying tokens). These can then be moved around various blockchains. 

There is a commendable effort to use as little centralization as possible.

## Qualitative Analysis  

Overall the code reviewed was of a great quality with good test coverage.

| Category | Comments |
| ---- | ---- |
| Test suite | Both unit, integration and fork tests in Foundry were provided with good coverage | 
| Natspec | There were rare, minor natspec errors |


## Centralization Risks  

The protocol is designed in such a way that there is little direct risk to user funds as a consequence of admin privilige abuse. 

It should be noted, however, that the Layer Zero communication protocol presents a singular point of failure. A failure in this regard may be, for example, Layer Zero pausing their contracts due to an exploit. It could also be due to a failure in any of the message channels.

## Systemic Risks  

An interesting risk to consider is that of local vs remote execution costs. As the Ulysses protocol operates across chains it may encounter novel attack patterns which exploit the difference in gas costs between chains (an example of this was made in one of the reviewer's QA submissions). In short: for some chains like zkEVMs with extremely low gas costs, a downward shift in market conditions might make it feasible to grief the protocol by incrementing the `depositNonce` in the `BranchBridgeAgent` contract via the `callOut` method. While economically unviable in the current market conditions, should execution cost fall far enough it may become a viable attack vector.

This means regular monitoring of the current gas and execution environment across relevant chains would be recommended. It is also crucial to enforce the correct gas parameters at the source chain in all functions that pass `gasParams`; at the moment these parameters can be set arbitrarily, allowing an attack path that could block message channels at will.

## Time Spent  

30 hours.  

### Time spent:
30 hours