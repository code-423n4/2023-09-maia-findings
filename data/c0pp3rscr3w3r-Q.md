Use error AmountsAreZero() instead of InvalidInputParams() in RootBridgeAgent when amount is zero instead of invalidParams.
As the former is not used but is more applicable than the latter.

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L1141