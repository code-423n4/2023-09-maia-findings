# [L-1] Constructors missing Zero address checks

## Summary
There are multiple instances of constructors missing the address(0) checks which can mistakenly set the contracts to address(0).

## Recommended Mitigation Steps:
Add a address(0) checks in the constructor

---

# [L-2] Do not hardcode zero bytes (bytes(0)) as adapterParameter

## Summary
Do not hardcode zero bytes (bytes(0)) as adapterParamers. Pass them as a parameter instead. They can be used for custom functionality. e.g. receive airdropped native gas from the relayer on destination

## Proof of Concept
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L946
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L793

## Recommended Mitigation Steps:
Instead of hardcoding , pass it as a parameter

---

# [L-3] Array Length Check Discrepancy Between bridgeInMultiple and bridgeOutMultiple

## Summary
In the codebase, there exists a notable discrepancy between the bridgeInMultiple and bridgeOutMultiple functions. While the bridgeOutMultiple function sensibly incorporates a length check to ensure that the input arrays are not greater than 255 elements in length, this critical check is conspicuously absent in the bridgeInMultiple function. The absence of this array length validation in bridgeInMultiple opens up the possibility of unexpected behavior if the input arrays exceed the specified limit. It is essential to maintain consistency in array length checks between these functions to ensure the secure and predictable operation of the contract.

## Proof of Concept
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L299
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L254

## Recommended Mitigation Steps:
Add the same check present in bridgeOutMultiple to  bridgeInMultiple

---

# [L-4] Use OZ reentrancy Guard

## Summary
Use OpenZeppelin reentrancy guard instead of implementing your own.

## Proof of Concept
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BaseBranchRouter.sol#L211-L217
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L921-l927
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L565-L572
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/MulticallRootRouter.sol#L589-L595
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L1189-L1195


