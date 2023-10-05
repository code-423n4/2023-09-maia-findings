## Audit Details

**Project Name** : Miadao

**Auditor**: Franklin

**Languages:** Solidity

**Date:** October 5th 2023

**Audit Goals:** The focus of the audit was to verify that the smart contract system is secure, resilient and working according to its specifications. The audit activities can be grouped in the following

**Security:** Identifying security related issues within each contract and within the system of the contracts. 
**Sound Architecture:**  Evaluation of the architecture of this system through the lens of established smart contract best practices and general software best practices
**Code correctness and Quality:**  A full review of the contract source code. The primary area of focus on correctness and readability 


**Overview of Maia**
Maia DAO is a decentralized finance (DeFi) yield powerhouse, dedicated to providing a comprehensive ecosystem for various native DeFi financial instruments. It aims to be a one stop shop for different DeFi native financial instruments.
The protocol integrates with Layer zero which is a cross chain communication protocol that enables multiple blockchains to communicate and share information seamlessly without compromising security or decentralization. It also integrates with Ulysses Protocol which is a permissionless omichain liquidity protocol designed to promote capital efficiency, enabling Liquidity Providers to deploy their assets from one chain and earn revenue from activity in an array of chains all while mitigating the negative effects of current market solutions.

## Architecture 

**Branch Bridge Agent** – This contract mediate interactions with a Branch Router, Deploys Branch Chains of Omnichain System and responsible for interfacing with Users and Routers. They also track user deposits and communicate with the local branch port for asset deposit and withdrawals. 

**Root Bridge Agent** – The Root Bridge Agents are located in the root chain. It interfaces with all connected Branch chains and their respective Branch Bridge Agents. Each Root agent connects to a root router, enabling seamless integration with other dapps in the root chain.

**Root Bridge Agent Executor** - Requests token settlement clearance and executes transaction requests from the branch chains

**Branch Port** – For each chain there is one Branch port which serves all the branch routers. The branch port manages the deposit and withdrawal of underlying assets from the Branch Chain in response to Branch Bridge Agents' requests

**Root Port** – The Root Port is also located in the Root Chain and communicates with all connected Root Routers. It Manages the deposit and withdrawal of assets between the Root Omnichain Environment and every Branch Chain in response to Root Bridge Agents requests. Any action that involves the addition, removal or verification of tokens and their balances has to go through this contract. 

**Routers** – This is another key component of the protocol. It which ensures composable and customizable cross chain interactions. There are two core routers in the protocol. 

**•CoreBranchRouter** – This contract serves as user-facing entry and exist points within the omnichain system, upon receiving a user request, they perform necessary state changes to the origin Branch Chain's contract and communicate with the connected Branch Bridge Agent for transmitting and receiving cross chain messages 

**•CoreRootRouter** – The Root Routers connect to a Root Bridge Agent for seamless request reception and response transmission.  

**Virtual Accounts** - The Virtual Account is a smart contract designed to provide encapsulated user balances for decentralized applications (dApps) operating in the Arbitrum root chain environment. It allows users to manage assets and perform actions like swapping tokens, adding liquidity, farming, and more from different blockchain networks. 


**Documentation**
The following documentation(s) was available to wardens and provides the necessary context for this report
https://v2-docs.maiadao.io/ 


**Scope Details**
The following smart contracts were in scope: 

ArbitrumBranchBridgeAgent.sol
ArbitrumBranchPort.sol
ArbitrumCoreBranchRouter.sol
MulticallRootRouter.sol
CoreRootRouter.sol
RootBridgeAgentExecutor.sol
CoreBranchRouter.sol
RootBridgeAgent.sol
VirtualAccount.sol
BranchPort.sol
MulticallRootRouterLibZip.sol
BranchBridgeAgent.sol
BaseBranchRouter.sol
RootPort.sol
BranchBridgeAgentExecutor.sol
ArbitrumBranchBridgeAgentFactory.sol
BranchBridgeAgentFactory.sol
ERC20hTokenBranchFactory.sol
ERC20hTokenRootFactory.sol
RootBridgeAgentFactory.sol
ERC20hTokenBranch.sol
ERC20hTokenRoot.sol
BridgeAgentStructs.sol
IRootBridgeAgent.sol
BridgeAgentConstants.sol
IERC20hTokenBranchFactory.sol
IPortStrategy.sol
ICoreBranchRouter.sol
IERC20hTokenBranch.sol
IBranchBridgeAgent.sol
IBranchBridgeAgentFactory.sol
IRootPort.sol
ILayerZeroEndpoint.sol
IERC20hTokenRoot.sol
IBranchPort.sol
IVirtualAccount.sol
IArbitrumBranchPort.sol
ILayerZeroUserApplicationConfig.sol
IMulticall2.sol
IBranchRouter.sol
IRootBridgeAgentFactory.sol
IERC20hTokenRootFactory.sol
IRootRouter.sol
ILayerZeroReceiver.sol 



**Issue Overview**
The following are some of the issues discovered during the audit. 


**[L-01] Owner can Initialize the state of the contract as many times as possible.** 
The initialize function is a regular function with intention that it is called only once to initialize the state of the contract. The problem is that there is no modifier or restrictions to ensure that this function is only callable once. Therefore, owner can decide to call this function and reinitialize the state of the contract as many times as possible. 

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L129 

**Recommendation**
If the intention of the initialize function is to be called only once, use a modifier to ensure that. 

```
contract RootPort {
    bool private Initialized = false;

    // Modifier to check if the function has not been called before
    modifier onlyOnce() {
        require(!Initialized, "Function can only be called once");
        _;
    }

   function initialize(address _bridgeAgentFactory, address _coreRootRouter)    external onlyOwner onlyOnce {        
         // Initialize state
         ...

        //Set the flag to indicate that the function has been called
        Initialized = true;
    }
}
```

**[L-02] In BridgeToRoot(),  amount can be less than deposit.**
With the current implementation BridgeAgent can mistakenly process inputs where amount is less than deposit. Though the if block will catch this when _amount is subtracted from deposit, which if above is the case it will result to a negative value and fail, nevertheless its best practice to add a require statement that validates these inputs. Also adding the require statement will help caller to know why the transaction failed when the error message is being thrown as to just as to just simply reverting. 

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L277

**Recommendation**
 Add a require statement that validates these inputs 
```require (_amount >= _deposit, Deposit cannot be greater than amount)```  
Also add these check where ever that deals with amount and deposits in the contract.

**[L-03] CoreBranchRouterAddress is being set/Overwritten with multiple functions**
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L129
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L334 
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L419 
This can lead to confusion and unexpected behaviors in the contract.
 
**Recommendation**
Use only one function for setting this address after initialization with access restriction. Remove unnecessary statements in functions that re-sets this address. Also, `setCoreRouter` checks if already set coreBranchRouterAddress is a zero address, this makes no sense and unnecessary because it has already been checked in the initialize function and also prevents it to be changed if that is the case.  


**[L-04] The array length in _approveMultipleAndCallOut in MulticallRootRouter.sol is not validated**
If the length of input arrays are not the same, It can result to one of the arrays being shorter than the other. For example if outputTokens has a length of 5 and amountsOut has a length of 4, iterating through them without proper validation can lead to an “index out of bound” error when accessing the fifth element of amountsOut.

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/MulticallRootRouter.sol#L544

**Recommendation**
Add checks to ensure that all input arrays have the same length before proceeding with the loop. 
```
require(
    outputTokens.length == amountsOut.length &&
    outputTokens.length == depositsOut.length,
    "Array lengths do not match"
);
```

Also validate array length in _transferAndApproveMultipleTokens() in  BaseBranchRouter and any other place necessary 


**[L-05] For re-entrancy check, Prefer using battle tested code and library like Openzeppelin re-entrancy check.**
The replenish Reserves uses a lock modifier to safeguard against reentrancy, Though this works fine without any issue, its best practice to use battle tested code and libraries to ensure premium security against re-entrancy.

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L167
 

**Recommendation**
Recommend using Open zeppelin reentrancy modifier instead of the lock modifier for reentrancy guard. 


**[L-06] StrategyTokens array might grow too big to loop over without running out of gas**
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L55 

**Recommendation**
Recommend setting a fixed array for strategy tokens maybe 500. Or use maxTokenLength 

**[L-07] In `addPortStatregy` method ensure the portStrategy and token being added is not a zero address**
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L382

**Recommendation**
Add a require statement to ensure that portStrategy and token are not a zero address 

**[N-01] In `coreRootRouter.sol` when checking if chainId is already added, For already added chainId, change error message to `chainId exists` instead of Invalid chainId**
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/CoreRootRouter.sol#L120 

**[N-02] Avoid use of Magic numbers**
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L299 
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L1210 
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L942 

**Recommendation**
Use a constant variable to represent this number so users and developers interacting with the contract can better understand the intent of the number. 

**[N-03] Incorrect Natspec definition for CoreRootBridgeAgent.sol**
The natspec definition for coreRootBridgeAgent states that CoreRootBridgeAgent is “The address of the core router in charge of adding new tokens to the system”. Same Definition was used for coreRootRouterAddress. This can be misleading and lead to confusion. 

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L43 
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L40C20-L40C41

**Recommendation**
Update Natspec to reflect the correct definition of CoreRootBridgeAgent

**[N-04] Incorrect Natspec definition for isChainId mapping** 
@notice says mapping from address to Bridge Agent. While the correct definition should be mapping from chainId to isActive (Boolean). Therefore, checking if ChainId is active or not. 

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L61C54-L61C63 

**Recommendation**
Update Natspec to reflect the correct definition of isChainId mapping 

**[N-05] Definition of  `minimumReservesRatio` and `_dailyManagementLimit` not documented in addStrategyToken and addPortStrategy functions in IBranchPort.sol**

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/interfaces/IBranchPort.sol#L181 
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/interfaces/IBranchPort.sol#L193C77-L193C98 

Not documenting the intent for these parameters can lead to confusion and lack of comprehension for its purpose for developers and users interacting with the contact. 

**Recommendation**
Update Natspec to document the intent of these parameters in their respective functions. 

## Codebase Quality Analysis
The quality of the codebase appears to be of high quality and well-structured. A key aspect is the separation of concerns with different agents handling different aspects of the system, like token settlement, clearance, execution, deposit and withdrawal management. The code is documented but can be improved on like documentation of some parameter in core functionalities like add strategy token etc, which would be beneficial for future developers.

**Systemic risks:**
The contracts relies heavily on LayzerZero for its communication between chains, Bugs in the contract can heavily impart the contracts as well.  

The protocol uses Lock modifier to guard against re-entrancy, its recommended to use battle tested code and libraries like open-zeppelin reentrancyguard.sol to guard against re-entrancy instead of the current Lock modifier.

In a situation where Bridge agent is compromised or malicious, it could disrupt Maia operations or perform actions that are not in the best interest or users.

**Centralization risks:**
Some key functionalities in the system like managePortstrategy relies on onlyOwner. Which raises concern for centralization risk. If owner Is compromised, they can manipulate the behavior of some key functionalities to their best interest. Centralization undermines the trust and transparency that decentralized systems aim to provide. it is preferable to have governance mechanisms for key functionalities instead of relying solely on a single entity with elevated privileges. 

**Conclusion**
In conclusion, the Maia project exhibits a strong and well-thought-out architecture and the team has done a great job regarding the code. However, addressing the identified risks will further ensure the project's security and usability. Additionally, it is recommended to improve the documentation and comments in the code to enhance understanding and collaboration among developers. it is also highly recommended that the team continues to invest in security measures such as mitigation reviews, audits, and bug bounty programs to maintain the security and reliability of the project.

**Tools used**
All the issues stated above were found during Manual Review

**Time spent on analysis**
70 hrs





