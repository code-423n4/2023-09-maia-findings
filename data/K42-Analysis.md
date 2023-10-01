### Advanced Analysis Report for [Maia-Dao-Ulysses](https://github.com/code-423n4/2023-09-maia)

#### Overview
- During this audit, I focused on gas optimizations; see my gas report for more details. This analysis encapsulates my understanding of the audit scope, focusing on key contracts. The Ulysses Protocol is engineered to optimize liquidity across multiple blockchains, aiming to enhance capital efficiency and user experience, while increasing opportunity of asset utilization in the decentralized finance sector. This analysis revolves around three of the core contracts for efficiency: [VirtualAccount.sol](https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol), [BranchPort.sol](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol), and [RootPort.sol](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol), each serving distinct functionalities from asset withdrawal to liquidity management and cross-chain interactions.

#### Understanding the Ecosystem:

- **[Virtualized liquidity](https://v2-docs.maiadao.io/protocols/Ulysses/overview/omnichain/virtual-liquidity) and [Virtual Accounts](https://v2-docs.maiadao.io/protocols/Ulysses/overview/omnichain/virtual-accounts):**
  - **Key Contract in Scope**: [`VirtualAccount.sol`](https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol)
  - **Technical Insight**: Handles asset withdrawals through [`withdrawNative()`](https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L51C3-L53C6), [`withdrawERC20()`](https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L56C1-L58C6), and [`withdrawERC721()`](https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L61C1-L63C6) functions.
  - **Security**: Access control relies on [`IRootPort(localPortAddress).isRouterApproved()`](https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L160C2-L167C6), which could be a point of failure if compromised.

- **[Ports](https://v2-docs.maiadao.io/protocols/Ulysses/overview/omnichain/ports):**
  - **Key Contracts in Scope**: [`BranchPort.sol`](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol), [`RootPort.sol`](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol), and [`ArbitrumBranchPort.sol`](https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol)
  - **Technical Insight**: Manages liquidity and asset transfers.
  - **Security**: No explicit reentrancy guards are present, posing a potential risk.

- **[Bridge Agents](https://v2-docs.maiadao.io/protocols/Ulysses/overview/omnichain/bridge-agents):**
  - **Key Contracts in Scope**: [`ArbitrumBranchBridgeAgent.sol`](https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchBridgeAgent.sol), [`RootBridgeAgentExecutor.sol`](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol), [`RootBridgeAgent.sol`](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol), [`BranchBridgeAgent.sol`](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol), [`BranchBridgeAgentExecutor.sol`](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgentExecutor.sol), [`RootBridgeAgentFactory.sol`](https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/RootBridgeAgentFactory.sol), and [`ArbitrumBranchBridgeAgentFactory.sol`](https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ArbitrumBranchBridgeAgentFactory.sol)
  - **Technical Insight**: Facilitates cross-chain communication.
  - **Security**: Requires secure message-passing protocols to prevent MITM attacks.

#### Codebase Quality Analysis:

- **1. [Structures and Libraries](https://github.com/code-423n4/2023-09-maia)**:
  - Uses custom modifiers like `requiresApprovedCaller` and `lock` for access control and reentrancy protection.

- **2. [Security Measures](https://github.com/code-423n4/2023-09-maia)**:
  - Access control is primarily done through custom modifiers.
  - Reentrancy protection is attempted but not guaranteed.

- **3. [Error Handling](https://github.com/code-423n4/2023-09-maia)**:
  - Uses `revert` statements with custom messages for debugging.

#### Architecture Recommendations:
- Consider using OpenZeppelin's `AccessControl` instead of current usage, for a more standardized approach to access control.
- Implement more specific error codes for easier debugging.

#### Centralization Risks:
- Dependency on [IRootPort](https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/IRootPort.sol). for access control in [VirtualAccount](https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol) could lead to unauthorized access if compromised.

#### Mechanism Review:
- Access control is primarily done through custom modifiers.
- Asset management is handled through various `withdraw` and `manage` functions.

#### Systemic Risks:
- Contract failures due to bugs in the `manage` function could lead to liquidity issues.
- Access control failures if [IRootPort](https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/IRootPort.sol). is compromised.

#### Areas of Concern:
- The contracts do not appear to be upgradeable.
- The `lock` modifier's effectiveness is not clear without full context.

#### Codebase Analysis:
- The contracts are modular but could benefit from using standard libraries for access control and storage.

**[VirtualAccount Contract](https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol)**:
- **Functions**: [withdrawNative()](https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L51C3-L53C6), [withdrawERC20()](https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L56C1-L58C6), and [withdrawERC721()](https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L61C1-L63C6)
 - **Purpose**: These functions handle asset withdrawals.

**Security Concerns**: 
- **Access Control**: 
  - **Modifier**: [requiresApprovedCaller](https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L160C3-L167C6)
  - **Dependency**: Relies on `IRootPort(localPortAddress).isRouterApproved()`
  - **Risk**: If the [IRootPort](https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/IRootPort.sol) contract is compromised, unauthorized access could occur.
  - **Context**: The [requiresApprovedCaller](https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L160C3-L167C6) modifier delegates access control to an external contract ( [IRootPort](https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/IRootPort.sol)). If that contract is compromised, it could grant approval to malicious addresses.
  - **Mitigation**: Implement a decentralized access control mechanism that doesn't rely solely on [IRootPort](https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/IRootPort.sol).

**[BranchPort Contract](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol)**:

**Technical Insights**:
- **Functions**: [manage](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L144C1-L164C6)
  - **Purpose**: Manages liquidity within the contract, allowing users to add or remove assets.

**Security Concerns**:
- **Reentrancy Protection**: 
  - **Modifier**: [lock](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L566C1-L571C6)
  - **Risk**: Potential for reentrancy attacks if `_unlocked` is not managed securely.
  - **Mitigation**: Use a well-vetted reentrancy guard such as OpenZeppelin's `ReentrancyGuard`.

**[RootPort Contract](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol)**:

**Security Analysis**:
**Functions and Risks**:
- **[initialize](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L129C1-L139C6)**: 
  - **Risk**: No validation for `_setup` variable, allowing for potential overwrites.
  - **Context**: Without validation, multiple calls to `initialize` could overwrite important state variables.
  - **Mitigation**: Implement a state variable to track initialization status.

- **[initializeCore](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L147C1-L163C6)**: 
  - **Risk**: No validation for `_setupCore` variable.
  - **Context**: Similar to `initialize`, multiple calls could overwrite core settings.
  - **Mitigation**: Implement a state variable to track core initialization status.

- **[renounceOwnership](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L165C1-L169C1)**: 
  - **Risk**: None. Function is overridden to revert.
  - **Context**: This is a good practice to prevent accidental renouncement of ownership.

- **[setAddresses](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L239C1-L256C6)**: 
  - **Risk**: Can overwrite existing mappings.
  - **Context**: Overwriting mappings without checks could lead to unintended consequences.
  - **Mitigation**: Implement checks to prevent unintended overwrites.

- **[addBridgeAgent](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L382C2-L391C1)**: 
  - **Risk**: Can add an arbitrary `_bridgeAgent`.
  - **Context**: Without a whitelist, any address could be added as a bridge agent, potentially compromising the system.
  - **Mitigation**: Implement a whitelist of approved agents.

- **[toggleBridgeAgent](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L414C2-L418C6)**: 
  - **Risk**: Can toggle the state of any `_bridgeAgent`.
  - **Context**: Toggling state without additional checks could enable or disable agents unintentionally.
  - **Mitigation**: Require multi-signature approval for this action.

- **[addNewChain](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L438C5-L481C1)**: 
  - **Risk**: Can add a new chain with arbitrary parameters.
  - **Context**: Adding a new chain without checks could introduce insecure or incompatible chains.
  - **Mitigation**: Require multi-signature approval for adding new chains.

- **[setCoreRootRouter](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L509C1-L518C6)**: 
  - **Risk**: Can change `coreRootRouterAddress` and `coreRootBridgeAgentAddress`.
  - **Context**: Changing these addresses without checks could redirect transactions to malicious contracts.
  - **Mitigation**: Implement a timelock for sensitive changes.

#### Recommendations:
- **Formal Verification**: Use tools like [Certora](https://www.certora.com/), and extend the test coverage from 69% to 100%. 
- **Monitoring Tools**: Use [Tenderly](https://tenderly.co/) and [Defender](https://openzeppelin.com/defender/) for real-time monitoring.

#### Conclusion:
- The Ulysses Protocol is a complex ecosystem with multiple functionalities. The use of custom modifiers for access control and reentrancy protection is noted, but these need to be more rigorously tested. Given the dependency on `IRootPort` for access control in [VirtualAccount.sol](https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol), a more standardized and decentralized solution for access control is recommended. Specific recommendations tied to individual functions have been provided for mitigating identified risks. More rigorous testing, 100% coverage using formal verification and invariant testing, and continued future third-party audits are crucial for its long-term success and security.

### Time spent:
36 hours