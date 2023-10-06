## [G-1] State Variable Packing for Gas Optimization

### Saves ``10000``

The _unlocked variable utilizes the value 1 to represent the state of the re-entrancy lock modifier. By employing uint96 for _unlocked, we can efficiently store it in a single slot, conserving gas.


```diff
FILE: src/BaseBranchRouter.sol

/// @notice Re-entrancy lock modifier state.
  -  uint256 internal _unlocked = 1;
  +  uint96 internal _unlocked = 1;


```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L42C36-L42C36


```diff
FILE: src/BranchPort.sol

/// @notice Reentrancy lock guard state.
  -  uint256 internal _unlocked = 1;
  + uint96 internal _unlocked = 1;

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L91


```diff
FILE: src/MulticallRootRouter.sol

/// @notice Reentrancy lock guard state.
  -  uint256 internal _unlocked = 1;
  + uint96 internal _unlocked = 1;

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L80

```diff
FILE: src/BranchBridgeAgent.sol

/// @notice Reentrancy lock guard state.
  -  uint256 internal _unlocked = 1;
  + uint96 internal _unlocked = 1;

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L101


```diff
FILE: src/RootBridgeAgent.sol

/// @notice Reentrancy lock guard state.
  -  uint256 internal _unlocked = 1;
  + uint96 internal _unlocked = 1;

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L92


## [G-2] Optimizing Loop Operations with Assembly

Employing inline assembly for loop operations in Solidity can lead to enhanced gas efficiency. Below is a demonstrative contract showcasing the conventional loop structure juxtaposed against its assembly-optimized counterpart.

Here is the example simulated using " for (uint256 i = 0; i < outputParams.outputTokens.length;)".

``` 

contract LoopSimulation {
    struct OutputParams {
        address[] outputTokens;
    }
    
    OutputParams public outputParams;
    uint256 public counter = 0;

    // Adding some dummy tokens for the simulation
    constructor() {
        outputParams.outputTokens.push(0x1111111111111111111111111111111111111111);
        outputParams.outputTokens.push(0x2222222222222222222222222222222222222222);
        // Add more if you want...
    }

    function loopSolidity() public {
        for (uint256 i = 0; i < outputParams.outputTokens.length;) {
            counter++; // dummy operation
            i++;
        }
    }

    function loopAssembly() public {
        assembly {
            let length := sload(outputParams.slot) // get the length of outputTokens array
            let i := 0
            for { } lt(i, length) { } {
                // Increase the counter (assuming counter is the next slot in storage after outputParams)
                let currentCounterValue := sload(add(outputParams.slot, 1))
                sstore(add(outputParams.slot, 1), add(currentCounterValue, 1))
                
                // Increment i
                i := add(i, 1)
            }
        }
    }
}

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L278C13-L278C73

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L281

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L318

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L447


## [G-3] <x> += <y> costs more gas than <x> = <x> + <y> for state variables

For state variables in Solidity, utilizing the compound assignment operator (e.g., +=) incurs a higher gas cost compared to explicitly reassigning the value with a standard operation (e.g., = <x> + <y>). This inefficiency stems from the EVM's underlying operations when handling compound assignments. Developers aiming for gas optimization should be cautious of this behavior and prefer direct assignments when updating state variables.

```diff
FILE: src/BranchPort.sol


         getPortStrategyTokenDebt[msg.sender][_token] += _amount;

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L157

```diff
FILE: src/BranchPort.sol

 getTokenBalance[chainId] += amount;

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/token/ERC20hTokenRoot.sol#L58

```diff
FILE: src/BranchPort.sol

 getStrategyTokenDebt[_token] -= _amount;
 
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L172

```diff
FILE: src/BranchPort.sol

    getPortStrategyTokenDebt[msg.sender][_token] -= _amount;
 
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L169

```diff
FILE: src/token/ERC20hTokenRoot.sol

           getTokenBalance[chainId] -= amount;
 
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/token/ERC20hTokenRoot.sol#L70


## [G-4] Dont catch immutables

The rationale behind this principle is that the Solidity compiler optimizes access to immutable state variables by embedding their values directly into the bytecode at deployment time

```diff
FILE: src/ArbitrumBranchPort.sol

        57.   address _rootPortAddress = rootPortAddress;
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol#L57

```diff
FILE: src/ArbitrumBranchPort.sol

        80.   address _rootPortAddress = rootPortAddress;
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol#L80
















