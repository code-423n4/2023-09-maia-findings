## Summary

### VirtualAccount.sol

- Manages virtual accounts associated with the Root Port.
- Implements functions for account management and token withdrawals.
- Risk points identified:
  - No events emitted on transfers.
  - No checks for the right token in withdrawERC20 and withdrawERC721.
  - No reentrancy checks.

### RootPort.sol

- Manages the Root Port functionality.
- Tracks local chain ID, Bridge Agents, Bridge Agent Factories, and hTokens.
- Implements various functions for managing tokens, Bridge Agents, and virtual accounts.
- Identified issues:
  - Contract is not upgradable, could be a potential limitation.

![hTokenTable](https://github.com/PFAhard/images-for-analysis/assets/50257230/8eb0ab17-1de6-47e3-92c8-718406448f7d)


### RootBridgeAgent.sol

- Manages interactions between the Root chain and other chains.
- Implements functions for bridging tokens, executing calls, and managing settlements.

### CoreRootRouter.sol

- Facilitates communications between the Root chain and other chains.
- Manages Bridge Agents, bridge agent factories, and strategy tokens.
- Identified risks:
  - If _checkTimeLimit doesn't work correctly, overdrafting daily limit may be possible. // TODO:

### MulticallRootRouter.sol

- Enables executing multiple calls in a single transaction on the Root chain.
- Works in conjunction with the Root Port and RootBridgeAgent.
- Identified issues:
  - Reverts on some execute functions due to missing implementation.

### Root deployment flow:

![Root_deployment_seq](https://github.com/PFAhard/images-for-analysis/assets/50257230/da746aa7-5887-461f-9ef5-e02e26a6f48e)

### ERC20hTokenRootFactory.sol

- Allows the creation of new ERC20hTokenRoot tokens on the Root chain.
- Primarily used by the CoreRootRouter contract.
- No specific risks or issues identified.

### BranchBridgeAgent.sol

- Manages interactions between the Root chain and a specific branch chain.
- Responsible for bridging tokens, executing calls, and managing settlements.

### ArbitrumCoreBranchRouter.sol

- Facilitates communications between the Root chain and the Arbitrum branch chain.
- Manages bridge agents and bridge agent factories for the Arbitrum chain.

### ArbitrumBranchPort.sol

- Manages the Arbitrum branch port functionality.
- Tracks strategy tokens, port strategies, and local chain details.
- Identified risk:
  - replenishReserves can be called by anyone while a potential withdrawal function that may be exploitable.

### ArbitrumBranchBridgeAgentFactory.sol

- A factory contract for creating Arbitrum branch bridge agents.
- No specific risks or issues identified.

## Any comments for the judge to contextualize your findings

During the audit, I identified a High severity vulnerability that enables unauthorized execution of code as a `VirtualAccount`. This vulnerability is attributed to insufficient test coverage for the `VirtualAccount` functionality. I recommend implementing comprehensive test cases to detect and prevent such vulnerabilities in the future.

## Approach taken in evaluating the codebase

To begin the audit, I familiarized myself with the protocol by thoroughly reviewing the available documentation. This allowed me to gain a comprehensive understanding of the key components and functionalities offered by the Ulysses omnichain solution, including Multichain Calls, Tokens Bridging, and Virtual Accounts.

Subsequently, I embarked on a detailed examination of the contracts within the designated audit scope. Through this meticulous review process, I gradually established a clear understanding of the interactions between different chains. To facilitate this analysis, I created visual representations, including graphs depicting the flow of functions responsible for chain interactions.

By following this rigorous approach, I successfully identified vulnerabilities within the project.

![2023-09-maia-global](https://github.com/PFAhard/images-for-analysis/assets/50257230/aefc29ca-7fbe-45a1-bf95-2768b7ce616f)


<details>
<summary>Some of cross chain calls</summary>

# LZ Calls

## ROOT CALLS (SYSTEM CALLS):

1. CoreRootRouter.addBranchToBridgeAgent(Cross-chain):
   - Notes:
     - The callback callOutSystem is performed by the old local Bridge Agent.
   - Impact:
     - Creates a new Bridge Agent associated with the root and links it in the root.
     - To the branch chain:
       - Creates a new Bridge Agent.
         - Sets rootBridgeAgentPath as (root, this(branch)).
         - Deploys a new Bridge Agent Executor.
       - Adds the new Bridge Agent to the Local Port.
         - Sets isBridgeAgent to true.
         - Pushes the new Bridge Agent to bridgeAgents.
       - Increments depositNonce.
     - To the root chain:
       - Links the Branch Agent.
         - Sets getBranchBridgeAgent[branch] to the new Branch Bridge Agent.
         - Sets getBranchBridgeAgentPath[branch] to (branch, this(root)).
   - PATH: CoreRootRouter → RootBridgeAgent → LZEndpoint → BranchBridgeAgent → BranchBridgeAgentExecutor → CoreBranchRouter → BranchBridgeAgent → LZEndpoint → RootBridgeAgent → RootBridgeAgentExecutor → CoreRootRouter → RootPort → RootBridgeAgent.
   
   ![SystemRequest](https://github.com/PFAhard/images-for-analysis/assets/50257230/ef731419-48bb-4ae9-a87a-aabcda254157)

   
2. RootPort.setCoreBranchRouter:
   - Impact:
     - To the root:
       - Increments settlementNonce.
     - To the Branch:
       - Sets coreBranchRouterAddress.
       - Sets isBridgeAgent.
       - Pushes bridgeAgents.

3. CoreRootRouter.toggleBranchBridgeAgentFactory:
   - Impact:
     - To the root:
       - Increments settlementNonce.
     - To the branch:
       - Toggles isBridgeAgentFactory.

4. CoreRootRouter.removeBranchBridgeAgent:
   - Impact:
     - To the Root:
       - Increments settlementNonce.
     - To the branch:
       - Toggles isBridgeAgent.

5. CoreRootRouter.manageStrategyToken:
   - Impact:
     - To the Root:
       - Increments settlementNonce.
     - To the Branch:
       - Sets isStrategyToken.
       - Retrieves getMinimumTokenReserveRatio.
       - Retrieves strategyTokens.

6. CoreRootRouter.managePortStrategy:
   - Impact:
     - To the Root:
       - Increments settlementNonce.
     - To the Branch:
       - Retrieves portStrategies.
       - Retrieves strategyDailyLimitAmount.
       - Sets isPortStrategy.

## BRANCH CALLS:

1. BaseBranchRouter.callOut:
   - Risk:
     - May fail in arbitrum.
   - Impact:
     - To the root:
       - Increments settlementNonce.
     - To the branch:
       - Increments depositNonce.
       - Increments depositNonce a second time.
       
   ![add_global_token](https://github.com/PFAhard/images-for-analysis/assets/50257230/8e94906e-e0ae-47e8-9982-69027a7df9c3)

2. BaseBranchRouter.callOutAndBridge:
   - Currently, it can only bridge tokens to the root. If calldata is presented, it reverts.
   
   ![bridge](https://github.com/PFAhard/images-for-analysis/assets/50257230/46fa590d-57b8-4162-9d52-b150b85bc154)

3. BaseBranchRouter.callOutAndBridgeMultiple:
   - Currently, it can only bridge tokens to the root. If calldata is presented, it reverts.
   
   ![bridge_multi](https://github.com/PFAhard/images-for-analysis/assets/50257230/98dff43a-d5a2-4035-84f2-d6ba08c16ce0)

4. ArbitrumCoreBranchRouter.addLocalToken:
   - Effect:
     - Increments depositNonce in ABBA.

5. CoreBranchRouter.addGlobalToken:
   
   ![2023-09-maia-addGlobalToken](https://github.com/PFAhard/images-for-analysis/assets/50257230/9c545b90-fe26-45c2-8c1b-7931f619c6d5)
   
   - Effect:
     - Increments depositNonce in BBA.
     - Increments settlementNonce in RBA.
     - Increments depositNonce in BBA again.
     - Tokens are created.
     - Retrieves RP.getGlobalTokenFromLocal.
     - Retrieves RP.getLocalTokenFromGlobal.

I hope this clarifies the audit content for you! Let me know if you have any further questions.

</details>


## New insights and learning from this audit

This was my first encounter with the zero layer concept during this audit. It was intriguing for me to delve into the mechanics of token movement across different chains.

## Codebase Quality Assessment

The codebase for Maia Ulysses is highly commendable in terms of quality. It showcases a high degree of maturity and extensive testing, evident from its implementation adhering to widely recognized standards like ERC20 and ERC721. Notably, certain sections of the codebase draw inspiration from Layer Zero protocols. Further details are elaborated upon below:

The codebase is accompanied by comprehensive unit testing, utilizing the Foundry framework. Its inclusion of fuzz tests is particularly impressive, although it is important to note that some parts of the protocol lack test coverage, resulting in potential vulnerabilities.

The codebase generally includes well-written comments, aiding in code comprehension and maintenance.

The overall documentation is exceptional, providing in-depth insights into the ecosystem. However, developers seeking to integrate into the Maia ecosystem would benefit from an explanation of the ecosystem's fundamental contract-level operations, making it more accessible.

The codebase showcases a mature and well-organized structure, promoting clarity and ease of navigation.

In summary, the codebase of Maia Ulysses demonstrates an excellent level of quality across multiple aspects. Its implementation reflects maturity and thorough testing, while the documentation provides comprehensive understanding of the ecosystem. Continuous efforts to update comments and enhance code organization will further elevate the overall codebase quality. Moreover, the use of industry-standard static analysis tools like Slither strengthens the codebase's robustness.

### Time spent:
57 hours