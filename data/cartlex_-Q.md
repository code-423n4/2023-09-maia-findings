# [L-01] Missing zero address checks.

## Lines of code
In `ArbitrumBranchBridgeAgentFactory.sol`

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/factories/ArbitrumBranchBridgeAgentFactory.sol#L32-L35

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/factories/ArbitrumBranchBridgeAgentFactory.sol#L80-L82

In `BranchBridgeAgentFactory.sol`

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/factories/BranchBridgeAgentFactory.sol#L116-L118

In `RootBridgeAgentFactory.sol`

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/factories/RootBridgeAgentFactory.sol#L36

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/factories/RootBridgeAgentFactory.sol#L54

in `ArbitrumBranchBridgeAgent.sol`

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/ArbitrumBranchBridgeAgent.sol#L45-L47

in `ArbitrumBranchPort.sol`

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/ArbitrumBranchPort.sol#L73

in `ArbitrumCoreBranchRouter.sol`

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/ArbitrumCoreBranchRouter.sol#L51

in `BranchBridgeAgentExecutor.sol`

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgentExecutor.sol#L53

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgentExecutor.sol#L66

in `RootPort.sol`

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L259

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L382

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L414

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L421

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L431

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L439

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L444-L445

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L483

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L522
## Tools Used
Manual review

## Recommended Mitigation Steps
Add require statement to check that address is not equal to zero.


# [NC-01] Typos.

## Proof of Concept
In `ERC20hTokenRootFactory.sol`

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/factories/ERC20hTokenRootFactory.sol#L31

```
@param _localChainId Local Network Layer Zerio Identifier. // @audit-issue typos `Zerio` => `Zero`
```

In `ArbitrumCoreBranchRouter.sol`

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/ArbitrumCoreBranchRouter.sol#L19
```
The function `addGlobalToken` is is not available since it's used // @audit-issue `is is` => `is`
```

In `RootPort.sol`

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L54

```
    // @audit-issue typos `acount` => `account`
    mapping(VirtualAccount acount => mapping(address router => bool allowed)) public isRouterApproved; 
```



## Tools Used
Manual review

## Recommended Mitigation Steps
Consider to remove typos.


# [NC-02] Change type of `localChainId` in `ERC20hTokenBranchFactory.sol`.

## Lines of code 
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/factories/ERC20hTokenBranchFactory.sol#L14

In the contructor of `ERC20hTokenBranchFactory.sol` contract `_localChainId` has a type of `uint16`, but on line 14 it has type of `uint24`.

## Tools Used
Manual review

## Recommended Mitigation Steps
Change `uint24` to `uint16` as in constructor.


# [NC-03] No need to cast to string twice.

## Lines of code 
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/token/ERC20hTokenBranch.sol#L20

In the constructor of `ERC20hTokenBranch` contract it casts strings twice.
```
constructor(
        string memory chainName,
        string memory chainSymbol,
        string memory _name,
        string memory _symbol,
        uint8 _decimals,
        address _owner
    ) ERC20(string(string.concat(chainName, _name)), string(string.concat(chainSymbol, _symbol)), _decimals) { // @audit no need to cast to string twice.
        _initializeOwner(_owner); 
    }
```

Same issue in the `ERC20hTokenRoot` contract.
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/token/ERC20hTokenRoot.sol#L38

Moreover, there is no need to concat one input parameter.
```
    // @audit no need to use `concat` operation since input parameter is only `_symbol`
   ) ERC20(string(string.concat(_name)), string(string.concat(_symbol)), _decimals) { 
```

## Tools Used
Manual review

## Recommended Mitigation Steps
Consider to remove double casting and `concat` operator in the constructor of `ERC20hTokenRoot` contract.


