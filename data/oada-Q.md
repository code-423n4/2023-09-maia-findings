## Missing `requiresApprovedCaller` modifier
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/VirtualAccount.sol#L85-L108
The `payableCall()` function lacks the `requiresApprovedCaller` modifier, allowing anyone to call the `payableCall()` function.And it is better to use `external` when the function has no internal calls.

    - function payableCall(PayableCall[] calldata calls) public payable returns (bytes[] memory returnData) {}
    + function payableCall(PayableCall[] calldata calls) external payable requiresApprovedCaller returns (bytes[] memory returnData) {}
