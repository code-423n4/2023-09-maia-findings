# Analysis Report

## Preface

This audit report should be approached with the following points in mind:

 - The report does not include repetitive documentation that the team is already aware of.
 - The report is crafted towards providing the sponsors with value such as unknown edge case scenarios, faulty developer assumptions and unnoticed architecture-level weak spots.
 - If there exists repetitive documentation (mainly in [Mechanism Review](#mechanism-review)), it is to provide the judge with more context on a specific high-level or in-depth scenario for ease of understandability.

## Comments for the judge to contextualize my findings

My findings include 3 High-severity issues, 1 Medium-severity issue and Gas/QA reports. Here are the findings I would like to point out and provide more context on.

#### High-severity issues
**1. No deposit cross-chain calls/communication can still originate from a removed branch bridge agent**
 - This finding explains how no deposit calls can still occur to/from a removed branch bridge agent. This is opposing since branch bridge agents that are toggled off/removed are disabled from further communication. Preventing outgoing no deposit calls are important because user's custom Root Router can communicate with other chains for various purposes such as bridging of tokens from Arbitrum (Root) to another branch chains like Avalanche, Fantom and many more. This is crucial since the call for bridging between Arbitrum and Avalanche originates from a branch bridge agent that is considered inactive. In the recommended mitigation steps, I have provided the exact solutions on how calls should only be allowed from Maia's administration contracts and how calls should be prevented from user's custom Root Router through this check `require(localPortAddress.isBridgeAgent(address(this)), "Unrecognized BridgeAgent!");`. 

**2. Missing requiresApprovedCaller() modifier check on function payableCall() allows attacker to drain all tokens from user's VirtualAccount**
 - This finding explains how an attacker can deplete all ERC20, ERC721 and ERC1155 tokens (except for native ARB) from the user's Virtual Account. Since this is a straightforward issue, I have provided an Attacker's contract to show how the attacker could build a contract to drain all tokens from the Virtual Account. The contract makes use of token contract examples (such as USDC, Arbitrum Odyssey NFT) that I have provided in my POC. Mitigating this issue is crucial since this attack can be executed on multiple Virtual accounts which can affect multiple users.

**3. ERC1155 tokens are permanently locked in user's Virtual Account**
 - This finding explains how ERC1155 tokens are locked in a user's Virtual Account. A user's Virtual Account will be a generalized UI that supports receiving and holding ERC20, ERC721 and ERC1155 tokens. The team has implemented withdraw mechanisms for ERC20 and ERC721 but not ERC1155 tokens, which does not allow normal users (making use of the UI) to withdraw these tokens. I say "normal users" since users who have experience using smart contracts can use the call() or payableCall() functions to directly make an external call to the safeTransferFrom() function. But this cannot be seen as a mitigation or solution since the average normal user will not know how to do so.

**4. lastManaged is not updated when managing a strategy token for the ongoing day**
 - In this issue, I've given an example of how 1999 tokens can be withdraw in a minimum time of 1 minute. Considering the example dailyManagementLimit of 1000 tokens provided in the POC, we can see that 1999 tokens (almost 2000 tokens - i.e. twice the dailyManagementLimit) can be managed/withdrawn from the Port to Strategy in just 1 minute. This issue arises because lastManaged is not updated when managing the token multiple times in a single day. I believe this is logically incorrect because although we are using up the daily limit pending, it is still considered as an **"action of managing the token"**. Additionally, this issue defeats the purpose of the daily management limit which is supposed to control the rate at which tokens are able to be withdrawn.

**5. QA Report**
 - My QA report mainly reflects Low-severity integration issues with LayerZero. One of these issues includes a discussion with the LayerZero Core team member 10xkelly, which shows that if these integration best practices are not adhered to, they can make the contracts (in our case bridge agents) difficult to adapt to those changes and upgrade. Some other issues include use of ERC1155Holder instead of receiver due to OpenZeppelin's latest v5 changes and how double entries in arrays residing in a Port are not prevented.

## Approach taken in evaluating the codebase

### Time spent on this audit: 14 days (Full duration of the contest)

Day 1
 - Understand the architecture and model design briefly 
 - Go through interfaces for documentation about functions

Day 2-7
 - Go through remaining interfaces
 - Clear up model design and functional doubts with the sponsor
 - Review main contracts such as Router, Ports and Bridge Agents starting from Root chain to Branch chain
 - Jot down execution paths for all calls
 - Noted analysis points such as similarity of this codebase with other codebases I've reviewed, mechanical working of contracts, centralization issues and certain edge cases that should be known to the developers.
 - Diagramming mental models on contract architecture and function flow for cross-chain calls
 - Add inline bookmarks while reviewing

Day 7-11
 - Review Arbitrum branch contracts and other independent contracts such as MultiCallRouter and VirtualAccount
 - Explore possible bug-prone pathways to find issues arising out of success/failure of execution path calls

Day 11-14
 - Filter bookmarked issues by confirming their validity
 - Write POCs for valid filtered issues for Gas/QA and HM issues
 - Discussions with sponsor related to issues arising out of model design 
 - Started writing Analysis Report
  

## Architecture recommendations

### Protocol Structure

The protocol has been structured in a very systematic and ordered manner, which makes it easy to understand how the function calls propagate throughout the system. 

With LayerZero at the core of the system, the UA entry/exit points are the Bridge Agents which further interact with LayerZero in the order Bridge Agent => Endpoint => ULN messaging library. Check out [Mechanism Review](#mechanism-review) for a hihg-level mental model.

Although the team has provided sample routers that users can customize to their needs, my only recommendation is to provide more documentation or a guide to the users on how custom Routers can be implemented while ensuring best encoding/decoding practices when using abi.encode and abi.encodePacked. Providing such an outstanding hard-to-break complex protocol is a plus but it can be a double-edged sword if users do not know how to use it while avoiding bugs that may arise.

### What's unique?

1. Separation of State from Communication Layer - Compared to other codebases, the fundamentals of this protocol is solid due to the ease with which one can understand the model. State/Storage in Ports has been separated from the Router and Bridge Agent pairs which serve as the communication layer.
2. Flexible Strategies - Strategies are unique and independent of the state since they can be flexibly added like the Router and Bridge Agent pair. This is unique since new yield earning opportunities can be integrated at any time.
3. Flexible migration - Port state always remains constant, thus making migration easier since only the factories and bridge agents need to changed. This is unique since it allows the Port to serve as a plug-in plug-out system.
4. Encoding/Decoding - The codebase is unique from the perspective of how encoding/decoding has been handled using both abi.encode and abi.encodePacked correctly in all instances. Very few projects strive for such complexity without opening up pathways for bugs. This shows that the team knows what they are building and the intracacies of their system.
5. Permissionless and Partially Immutable - Although the protocol is partially immutable (since Maia Core Root Routers can toggle off any bridge agent), deploying routers and bridge agents are permisionless on both the Root chain and Branch chain. This is unique because not only does the partially immutable nature allow the Maia administration to disable anybody's bridge agent but also does not stop users from deploying another router and bridge agent due the permissionless nature of bridge agent factories. This is a win-win situation for both the Maia team and users.
6. Use of deposit nonces - The use of deposit nonces to store deposit info in the codebase allows the protocol to make calls independent of timing problems across source and destination chains such as Ethereum and Arbitrum. 

### What's using existing patterns and how this codebase compare to others I'm familiar with

1. Maia VS [Altitude DEFI](https://altitude-1.gitbook.io/altitude/) - Similar to Maia, [Altitude DEFI](https://www.altitudedefi.com/transfer) is a cross-chain protocol that uses LayerZero. But they both do have their differences. First, Maia allows users to permisionlessly add ERC20 tokens to the protocol but Altitude DEFI does not. Second, Maia identifies token pairs using chainIds and local, global and underlying tokens stored in Port mappings but Altitude DEFI uses an ID system for token pairs that are identified through their Pool IDs ([More about Altitude Pool Ids here](https://altitude-1.gitbook.io/altitude/ecosystem/pool-ids)). Maia has a much better model overall if we look at the above differences. I would still recommend the sponsors to go through [Altitude's documentation](https://altitude-1.gitbook.io/altitude/) in case any more value can be extracted from a protocol that is already live onchain.

2. Maia VS [Stargate Finance](https://stargate.finance/transfer) - Similar to Maia, [Stargate Finance](https://stargate.finance/transfer) is a cross-chain protocol that uses LayerZero. There is a lot to gain from this project since it is probably the best ([audited 3 times](https://stargateprotocol.gitbook.io/stargate/audits)) live bridging protocol on LayerZero. Let's understand the differences. Similar to Altitude DEFI, Stargate uses Pool IDs to identify token pairs while Maia identifies token pairs using chainIds and local, global and underlying tokens stored in Port mappings. Secondly, Stargate has implemented an [EQ Gas Fee projection tool](https://stargateprotocol.gitbook.io/stargate/developers/eq-fee-projection) that provides fee estimates of different chains on [their frontend](https://stargate.finance/transfer). Maia has implemented the function [getFeeEstimate()](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L161) function but it does not provide Branch-to-Branch estimates. For example, when bridging from Branch Bridge Agent (on AVAX) to another Branch Bridge Agent (on Fantom) through the RootBridgeAgent, the [getFeeEstimate()](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L161) function in BranchBridgeAgent (on AVAX) only allows estimation of fees for the call from Branch chain (AVAX) to Root chain (Arbitrum) and not the ultimate call from AVAX to Fantom. This is because the [dstChainId field is hardcoded with rootChainId](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L167). This is a potential shortcoming of the estimator since it does not calculate the fees for the whole exection path (in our case fees from Arbitrum to Fantom). Maia has the potential to be at or above par with Stargate Finance, thus I would recommend the sponsors to go through [their documentation](https://stargateprotocol.gitbook.io/stargate/) in case any further protocol design weak spots or features could be identified.

There are more protocols that use LayerZero such as Across and Synapse. Each of them have similar implementations and small differences. I would recommend the sponsors to go through their documentations in case any value could further be extracted from them.

### What ideas can be incorporated?

 - Having a fee management system to allow paying gas for cross-chain call in hTokens itself instead of only native fee payment option.
 - Integrating ERC721 and ERC1155 token bridging 
 - Farming, staking and pooling to ensure liquidity exists in ports through strategies. 
 - Providing liquidity in return for voting power or a reward based system. 
 - Transfer Gas estimator on frontend - Check out [Stargate Finance's estimator](https://stargate.finance/transfer) at the bottom of their frontend to get an idea about this.
 - Management contract to filter out poison ERC20 contracts from verified and publicly known token contracts.

## Codebase quality analysis
This is one of the most simplest yet complex high quality cross-chain protocol I have audited. Simple because of the ease with which one can form a mental model of the Root chain and Branch chain high-level interactions. Complex because of the use of the internal routing of calls, encoding/decoding using abi.encode() and abi.encodePacked() and lastly the use of Func IDs to identify functions on destination chains. 

The team has done an amazing job on the complex interactions part. Data sent through each execution path has been correctly encoded and decoded without any mistakes. This is a positive for the system since the team knows the intracacies of their system and what they are doing. 

There are some simple high-level interactions that have not been mitigated such as disabling no deposit calls from a removed bridge agent and missing modifier checks in VirtualAccount.sol. Other than these, all possible attack vectors have been carefully mitigated by the team through access control in Ports, which are the crucial state/storage points of the system. Check out [this section](#areas-of-concern-attack-surfaces-invariants---qna) of the Analysis report to find more questions I asked myself and how they have been mitigated by the team cleverly.

Overall the codebase quality deserves a high quality rank in the list of cross-chain protocols using LayerZero.

## Centralization risks
There are not many centralization points in the system, which is a good sign. The following mentions the only but the most important trust assumption in the codebase:

1. Maia Administration's CoreRootRouter and CoreBridgeAgent contracts
   - These contracts have access to manage Port State such as toggling on/off bridge agents, strategy tokens, port strategies and bridge agent factories
   - From a user perspective, the most important risk is that the administration has the power to toggle on/off their bridge agents. This decreases user's trust in the system.
   - But on the other hand, due to the permissionless nature of the bridge agent factories, users can just deploy another bridge agent with the respective router again. This neutralizes the distrust from the previous point.
   - Therefore, overall the codebase can be termed as **"Permissionless but Partially Immutable"**. Although I've said this before, this is a win-win situation for both the Maia team and users, who can instil a degree of trust into the fact that the system is trustless and in some way permisionless.

## Resources used to gain deeper context on the codebase

 - LayerZero docs and Whitepaper - https://layerzero.gitbook.io/docs/
 - LayerZero integration list - https://layerzero.gitbook.io/docs/evm-guides/layerzero-integration-checklist
 - Maia Ulysses Docs - https://v2-docs.maiadao.io/protocols/Ulysses/introduction
 - Ulysses Omnichain Contract videos - https://maiaeco.notion.site/Code4rena-Code-Overview-3f455e66c457427ab3650a73a614641c
 - This section in the LayerZero docs helped me understand how the team has used version 2 when packing adapterParams in the send() function - https://layerzero.gitbook.io/docs/evm-guides/advanced/relayer-adapter-parameters

## Mechanism Review

### High-level System Overview

![Imgur](https://i.imgur.com/WghH7tl.png)

### Chains supported
**Root:** Arbitrum
**Branch:** Arbitrum, Ethereum Mainnet, Optimism, Base, Polygon, Binance Smart Chain, Avalanche, Fantom, Metis

### Understanding the different token types in the system
Credits to sponsor 0xbuzzlightyear for this explanation:
1. Underlying tokens are the ones that are deposited into BranchPorts on each chain (like WETH, USDC)
2. Global hTokens are minted in the Root chain by the RootPort (arbitrum) that are minted 1:1 with deposited underlying tokens. They are burned before withdrawing a underlying from a BranchPort.
3. Local hTokens are minted in Branch Chains by BranchPort and are minted when a global hToken is bridged from the root to a branch chain. They are burned when bridged from a branch chain to the root chain

In short, you deposit underlying tokens to the platform and get an hToken. If that hToken is in the root it is a global hToken. if that hToken is in a branch chain it is a local hToken,

There is another type of token in the protocol:
Ecosystem tokens are tokens that Maia governance adds to the system and don't have underlying token address in any branch (only the global representation exists). When they are bridged from root to branch, they are stored in rootPort and receipt is provided in branch.

### Documentation/Mental models

#### Here is the basic inheritance structure of the contracts in scope

**Note: Routers not included since they can have different external implementations**

The inheritance structure below is for Bridge Agent contracts:

![Imgur](https://i.imgur.com/xZSvrzY.png)

The inheritance structure below is for Ports contracts:

![Imgur](https://i.imgur.com/S7vtvl2.png)

The inheritance structure below is for the MultiCallRouter contracts:

![Imgur](https://i.imgur.com/WAHosDl.png)

The inheritance structure below is for the factory contracts **(Note: BridgeAgentFactories and ERC20hTokenFactories inherit from their respective BridgeAgent and ERC20hToken contracts though not represented below for separation of contract types and clarity in diagramming):**

![Imgur](https://i.imgur.com/1dXNMRV.png)

The below contracts are Singluar/Helper contracts that do not inherit from any in-scope contracts other than interfaces and the OZ ERC20 token standard:

![Imgur](https://i.imgur.com/1dXNMRV.png)

#### Features of Core Contracts

1. Bridge Agents

![Imgur](https://i.imgur.com/Brjfm7l.png)

2. Ports

![Imgur](https://i.imgur.com/1HI0rIB.png)

3. Routers

![Imgur](https://i.imgur.com/pmaYHBg.png)

#### Simple Data Flow Representation

![Imgur](https://i.imgur.com/Pr23yvA.png)

### Execution Path of each function from each contract

1. CoreRootRouter.sol

![Imgur](https://i.imgur.com/XZmY3zu.png)
![Imgur](https://i.imgur.com/dwnCH4N.png)

2. BaseBranchRouter.sol

![Imgur](https://i.imgur.com/2wT1gt4.png)
![Imgur](https://i.imgur.com/KNiADHw.png)

3. CoreBranchRouter.sol

![Imgur](https://i.imgur.com/Om27PVB.png)
![Imgur](https://i.imgur.com/0FFtNmL.png)

4. BranchBridgeAgent.sol

![Imgur](https://i.imgur.com/JVyKAOo.png)
![Imgur](https://i.imgur.com/dH5VoI5.png)

5. RootBridgeAgent.sol

![Imgur](https://i.imgur.com/eovfF16.png)

## Systemic Risks/Architecture-level weak spots and how they can be mitigated

 - Configuring with incorrect chainId possibility - Currently the LayerZero propritary chainIds are constants (as shown on their website) and are highly unlikley to change. Thus when pasing them as input in the constructor, they can be compared with hardcoded magic values (chainIds) to decrease chances of setting a wrong chainId for a given chain.
 - Ensuring safer migration - In case of migration to a new LayerZero messaging library, it is important to provide the users and members of the Maia community with a heads up announcement in case there are any pending failed deposits/settlements left to retry, retrieve or redeem.
 - Filtering poison ERC20 tokens - Filtering out poison ERC20 tokens from the user frontend is important. Warnings should be provided before cross-chain interactions with a non-verified or unnamed or non-reputed token is being made.
 - Hardcoding _payInZRO to false - Although there are several integration issues (included in QA report), the team should not hardcode the `_payInZRO` field to false since according to the LayerZero team it will make future upgrades or adapting to new changes difficult for the current bridge agents (Mostly prevent the bridge agent from using a future ZRO token as a fee payment option). Check out [this transcript](https://tickettool.xyz/direct?url=https://cdn.discordapp.com/attachments/1155911737397223496/1155911741725757531/transcript-tx-mrpotatomagic.html) or the image below for a small conversation between 10xkelly and I on this subject

![Imgur](https://i.imgur.com/2AFJWLg.png)

## Areas of Concern, Attack surfaces, Invariants - QnA

1. BranchPort's Strategy Token and Port Strategy related functions.
   - There is a possible issue with _checkTimeLimit(), which allows for strategy token withdrawals almost twice the dailyManagementLimit in a minimum time of 1 minute. This issue has been submitted as Medium-severity issue with an example that explains the issue.
2. Omnichain balance accounting
   - Correct accounting in all instances
3. Omnichain execution management aspects, particularly related to transaction nonce retry, as well as the retrieve and redeem patterns:
    a. srChain settlement and deposits should either have status set to STATUS_SUCCESS and STATUS_FAILED depending on their redeemability by the user on the source.
    b. dstChain settlement and deposit execution should have executionState set to STATUS_READY, STATUS_DONE or STATUS_RETRIEVE according to user input fallback and destination execution outcome.
     - Retry/Retrieve and Redeem patterns work as expected and smoothly due to the non blocking nature of the protocol, which prevents issues that could've arised such as permanent message blocking for calls that always fail/revert.

4. Double spending of deposit and settlement nonces / assets (Bridge Agents and Bridge Agent Executors).
   - Not possible since STATUS is updated correctly.
5. Bricking of Bridge Agent and subsequent Omnichain dApps that rely on it.
   - Bridge agents cannot be bricked from incoming calls through LayerZero but might be possible depending on mistakes in the router implementation that dApps on the same chain make. This is a user error due to misconfiguration in implementation and is external to the core working of the current system.
6. Circumventing Bridge Agent's encoding rules to manipulate remote chain state.
   - Encoding/decoding and packing is strictly maintained through the use of BridgeAgentConstants and function Ids, thus there are no mistakes made according to my review.

**Some questions I asked myself:**

Can you call lzReceive directly to mess with the system?
 - No since Endpoint is always supplied as msg.sender to lzReceiveNonBlocking.

Is it possible to block messaging?
 - No, since the protocol uses Non-blocking model.

Can you submit two transactions where the first tx goes through and second goes through but on return the second one comes through before the first one (basically frontrunning on destination chain)?
 - I think this is technically possible since nothing stops frontrunning on destination chain but just posing this question in case sponsors might find some leads

Try calling every function twice to see if some problem can occur
 - Yes, this is where I found the double entry problem. When re-adding or toggling back on a bridge agent (or strategy token, port strategy, bridgeAgentFactory), a second entry is created for the same bridge agent.

Are interactions between Arbitrum branch and root working correctly?
 - Yes, since calls do not go through LayerZero, the interactions are much simplistic

Is it possible for branch bridge agent factories to derail the message from its original execution path?
 - Not possible

Is the encoding/decoding with abi.encode, abi.encodePacked and abi.decode done correctly?
 - Yes, the safe encoding/decoding in the Maia protocol is one of the strongest aspect that makes it bug-proof. 

### Time spent:
150 hours