# Approach

During this audit, each one of the files in the scope and their functionalities have been examined. The flow and the interactions between the different parts of the protocol are complex. Because of that each interaction had to be properly explored, including making different diagrams. Whenever a doubt for a potential vulnerability was present, it was confirmed or denied with a Foundry test.

# Codebase quality

The codebase quality is good. There are, however, some inconsistencies. For example, defined constants are used in one file, while in another one their hardcoded value is used instead.

# Centralization risks

While the protocol has some functionalities available for only trusted parties, it is not that heavily centralized. The protocol's tokens expose their mint and burn functions only for the owner, but that is not an issue, since the owner is a contract of the protocol. Also, there are some mechanisms, like minimumReserveRatio in BranchPort, that prevent trusted parties (a strategy in this case) to gain full access over the given module.

# Mechanism review

**Ulysses** is a multi-chain protocol that enables users to effectively use their assets. The project can be divided in two main parts - **ROOT** and **BRANCH**. **Root** is Arbitrum and **Branch** is any other supported chain. The [**BranchBridgeAgent.sol**](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol) and [**RootBridgeAgent.sol**](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol) enable the communication from branch to root and vice versa. 

There are three types of tokens: 
 - Underlying - an asset deposited in a branch chain
 - Local - a representation of an underlying token (on the branch chain)
 - Global - a representation of an underlying token (on the root chain)

The root serves as a single source of truth, it tracks the balances of various tokens across different chains, maps local tokens to global tokens and etc...

Before depositing, the deposited tokens have to be registered in the system. Bringing assets from a branch chain to a root chain is achieved via deposits. The opposite is done with settlements. Let's look at an example deposit.

 - The **underlying** token A is added as a **local token** on a branch chain. That means that a representation of it is minted on the branch chain (**local token**) and on the root chain (**global token**)
 - User locks an amount of token A into the Branch Port. 
 - A call to the root agent is made using a LayerZero endpoint. 
 - The root agent then mints new global tokens to the user on the root chain.

# Systematic Risks

It goes without saying that an audit is never a guarantee for a bug-free code. So any problems found in the protocol after it goes live, can seriously damage it.

The protocol also interacts with **LayerZero**. That means if there are any high-impact vulnerabilities in the messaging layer on their side, **Ulysses** is susceptible to be exploited. 
 





### Time spent:
40 hours