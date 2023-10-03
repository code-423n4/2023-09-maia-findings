# 🛠️ Analysis - Maia DAO Ulysses Project 
### Summary
| List |Head |Details|
|:--|:----------------|:------|
|a) |The approach I followed when reviewing the code | Stages in my code review and analysis |
|b) |Analysis of the code base | What is unique? How are the existing patterns used? "Solidity-metrics" was used  |
|c) |Test analysis | Test scope of the project and quality of tests |
|d) |Architectural | Architecture feedback |
|e) |Documents  | What is the scope and quality of documentation for Users and Administrators? |
|f) |Centralization risks | How was the risk of centralization handled in the project, what could be alternatives? |
|g) |Systemic risks | Potential systemic risks in the project |
|h) |Security Approach of the Project | Audit approach of the Project |
|i) |Other Audit Reports and Automated Findings | What are the previous Audit reports and their analysis |
|j) |Gas Optimization | Gas usage approach of the project and alternative solutions to it |
|k) |Packages and Dependencies Analysis | Details about the project Packages |
|l) |Other recommendations | What is unique? How are the existing patterns used? |
|m) |New insights and learning from this audit | Things learned from the project |
|n) |Summary of Vulnerabilities | Audit Results |
|0) |Time spent on analysis | Time detail of individual stages in my code review and analysis |


## a) The approach I followed when reviewing the code

First, by examining the scope of the code, I determined my code review and analysis strategy.
https://github.com/code-423n4/2023-09-maia

Accordingly, I analyzed and audited the subject in the following steps;

| Number |Stage |Details|Information|
|:--|:----------------|:------|:------|
|1|Compile and Run Test|[Installation](https://github.com/code-423n4/2023-09-maia#tests)|Test and installation structure is simple, cleanly designed|
|2|Architecture Review|The [Ulysses](https://v2-docs.maiadao.io/protocols/Ulysses/introduction) explainer provides a high-level overview of the Project system and the [docs](https://v2-docs.maiadao.io/) describe the core components.|Provides a basic architectural teaching for General Architecture|
|3|Graphical Analysis  |Graphical Analysis with [Solidity-metrics](https://github.com/ConsenSys/solidity-metrics)|A visual view has been made to dominate the general structure of the codes of the project.|
|4|Slither Analysis  | [Slither Report](https://github.com/code-423n4/2023-09-maia/blob/main/slither.txt)|The Slither output did not indicate a direct security risk to us, but at some points it did give some insight into the project's careful scrutiny. These ; 1) Re-entrancy risks |
|5|Test Suits|[Tests](https://github.com/code-423n4/2023-09-maia#tests)|In this section, the scope and content of the tests of the project are analyzed.|
|6|Manuel Code Review|[Scope](https://github.com/code-423n4/2023-09-maia#scope)|Ulysses scope for this audit focuses on Ulysses Omnichain Liquidity and Execution Platform built on top of Layer Zero.|
|7|Project Other Audit |[February Zellic Audit Report - May Zellic Audit Report](https://github.com/code-423n4/2023-05-maia/blob/main/audits/Ulysses%20Protocol%20May%202023%20-%20Zellic%20Audit%20Report.pdf)|This reports gives us a clue about the topics that should be considered in the current audit|
|8|Infographic|[Figma](https://www.figma.com/)|I made Visual drawings to understand the hard-to-understand mechanisms|
|9|Special focus on Areas of  Concern|[Areas of Concern](https://github.com/code-423n4/2023-09-maia#areas-of-concern)|There are specific concerns that special attention to. These are:  BranchPort's Strategy Token and Port Strategy related functions ,  Omnichain balance accounting , Omnichain execution management aspects|

## b) Analysis of the code base

The most important summary in analyzing the code base is the stacking of codes to be analyzed.
In this way, many predictions can be made, including the difficulty levels of the contracts, which one is more important for the auditor, the features they contain that are important for security (payable functions, uses assembly, etc.), the audit cost of the project, and the time to be allocated to the audit;
Uses Consensys Solidity Metrics
-  **Lines:** total lines of the source unit
-  **nLines:** normalized lines of the source unit (e.g. normalizes functions spanning multiple lines)
-  **nSLOC:** normalized source lines of code (only source-code lines; no comments, no blank lines)
-  **Comment Lines:** lines containing single or block comments
-  **Complexity Score:** a custom complexity score derived from code statements that are known to introduce code complexity (branches, loops, calls, external interfaces, ...)

![image](https://github.com/code-423n4/2023-09-maia/assets/104318932/f320e258-fdc2-4e42-8e1d-8a4a04286196)

</br>
</br>


### Entry points

The diagram of the entry points where a user interacts with the protocol is the most practical and understandable way to first understand the protocol. In this diagram, the entry points of the project can be seen;

![image](https://github.com/code-423n4/2023-09-maia/assets/104318932/668632c7-decc-4b86-8bd3-5668d8bd0dc5)

</br>
</br>


### RootBridgeAgent.sol

![image](https://github.com/code-423n4/2023-09-maia/assets/104318932/5c9ca987-169e-426a-ac82-86ee1fdb3b27)

The Root Bridge Agent is responsible for sending/receiving requests to/from the LayerZero Messaging Layer for execution, as well as requests tokens clearances and tx execution from the `RootBridgeAgentExecutor`.

Bridge Agents allow for the encapsulation of business logic as well as the standardized cross-chain communication, allowing for the creation of custom Routers to perform actions as a response to remote user requests. This contract is for deployment in the Root Chain Omnichain Environment based on Arbitrum.

In order to understand the project, it is very important to generally understand the logic of Bridge Agents - Bridge Agents Executor and Router, which is the main working system; 


![image](https://github.com/code-423n4/2023-09-maia/assets/104318932/d810f854-e93b-4e68-bbf2-3be97f8134cd)


## c) Test analysis
### What did the project do differently? ;
-   1) The tests of the project were designed according to the threat model and it was a part where the team did a great job.


### What could they have done better?
-   1) It is important to have 100% test coverage, the slightest gap in a project's test coverage prevents unexpected major issues from being uncovered.

```js
README.md:
  - What is the overall line coverage percentage provided by your tests?: 69%
```
</br>

-   2) The project utilizes Foundry as its development pipeline tool, containing an array of testsand scripts coded in Solidity.
The Foundry tool automatically selects Solidity version 0.8.19 based on the version specified within the foundry.toml file.
The project contains discrepancies with regards to the Solidity version used as the pragma statements of the contracts are open-ended ( ^0.8.0 ).
We advise them to be locked to 0.8.19 ( =0.8.19 ), the same version utilized for our static analysis as well as optimizational review of the codebase.

</br>
</br>

- 3) Test suites do not test for re-entrancy
Test teams are testing many functions and variables, but recently, due to the vulnerability in the Vyper Compiler, the hacking of the projects using certain Vyper compiler and losing 50 million $ has revealed the security weakness here. https://cointelegraph.com/news/curve-finance-pools-exploited-over-24-reentrancy-vulnerability

```
lock modifier functions : 87 results - 12 files

```
The accuracy of the functions has been tested, but it has not been tested whether the `lock` modifier in the function works correctly or not. For this, a test must be written that tries to reentrancy and was observed to fail.

Let's take the depositToPor function from the project as an example, we can write any test of this function, it has already been written by the project teams in the test files, but the risk of reentrancy with the lock modifier in this function has not been tested, a test file can be added as follows;

Project File:
```solidity
src\ArbitrumBranchBridgeAgent.sol:
  67       */
  68       */
  69:     function depositToPort(address underlyingAddress, uint256 amount) external payable lock {
  70:         IArbPort(localPortAddress).depositToPort(msg.sender, msg.sender, underlyingAddress, amount);
  71:     }
```
</br>

Reentrancy Test  File:
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import {DSTest} from "forge-std//test.sol"
import {console2} from "forge-std/console2.sol";

import "./ArbitrumBranchBridgeAgent.sol"; // Replace with the path to your contract

contract ReentrancyAttack {
    ArbitrumBranchBridgeAgent victim;

    constructor(address _victim) {
        victim = ArbitrumBranchBridgeAgent(_victim);
    }

    // Fallback function to simulate reentrancy
    fallback() external payable {
        victim.depositToPort(address(0x0), 1 ether); // This should fail due to the lock modifier
    }

    function attack() external payable {
        victim.depositToPort(address(this), 1 ether); // This will trigger the fallback function
    }
}

contract TestReentrancyProtection is DSTest {
    ArbitrumBranchBridgeAgent ArbitrumBranchBridgeAgentInstance;
    ReentrancyAttack attacker;

    function setUp() public {
        ArbitrumBranchBridgeAgentInstance = new ArbitrumBranchBridgeAgent();
        attacker = new ReentrancyAttack(address(ArbitrumBranchBridgeAgentInstance));
    }

    function testReentrancyAttack() public {
        // Fund the attacker contract
        address(attacker).transfer(2 ether);

        // Try the reentrancy attack
        bool success = address(attacker).call(abi.encodeWithSignature("attack()"));

        // The reentrancy attack should fail due to the lock modifier
        assertFalse(success, "Reentrancy attack succeeded!");
    }
}

```


## d) Architectural 

Ulysses Protocol has two major components, the Virtualized and the Unified Liquidity Management. The latter being inspired by the Delta Algorithm put forward by Layer 0;
https://www.dropbox.com/s/gf3606jedromp61/Delta-Solving.The.Bridging-Trilemma.pdf?dl=0

Ulyssess: It has two separate concepts as Virtualization and Unified
Virtualization: Allows the use of the same unit token across different networks
Unified: Allows merging the same volume tokens from different networks in the same ERC4626 vault
It uses Anyswap in `annyCall v7` to use these two architectures;
https://docs.multichain.org/developer-guide/anycall-v7


![image](https://github.com/code-423n4/2023-09-maia/assets/104318932/16504310-7e2b-4350-b0cd-8355e7fba64e)


#### Ports
The protocol's omnichain architecture has its foundations set around the Ports. These constitute the liquidity repositories which are in charge of retaining and managing capital deposited in each different chain where Ulysses is present.

How do they work?
In short, a Port is in play whenever assets are deposited into or withdrawn from the Ulysses Omnichain Liquidity System, in effect serving as a vault with a single-asset pool for each omnichain token active in a given chain. When interacting with a Port to perform any of these actions, no protocol fees will be charged from user's assets.

There are two types of Ports:
1. Branch Ports
2. Root Port

![image](https://github.com/code-423n4/2023-09-maia/assets/104318932/41e42c59-df3f-43df-8cc0-aeffe74f065c)


## e) Documents 


### - Philosophy - The Myth of Maia
The eldest of the seven nymphs that make up the constellation of Pleiades, Daughter of Atlas, who carries the world’s weight on his shoulders and mother of Hermes, messenger of the gods.
Spending most her life alone inside a cave in the peak of Mount Kyllene, not much is known about Maia other than she was a shy Earth goddess, devoting her life to nurture and whose very name translated to “Nursing Mother” the embodiment of growth! 🌿


### -Video explanation of the codes of the Project;

Video narration of the code of the project is effective, especially for auditors who want visual learning. [Project Video](https://maiaeco.notion.site/Code4rena-Code-Overview-3f455e66c457427ab3650a73a614641c#00dbbf98f5624342a1273a6c706112b6)



### -Project details;
https://v2-docs.maiadao.io/
In the documentation, the link of the [Hermes-Twitter](]https://twitter.com/HermesOminchain) account is broken at the bottom of the page, apart from that, the documents summarize the project in general terms.



### - Ulysses
Ulysses is devided in two separate concepts: Virtualized and Unified Liquidity. Virtualized liquidity is made possible by using anycall v7 as our messaging layer and means that an asset deposited from a specific chain, is recognized as a different asset from the "same" asset but from a different chain (ex: arb ETH is different from mainnet ETH). Unified Liquidity then unifies these tokens using a stableswap AMM and then depositing them in a Unified Liquidity Token, which is a Multi-Asset ERC4626 tokenized vault. This allows users to deposit any asset from any chain and receive a 1:1 representation of the underlying assets. These Unified Liquidity Tokens can then be used in any other protocol, like Uniswap v3.

All Maia Dao Projects-Documents Links;
[Twitter - MaiaDAOEco](https://twitter.com/MaiaDAOEco)
[Twitter - HermesOmnichain](https://twitter.com/HermesOmnichain)
[Discord - Maia DAO Ecosystem](https://discord.com/invite/gCggeM65y2)
[Telegram - Maia DAO ](https://t.me/maiadao)
[Medium - Maia DAO](https://medium.com/@maiaDAO)
[Website - Maia DAO](https://maiadao.io/#/stake)
[Website - Hermes](https://hermes.maiadao.io/#/swap)
[Maia Dao Documents ](https://docs.maiadao.xyz/)
[Hermes Documents](https://docs.maiadao.io/)
[Governance - Proposal Maia DAO](https://snapshot.org/#/maiadao.eth)


##  f) Centralization risks 
- DAO Mechanism

It is observed that the DAO proposals of the project are voted by a small number of people, for example this can be seen in the proposal below.As the project is new, this is normal in DAO ecosystems, but the centrality risk should be expected to decrease over time;

https://snapshot.org/#/maiadao.eth/proposal/0x38d9ffaa0641eb494c20d2b034df321a6865bf8859487468e1583dd095837488

Centralization risk in the DOA mechanism is that the people who can submit proposals must be on the whitelist, which is contrary to the essence of the DAO as it carries the risk of a user not being able to submit a proposal in the DAO even if he has a very high stake.

We should point out that this problem is beyond the centrality risk and is contrary to the functioning of the DAO. Because a user who has a governance token to be active in the DAO may not be able to bid if he is not included in the whitelist (there is no information in the documents about whether there is a warning that he cannot make a proposal if he is not on the whitelist)

There is no information on how, under what conditions and by whom the While List will be taken.
1- In the short term; Clarify the whitelist terms and processes and add them to the documents, and inform the users as a front-end warning in Governance token purchases.
2- In the Long Term; In accordance with the philosophy of the DAO, ensure that a proposal can be made according to the share weight.


- Centralization risk in contracts

There is a risk of centrality in the project as the onlyOwner The owner role has a single point of failure and onlyOwner can use critical functions, posing a centralization issue. There is always a chance for owner keys to be stolen, and in such a case, the attacker can cause serious damage to the project due to important functions.

The code detail of this topic is specified in automatic finding; https://github.com/code-423n4/2023-09-maia/blob/main/bot-report.md#m03-the-owner-is-a-single-point-of-failure-and-a-centralization-risk

Reducing the onlyOwner centrality risk in blockchain projects is essential to enhance security, decentralization, and community trust. Relying too heavily on a single owner or administrator can lead to various issues, such as single points of failure, potential abuse of power, and lack of transparency. Here are some strategies to mitigate the onlyOwner centrality risk:

1-Multi-Signature (Multi-Sig) Wallets: Instead of having a single owner with full control, you can implement multi-signature wallets. Multi-sig wallets require multiple parties to sign off on transactions or changes to the smart contract, making it more decentralized and secure.

2-Timelocks and Governance Delays: Introduce timelocks or governance delays for critical operations or updates. This means that proposed changes need to wait for a specified period before being executed. During this period, the community can review and potentially veto the changes if they identify any issues.

3-Decentralized Governance Mechanisms: Implement decentralized governance mechanisms, such as DAOs (Decentralized Autonomous Organizations) or community voting systems. This allows token holders or stakeholders to have a say in important decisions, making the project more community-driven.

Great summary of GalloDaSballo with examples of centrality risk and how to reduce it in previous Code4rena audits; https://gist.github.com/GalloDaSballo/881e7a45ac14481519fb88f34fdb8837





##  g) Systemic risks 
According to the information of the project, it is stated that it can be used in rollup chains and popular EVM chains.

```js
README.md:
  - Does it use a side-chain?: true
```
We can talk about a systemic risk here because there are some Layer2 special cases.
Some conventions in the project are set to Pragma ^0.8.0, allowing the conventions to be compiled with any 0.8.x compiler. The problem with this is that Arbitrum is [Compatible](https://developer.arbitrum.io/solidity-support) with 0.8.20 and newer. Contracts compiled with these versions will result in a non-functional or potentially damaged version that does not behave as expected. The default behavior of the compiler will be to use the latest version which means it will compile with version 0.8.20 which will produce broken code by default.


## i) Security Approach of the Project

### Successful current security understanding of the project;
1 - First they did the main audit from a reputable auditing organization like Zellic  and resolved all the security concerns in the report

2- They manage the 2nd audit process with an innovative audit such as Code4rena, in which many auditors examine the codes.

### What the project should add in the understanding of Security;

1- By distributing the project to testnets, ensuring that the audits are carried out in onchain audit. (This will increase coverage)

2- After the project is published on the mainnet, there should be emergency action plans (not found in the documents)

3- Add On-Chain Monitoring System; If On-Chain Monitoring systems such as Forta are added to the project, its security will increase.

For example ; This bot tracks any DEFI transactions in which wrapping, unwrapping, swapping, depositing, or withdrawals occur over a threshold amount. If transactions occur with unusually high token amounts, the bot sends out an alert. https://app.forta.network/bot/0x7f9afc392329ed5a473bcf304565adf9c2588ba4bc060f7d215519005b8303e3

4- After the Code4rena audit is completed and the project is live, I recommend the audit process to continue, projects like immunefi do this. https://immunefi.com/

5- Pause Mechanism
This is a chaotic situation, which can be thought of as a choice between decentralization and security. Having a pause mechanism makes sense in order not to damage user funds in case of a possible problem in the project.

6- Upgradability
There are use cases of the Upgradable pattern in defi projects using mathematical models, but it is a design and security option.

## i) Other Audit Reports and Automated Findings 

**Automated Findings:**
https://github.com/code-423n4/2023-09-maia/blob/main/bot-report.md

**Slither Report:**
https://github.com/code-423n4/2023-09-maia/blob/main/slither.txt

**Other Audit Reports (Zellic):**
[Zellic Audit Report](https://github.com/code-423n4/2023-05-maia/blob/main/audits/Ulysses%20Protocol%20May%202023%20-%20Zellic%20Audit%20Report.pdf)

While investigating the potential risks in the project, the past audit reports can give us serious insights, so they should be taken into account and analyzed in detail. 



## Zellic Audit Report - Ulysses Protocol
| Vulnerability |Risk Level |Description |Category|Remediation|
|:--|:----------------|:------|:------|:------|
| 3.1|Critical Risk |Lack of the RootBridgeAgent contract verification   | Coding Mistakes   | OK✓ |
| 3.2|Critical Risk |  Missing access control for Multicall | Coding Mistakes   | OK✓ |
| 3.3|Critical Risk |  Multitoken lacks validations | Coding Mistakes   | OK✓ |
| 3.4|Critical Risk|  Multiple redeems of the same deposit are possible  | Coding Mistakes   | OK✓ |
| 3.5|Critical Risk | Broken fee sweep  | Coding Mistakes   | OK✓ |
| 3.6|High | Asset removal is broken | Coding Mistakes   | OK✓ |
| 3.7|High | Unsupported function codes  | Coding Mistakes   | OK✓ |
| 3.8|High |  Missing access control on ``anyExecuteNoSettlement``  | Coding Mistakes   | OK✓ |
| 3.9|High|  The protocol fee from pools will be claimed to zero address| Coding Mistakes  | OK✓ |
| 3.10|High | Unlimited cross-chain asset transfer without deposit requirement | Coding Mistakes   | OK✓ |
| 3.11|Medium |  ChainId type confusion  | Coding Mistakes   | OK✓ |
| 3.12|Medium| Incorrect accounting of total and strategies debt  | Coding Mistakes   | OK✓ |
| 3.13|Medium |  Bridging assets erroneously mints new assets | Coding Mistakes   | OK✓ |
| 3.14|Low | Lack of input validation  | Coding Mistakes   | OK✓ |
| 3.15|Informational |  Lack of new owner address check |Coding Mistakes  | OK✓ |
| 3.16|Informational |  Addresses may accidentally be overwritten |Coding Mistakes  | OK✓ |
| 3.17|Informational | Ownable contracts allow renouncing |Coding Mistakes  | OK✓ |
| 3.18|Informational | Lack of validation in UlyssesFactory |Coding Mistakes  | OK✓ |



## j) Gas Optimization
The project is generally efficient in terms of gas optimizations, many generally accepted gas optimizations have been implemented, gas optimizations with minor effects are already mentioned in automatic finding, but gas optimizations will not be a priority considering code readability and code base size

Permit2 can integrated  into your project, this will reduce  gas usage;
https://medium.com/morpho-labs/how-permit2-improves-the-user-experience-of-morpho-1fdbfb5843fe
https://github.com/Uniswap/permit2/issues/232


##  k) Packages and Dependencies Analysis 📦

| Package                                                                                                                                     | Version                                                                                                               | Usage in the project                               | Audit Recommendation                                                                                                             |
| ------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- | -------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| [`openzeppelin`](https://www.npmjs.com/package/@openzeppelin/contracts)         | [![npm](https://img.shields.io/npm/v/@openzeppelin/contracts.svg)](https://www.npmjs.com/package/@openzeppelin/contracts)                         | IERC1155Receiver.sol, ERC1155Receiver.sol, IERC721Receiver.sol   |-  Version `4.9.0` is used by the project, it is recommended to use the newest version `4.9.3`                                                                                                                  |
| [`solady`](https://www.npmjs.com/package/solady)                                                                                        | [![npm](https://img.shields.io/npm/v/solady)](https://www.npmjs.com/package/solady)   | Ownable.sol, </br> LibZip.sol, </br> SafeTransferLib.sol   |  - The latest updated version is used.  </br> - An updated audit of the solady library was done by SpearBit last week, I recommend checking this out [Audit Report](https://github.com/Vectorized/solady/blob/main/audits/cantina-solady-report.pdf)                                                                                  |
| [`solmate`](https://www.npmjs.com/package/solmate)                                                                                        | [![npm](https://img.shields.io/npm/v/solmate.svg)](https://www.npmjs.com/package/solmate)   | solmate/tokens/ERC20.sol </br> solmate/tokens/ERC721.sol                                                                                     |  - The latest updated version is used. 
| [nomad-xyz](https://github.com/nomad-xyz/ExcessivelySafeCall )                                                                                       | [![npm](https://img.shields.io/npm/v/@nomad-xyz/contracts-core.svg)](https://www.npmjs.com/package/@nomad-xyz/contracts-core)   |lib/ExcessivelySafeCall.sol  |  - The latest updated version is used.  </br> - This solidity library helps you call untrusted contracts safely. Specifically, it seeks to prevent all possible ways that the callee can maliciously cause the caller to revert. Most of these revert cases are covered by the use of a [low-level call](https://solidity-by-example.org/call/). The main difference with between address.call()call and address.excessivelySafeCall() is that a regular solidity call will automatically copy bytes to memory without consideration of gas.                         

## l) Other recommendations

✅ The use of assembly in project codes is very low, I especially recommend using such useful and gas-optimized code patterns; https://github.com/dragonfly-xyz/useful-solidity-patterns/tree/main/patterns/assembly-tricks-1

✅ It is seen that the latest versions of imported important libraries such as Openzeppelin are not used in the project codes, it should be noted. https://security.snyk.io/package/npm/@openzeppelin%2Fcontracts/

✅A good model can be used to systematically assess the risk of the project, for example this modeling is recommended; https://www.notion.so/Smart-Contract-Risk-Assessment-3b067bc099ce4c31a35ef28b011b92ff#7770b3b385444779bf11e677f16e101e

✅ Due to `onlyOwner` powers; It is better to override `renounceOwnership` so that it cannot be used.

✅ Use the constant.sol pattern, if all constants are collected in one file this provides a more compact and systematic structure

✅My suggestion is to improve the naming of contracts, design high-level architectural resources such as diagrams, user and contract interaction flows from different participants' perspectives, and further expand documentation

✅ Economic risk audit; Having the economic models audited separately, as professional projects have done recently, will provide higher security. For example: [@Risk_DAO](https://riskdao.org/) project only performs economic structure inspections
 
✅ Formal specification & invariant testing ; Having formal specification & invariant testing done separately will provide higher security by experts on this subject. For example: [@agfviggiano](https://twitter.com/agfviggiano) only does these checks

## m) New insights and learning from this audit 

1- I learned the concept of `Virtualized liquidity` and  `Arbitrary Cross-Chain Execution` 

2-  I learned the concept of [Layer0](https://layerzero.gitbook.io/docs/)


## n) Summary of Vulnerabilities 

The list and details of all the results that I analyzed the project and shared in Code4rena are shared separately.

 Audit Results
- [x] **Medium 01** → ERC1155 tokens coming to the 'VirtualAccount.sol' contract are stuck in the contract because there is no withdraw function
- [x] **Low 01** → Head Overflow Bug in Calldata Tuple ABI-Reencoding
- [x] **Low 02** → Important `_fee` return value doesn’t check
- [x] **Low 03** → `remoteBranchExecutionGas`  must be lower than `gasLimit`
- [x] **Low 04** → If the same values are used for status values in the same contract, this may cause logic errors
- [x] **Low 05** → Project Upgrade and Stop Scenario should be
- [x] **Low 06** → Array lengths doesn't check  a few Struct
- [x] **Low 07**→ Missing Event for  initialize
- [x] **Non-Critical 08**→ The function updates addresses in bulk. This can lead to unnecessary transaction costs and complexity.

## o) Time spent on analysis
I allocated about 6 days to the project, most of this time was spent with architectural examination and analysis.


### Time spent:
24 hours