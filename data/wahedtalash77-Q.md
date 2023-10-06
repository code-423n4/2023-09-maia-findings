## \[L-01\] Insufficient coverage

Description
The test coverage rate of the project is ~69%. Testing all functions is best practice in terms of security criteria.

Due to its capacity, test coverage is expected to be 100%.

## \[N-01\] Showing the actual values of numbers in NatSpec comments makes checking and reading code easier

```
97: uint256 internal constant DIVISIONER = 1e4;

uint256 internal constant MIN_RESERVE_RATIO = 3e3;
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L97

## \[N-02 Inconsistent Solidity Versions

```
2	pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchBridgeAgent.sol#L2

```
3: pragma solidity >=0.5.0;
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/ILayerZeroEndpoint.sol#L3

https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/ILayerZeroUserApplicationConfig.sol#L3

https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/ILayerZeroReceiver.sol#L3

## \[N-03\] NatSpec comments should be increased in contracts

https://docs.soliditylang.org/en/v0.8.15/natspec-format.html

## \[N-04\] Function writing that does not comply with the Solidity Style Guide

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

## \[N-05\] Use constants for numbers

==In several locations in the code, numbers like 1e4, 3e3, 1e27 are used...==
https://github.com/code-423n4/2021-05-yield-findings/issues/3#issuecomment-852039791

## \[N-06\] Use SMTChecker

The highest tier of smart contract behavior assurance is formal mathematical verification. All assertions that are made are guaranteed to be true across all inputs → The quality of your asserts is the quality of your verification.

https://twitter.com/0xOwenThurm/status/1614359896350425088?t=dbG9gHFigBX85Rv29lOjIQ&s=19

## \[N-07\] Assembly Codes Specific – Should Have Comments

Since this is a low level language that is more difficult to parse by readers, include extensive documentation, comments on the rationale behind its use, clearly explaining what each assembly instruction does.

This will make it easier for users to trust the code, for reviewers to validate the code, and for developers to build on or update the code.

Note that using Assembly removes several important security features of
Solidity, which can make the code more insecure and more error-prone.

## \[N-08\] Use the delete keyword instead of assigning a value of 0

Using the ‘delete’ keyword instead of assigning a ‘0’ value is a detailed optimization that increases code readability and audit quality, and clearly indicates the intent.

Other hand, if use delete instead 0 value assign , it will be gas saved.

## \[N-9\] Function writing that does not comply with the Solidity Style Guide

Context
All Contracts

Description
Order of Functions; ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

Functions should be grouped according to their visibility and ordered:

constructor
receive function (if exists)
fallback function (if exists)
external
public
internal
private
within a grouping, place the view and pure functions last

\[S-01\] You can explain the operation of critical functions in NatSpec with an infographic.