## Title
Unused Return Value Warning in ArbitrumBranchBridgeAgent.sol

## Impact
The warning indicates that the return value of a low-level call to `_rootBridgeAgentAddress` is not being used in the contract. While this warning is not an error, it suggests that the contract might not be handling potential errors or responses correctly when interacting with `_rootBridgeAgentAddress`. Ignoring return values from low-level calls can lead to unexpected behavior or vulnerabilities.

## Proof of Concept
The warning is located in the following line of code in ArbitrumBranchBridgeAgent.sol:
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchBridgeAgent.sol#L103
```solidity
_rootBridgeAgentAddress.call{value: msg.value}("");
```
This code makes a low-level call to `_rootBridgeAgentAddress`, but the return value is not used or checked for errors.

## Tools Used
code review

## Recommended Mitigation Steps with Code
To address this warning and ensure proper error handling, you can modify the code as follows:

```solidity
(bool success, ) = _rootBridgeAgentAddress.call{value: msg.value}("");
require(success, "Low-level call to _rootBridgeAgentAddress failed");
```

By checking the `success` variable and requiring that it's `true`, you ensure that the low-level call to `_rootBridgeAgentAddress` was successful. This helps prevent unexpected issues and improves the security of your contract.