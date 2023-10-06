# Analysis of Maia DAO - Ulysses

![Maia DAO - Ulysses](https://assets.coingecko.com/coins/images/22502/large/whiteicon.4a79cf8b.png?1641953707)

## Description

The Ulysses Protocol revolves around creating a decentralized and community-owned platform known as an "Omnichain Liquidity Protocol." This platform addresses the challenges posed by fragmented liquidity in the DeFi landscape.

The key components of Ulysses Protocol include:

Liquidity Providers: Users can deploy their assets on a single blockchain and earn rewards from activities spanning multiple blockchains, promoting efficient capital utilization.

Unified Liquidity Management: Ulysses Protocol simplifies liquidity management across different blockchains, reducing complexity for liquidity providers.

Single Unified Token: Users can create a single token representing their liquidity across various blockchains, enabling versatility in DeFi applications.

**The key contracts of the protocol for this Audit are**:

- **ArbitrumBranchBridgeAgent.sol**: This contract serves as an interface with Users/Routers and acts as a middleman.

- **ArbitrumBranchPort.sol**: It's an implementation of the Ulysses Port specifically designed for deployment on the Arbitrum Branch Chain.

- **ArbitrumCoreBranchRouter.sol**: This contract represents the core Branch Router implementation tailored for the Arbitrum deployment.

- **MulticallRootRouter.sol**: This is the Root Router implementation used to interface with third-party decentralized applications (dApps) within the Root Omnichain Environment.

- **CoreRootRouter.sol**: It's the core implementation of the Root Router, deployed in the Root Environment. It plays a crucial role in the protocol's operation.

- **RootBridgeAgentExecutor.sol**: This contract handles token settlement clearance requests and executes transaction requests originating from the branch chains.

- **CoreBranchRouter.sol**: Similar to ArbitrumCoreBranchRouter.sol, this contract represents the core Branch Router but is intended for deployment in Branch Chains.

- **RootBridgeAgent.sol**: Responsible for interfacing with Users and Routers, this contract acts as a middleman within the Root Environment.

- **VirtualAccount.sol**: This contract allows users to manage assets and perform remote interactions. It is a fundamental component of the protocol.

- **BranchPort.sol**: Used to manage the deposit and withdrawal of underlying assets in response to requests from Branch Bridge Agents. It facilitates interaction with the Branch Chain.

- **MulticallRootRouterLibZip.sol**: Another implementation of the Root Router for interfacing with third-party dApps in the Root Omnichain Environment.

- **BranchBridgeAgent.sol**: This contract is deployed in Branch Chains and is responsible for interfacing with Users and Routers.

- **BaseBranchRouter.sol**: A base Branch Contract that serves as an interface with Branch Bridge Agents, common functionality across Branch Chains.

- **RootPort.sol**: Manages the deposit and withdrawal of assets between the Root Omnichain Environment and every Branch Chain in response to Root Bridge Agents' requests.

- **BranchBridgeAgentExecutor.sol**: Used for requesting token deposit clearance and executing transactions in response to requests from the root environment.

- **ERC20hTokenRoot.sol**: Represents a 1:1 ERC20 representation of a token deposited in a Branch Chain's Port in the Root Chain.

- **ERC20hTokenBranch.sol**: Represents a 1:1 ERC20 representation of a token deposited in a Branch Chain's Port in the Branch Chains.

- **BranchBridgeAgentFactory.sol**: This factory contract allows permissionless deployment of new Branch Bridge Agents, enhancing flexibility.

- **RootBridgeAgentFactory.sol**: A factory contract used for deploying new Root Bridge Agents in the protocol.

- **ERC20hTokenBranchFactory.sol**: A factory contract enabling permissionless deployment of new Branch hTokens in Branch Chains.

- **ERC20hTokenRootFactory.sol**: Similar to the Branch counterpart, this factory contract allows permissionless deployment of new Root hTokens in the Root Chain.

- **ArbitrumBranchBridgeAgentFactory.sol**: Another factory contract, specifically for enabling permissionless deployment of new Arbitrum Branch Bridge Agents.

## Approach Taken-in Evaluating The Maia DAO - Ulysses

Accordingly, I analyzed and audited the subject in the following steps;

1.  **Core Protocol Contract Overview**:

    I focused on thoroughly understanding the codebase and providing recommendations to improve its functionality.
    The main goal was to take a close look at the important contracts and how they work together in the Maia DAO Ulysseso Protocol.

    **Main Contracts I Looked At**

                ArbitrumBranchPort.sol
                ArbitrumCoreBranchRouter.sol
                RootBridgeAgent.sol
                VirtualAccount.sol
                BranchPort.sol
                BranchBridgeAgent.sol
                RootPort.sol
                ERC20hTokenRoot.sol
                ERC20hTokenBranch.sol
                BranchBridgeAgentFactory.sol

    I started my analysis by examining the ArbitrumBranchPort.sol contract. This is the Ulysses Port implementation specifically designed for deployment on the Arbitrum Branch Chain. Ports are essential components of the protocol, making it a critical contract to audit first.

    Then, I turned our attention to the ArbitrumCoreBranchRouter.sol contract represents the core Branch Router implementation tailored for the Arbitrum deployment. Routers play a fundamental role in the protocol's operation, so auditing this contract is crucial.

    RootBridgeAgent.sol: Responsible for interfacing with Users and Routers within the Root Environment, this contract acts as a middleman. It's a central component that interacts with the protocol's users, making it an important contract to consider for auditing.

    VirtualAccount.sol: As a contract that allows users to manage assets and perform remote interactions, it plays a pivotal role in user interactions. Auditing its functionality is important to ensure the security of user assets.

    BranchPort.sol: This contract manages the deposit and withdrawal of underlying assets in response to requests from Branch Bridge Agents. Given its role in handling user assets, it's a key contract for auditing.

    BranchBridgeAgent.sol: Deployed in Branch Chains, this contract is responsible for interfacing with Users and Routers. Since it interacts with users and bridges between chains, auditing its functionality is essential.

    RootPort.sol: Manages the deposit and withdrawal of assets between the Root Omnichain Environment and every Branch Chain in response to Root Bridge Agents' requests. Given its central role in asset movement, auditing is crucial.

    ERC20hTokenRoot.sol: This contract represents a 1:1 ERC20 representation of a token deposited in a Branch Chain's Port in the Root Chain. Auditing token contracts is essential to ensure the security of token handling.

2.  **Documentation Review**:

    In the main time Review [this document](https://v2-docs.maiadao.io/protocols/Ulysses/introduction) and Learn some basic concept in [Readme](https://code4rena.com/contests/2023-09-maia-dao-ulysses#top).

3.  **Manuel Code Review** In this phase, I initially conducted a line-by-line analysis but this was a little harder to me because of a alot of contracts, following that, I engaged in a comparison mode.

    - **Line by Line Analysis**: Pay close attention to the contract's intended functionality and compare it with its actual behavior on a line-by-line basis.

    - **Comparison Mode**: Compare the implementation of each function with established standards or existing implementations, focusing on the function names to identify any deviations.

## Architecture Description and Diagram

Architecture of the key contracts that are part of the Maia DAO - Ulysses:

The Ulysses Protocol is designed to be an "Omnichain Liquidity Protocol" that connects multiple blockchain networks to enhance liquidity and capital efficiency in the DeFi space. Its architecture involves several key components:

- **Root Chain**: The protocol operates on a root blockchain, which serves as the primary chain for managing the protocol's core functionalities.

- **Branch Chains**: These are secondary blockchains connected to the root chain. Branch chains play a crucial role in handling specific tasks related to liquidity, asset management, and interactions with various DeFi platforms.

- **Liquidity Pools**: Within each branch chain, there are liquidity pools where users can deposit their assets. These pools are managed by the protocol and serve as the source of liquidity for various DeFi activities.

- **Bridge Agents**: Bridge agents act as intermediaries between different chains. They facilitate the movement of assets between the root chain and branch chains, allowing users to deposit and withdraw assets seamlessly.

- **Virtual Accounts**: Users interact with the protocol through virtual accounts, which enable them to manage assets and perform actions across different chains remotely.

- **Token Contracts**: The protocol uses ERC-20 token contracts on both root and branch chains to represent assets. These tokens are created when users deposit assets into liquidity pools.

- **Factories**: Factory contracts allow for the permissionless deployment of various protocol components, including bridge agents and tokens.

## Codebase Quality

Overall, I consider the quality of the to be excellent. The code appears to be very mature and well-developed. Details are explained below:

| Codebase Quality Categories | Comments                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| --------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Code-Readability**        | The codebase readability of contracts is quite good. The extensive comments and clear structure make it accessible for developers to understand and work with. However, the actual effectiveness and security of the contract also depend on the implementation of the functions and the correctness of the logic, which would require a more thorough code review.                                                                                                                           |
| **Modularity**              | While some contracts are relatively large, they do serve specific purposes. Consider further decomposing larger contracts into smaller, more modular components when possible, as it can improve maintainability and readability. Additionally, leveraging well-tested external libraries and interfaces contributes to modularity.                                                                                                                                                           |
| **Documentation-Comments**  | The contracts includes extensive inline comments, which is excellent for understanding the code's purpose and functionality. These comments help developers understand each function's role and usage. It follows the NatSpec style for comments.The comments explanations for the contract's purpose, functions, and important variables, making it easier for others to review and work with the code. They also specify details such as function parameters, flags, and expected behavior. |
| **Code-Complexity**         | The contracts has multiple components and handles various functionalities, it appears to be reasonably structured and modular. The use of interfaces and inheritance helps manage complexity and promotes code reuse in contracts. However, the contracts do involves some complexities due to the nature of its responsibilities, cross-contract communication, and error handling. Careful testing and auditing would be essential to ensure the correctness and security of the contracts  |
| **Event-Logging**           | Event logging is a crucial mechanism for emitting notifications and recording significant state changes within the smart contract. Events allow external applications and users to track and respond to these changes on the blockchain but some contracts do not include event logging, Including events to log important contract actions is generally recommended.                                                                                                                         |
| **Error-Handling**          | A lot of contracts includes error handling mechanisms using require statements to revert transactions when certain conditions are not met. it's recommended to use custom errors, using the error keyword. Custom error types are used to throw exceptions in specific error scenarios to help improve the clarity of error messages and provide more information about why a transaction failed.                                                                                             |
| **Testing**                 | This test script outlines the process for building and testing contracts in a Layer Zero fork testing environment. It covers setting up environment variables, installing libraries, and provides instructions for using 'LzForkTest' to change chains, along with static code analysis using Slither                                                                                                                                                                                         |

## Possible Systemic & Centralization Risks

1. The **ArbitrumBranchBridgeAgent** contract facilitates cross-chain communication between different environments (e.g., Arbitrum Branch Chain and Root Omnichain Environment). Systemic risks could occur if there are issues with the interoperability of these environments, leading to unexpected behavior.

2. The **RootBridgeAgent** maintains a mapping of executionState to track the execution status of requests from different chains. If there's a bug in how this state is managed or if an attacker can manipulate it, it could lead to unintended execution or denial of service.

3. The **MulticallRootRouter** contract relies on the initialize function to set the bridgeAgentAddress. Centralization risks may arise if this function is called by the contract owner and the owner has centralized control over contract initialization.

4. **MulticallRootRouter** contract has Non-Reversible Functions, Certain functions in the contract, such as executeResponse and executeDepositSingle, are designed to revert when called. While this might be intentional, it can limit the contract's functionality and could cause issues if not properly documented.

5. The **CoreRootRouter** contract allows adding global tokens (`_addGlobalToken`). If a malicious token is added, it could lead to systemic risk for all users.There's no mechanism to remove global tokens, which can be a risk if a token needs to be delisted.

6. The **ArbitrumCoreBranchRouter** contract relies on a branchBridgeAgentFactory address to create and add new bridge agents. If this address is controlled by a centralized entity, it could control the creation of bridge agents, potentially affecting the system's decentralization.

7. The **ArbitrumCoreBranchRouter** contract uses a fallback function (executeNoSettlement) to execute different functions based on function selectors in the provided data. If not properly validated, this could lead to unexpected behavior and vulnerabilities.

8. In **ArbitrumBranchBridgeAgent** contract the owner address has significant control over contract actions. If the owner address is compromised or controlled by a single entity, it could lead to centralization concerns, as this entity can potentially manipulate contract behavior.

9. **BranchPort** contract has several state variables that can be modified by external parties. These state variables include lists of bridge agents, bridge agent factories, strategy tokens, and port strategies. If these lists grow too large or are modified in unexpected ways, it could affect the contract's behavior and performance.

10. In **ArbitrumBranchPort** contract owner can control the execution of bridging functions (`_bridgeIn` and `_bridgeOut`), which can impact the movement of assets between chains. Centralization concerns may arise if there is limited oversight or transparency in how these functions are executed.

11. This **RootBridgeAgentExecutor** contract relies on `RootBridgeAgent` to execute various operations. If RootBridgeAgent is centralized or controlled by a single entity, it introduces centralization risks.

12. The **ERC20hTokenRootFactory** contract manages port strategies and their associated tokens. maybe some vulnerabilities in managing strategies could lead to risks, including unauthorized manipulation of strategies.

13. The factoryAddress is set to msg.sender in the constructor of **RootBridgeAgent**, which means that only the deployer of the contract can create instances of this contract. This centralization of deployment authority can be a risk if the deployer becomes malicious or compromised.

14. The **VirtualAccount** uses the requiresApprovedCaller modifier to control access. It allows either the owner or approved routers to access the contract. If the approval process for routers is not sufficiently decentralized or transparent, it could centralize control over the contract.

15. The payableCall function allows multiple external calls with different values to be executed in a single transaction in **VirtualAccount** contract. Depending on how this function is used, this is centralize control over funds and introduce risks related to the order of execution.

16. **MulticallRootRouterLibZip** uses data compression through the LibZip library's cdDecompress function. While compression can be useful for saving gas costs, it introduces a potential systemic risk if the decompression process is not well-audited or has vulnerabilities.

17. there is a centrlization risks in RootPort contract if this contract allows the owner to add new chains and set various parameters related to these chains. If the owner has unilateral control over adding chains, it centralizes control over the network's expansion.

18. The **BranchBridgeAgentExecutor** contract handles settlements involving tokens and native assets. Incorrect execution of settlements or vulnerabilities in the settlement process can result in financial losses, which can be systemic risks and aslo the contract supports multiple settlements in a single transaction. This introduces complexity and potential risks related to managing multiple assets and settlements within a single transaction..

**Properly managing these risks and implementing best practices in security and decentralization will contribute to the sustainability and long-term success of the Venus Prime protocol.**

## Gas Optimization

Gas optimization techniques are critical in smart contract development to reduce transaction costs and make contracts more efficient. The Canto protocol, as observed in the provided contract, can benefit from various gas-saving strategies to enhance its performance. Below is a summary of gas optimization techniques followed by categorized findings within the contract:

# Summary

| Finding                                                                               | Occurrences |
| :------------------------------------------------------------------------------------ | :---------: |
| Use mappings instead of arrays                                                        |     42      |
| Using Storage instead of memory for structs/arrays saves gas                          |     14      |
| Use assembly to write address storage values                                          |     40      |
| A modifier used only once and not being inherited should be inlined to save gas       |     12      |
| ERC1155 is a cheaper non-fungible token than ERC721                                   |      3      |
| Use one ERC1155 or ERC6909 token instead of several ERC20 tokens                      |      9      |
| Using assembly to revert with an error message                                        |     177     |
| Split revert statements                                                               |      1      |
| Understand the trade-offs when choosing between internal functions and modifiers      |     27      |
| Using > 0 costs more gas than != 0                                                    |     17      |
| Do-While loops are cheaper than for loops                                             |     15      |
| Use assembly to reuse memory space when making more than one external call            |     44      |
| Missing Zero address checks in the constructor                                        |     14      |
| Short-circuit booleans                                                                |      4      |
| 15 Pre-increment and pre-decrement are cheaper than +1 ,-1                            |      4      |
| Use hardcode address instead address(this)                                            |     33      |
| Use bytes.concat() instead of abi.encodePacked(), since this is preferred since 0.8.4 |     46      |
| Shorten the array rather than copying to a new one                                    |     14      |
| Initializers can be marked as payable to save deployment gas                          |      9      |
| Use assembly to validate msg.sender                                                   |     22      |
| Use selfdestruct in the constructor if the contract is one-time use                   |             |
| Avoid contract calls by making the architecture monolithic                            |      8      |
| It is sometimes cheaper to cache calldata                                             |      7      |
| Use ++i instead of i++ to increment                                                   |      4      |
| Always use Named Returns                                                              |     16      |
| Use bitmaps instead of bools when a significant amount of booleans are used           |     10      |
| Empty blocks should be removed or emit something                                      |      5      |

## Conclusion

The Maia DAO Ulysses Protocol is a comprehensive and ambitious project aiming to create an Omnichain Liquidity Protocol that addresses the challenges posed by fragmented liquidity in the DeFi space. The protocol's architecture involves multiple contracts and components designed to enable users to deploy assets on a single blockchain and earn rewards from activities spanning multiple blockchains. It also simplifies liquidity management across different blockchains and provides users with a single unified token for versatility in DeFi applications.

In this analysis, we've reviewed several key contracts within the protocol, providing insights into their functionality, codebase quality, and potential systemic and centralization risks. Here are some key takeaways:

**Codebase Quality**:

The codebase quality is generally good, with extensive inline comments that provide clear explanations of the contract's purpose, functions, and variables.
Contracts are modular, making use of interfaces and inheritance for code reuse and maintainability.
Error handling is present in contracts, utilizing require statements for validation.
The use of events for logging significant state changes could be improved, and more events could be added for transparency.
Some contracts involve complexity due to cross-contract communication and error handling, highlighting the need for careful testing and auditing.

**Potential Systemic & Centralization Risks**:

Systemic risks can arise from cross-chain communication, interoperability issues, and unexpected behavior.
Centralization risks are present in contracts where the owner has significant control over contract actions or deployment, which can lead to a lack of decentralization.
Contracts that rely on external addresses, such as factories, can introduce centralization risks if not managed properly.
Proper testing, auditing, and the implementation of best practices in security and decentralization are essential to mitigate these risks effectively.

**Gas Optimization**:

Gas optimization techniques are essential for reducing transaction costs and improving contract efficiency.
The contract could benefit from various gas-saving strategies, including the use of mappings instead of arrays, storage for structs/arrays, assembly for address storage values, and more.
Several opportunities exist to optimize gas usage in the contract, such as using ERC1155 or ERC6909 tokens instead of multiple ERC20 tokens and using assembly for certain operations.


### Time spent:
40 hours