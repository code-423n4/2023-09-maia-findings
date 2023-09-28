## A. Duplicate New Bridge Agents
[Link](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/ArbitrumCoreBranchRouter.sol#L126-L143)
This code creates a new Bridge Agent every time it is called, without checking if a Bridge Agent with the same address already exists. Consequently, it is possible to add duplicate Bridge Agents to the system.
## Impact:
The impact of allowing duplicate Bridge Agents is that it can lead to confusion and unintended behavior within the system. It may also result in increased gas costs due to the unnecessary creation of duplicate Bridge Agents.
## Mitigation:
To mitigate this issue, consider implementing logic in the `_receiveAddBridgeAgent` function that checks whether a Bridge Agent with the same address already exists before creating a new one. 