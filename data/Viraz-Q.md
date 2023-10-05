# Lines of code

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L490


# Vulnerability details

## Impact
When port strategy calls `manage` method in branch port to get funds transferred to it `_checkTimeLimit` is called to calculate the daily limit of the token that the strategy can hold so while calculating `lastManaged` time there is a precision loss `(block.timestamp / 1 days) * 1 days;` as the value is calculated like this

## Tools Used
Foundry
## Recommended Mitigation Steps
`lastManaged` should just be `block.timestamp`