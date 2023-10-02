# Bauchibred's # Analysis of the Maia DAO - Ulysses Codebase

## Approach

The auditing process commenced with a high level but brief overview of the entire Maia DAO ecosystem. This wasn't restricted merely to the recent updates in the Ulysses scope. Subsequent to this, several hours were dedicated to examining findings from previous audits and drawing insights from them. This was followed by a meticulous, line-by-line security review of the sLOC, beginning from the Root contracts and extending down to the Branch contracts and even the Arbitrum-specific contracts. The concluding phase of this process entailed interactive sessions with the developers on Discord. This fostered an understanding of unique implementations and facilitated discussions about potential vulnerabilities and observations that I deemed significant, requiring validation from the team.

## Architecture Feedback

- While the code documentation currently sits at a medium-acceptable level, there's potential for enhancement. This is cause a test coverage of 69% is not really a wide coverage and leaves room for improvement.
- Extensive usage of factories also add an extra layer of complexity but that is unfortunately unavoidable in the web3 space, especialy with the fact that the Omnichain system aims to implement cross-chain messaging and asset transfer, and with the update it now uses LayerZero to help with this, which naturally is a complex challenge and inherently requires complex code.
- Would be important to also note that `Solmate ERC20` implementation is being used , and this implementation allows [transfer to the 0x0 address](https://github.com/transmissions11/solmate/blob/main/src/tokens/ERC20.sol#L76-L110). This could have negative repercussions in the future. As a suggestion, add checks that no transfers can go to zero address or user OpenZeppelin's ERC20 implementation.
- The current implementation utilizes Balancer's composable stable pools for hTokens and their underlyings to address discrepancies in token decimal counts compared to the prior codebase. While this resolves certain challenges, it also introduces third-party risks from Balancer. MAIA lacks control over Balancer pool actions, leaving them vulnerable to pauses or potential reactionary measures like Balancer's recent activation of the `proportional-exit` feature following a security breach. This underscores the inherent risks associated with increased external integrations and the need for caution.
- Some contracts, functions/variables need better naming as in some cases the name has nothing to do with the functionality. Take the the internal function used to check if a Port Strategy has reached its daily management limit as an example: [`_checkTimeLimit()`](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L485), the current implementation is not actually a 24 hour limit which in a way goes against the naminng, so a better documentation/naming around this would have been better, talking about the `_checkTimeLimit()` function it's current implementation is to ensure that it always counts from a specific time on a daily basis, _say 00:00 for this Analysis sake_ , but the current implementation does not take in to account that the issue of leap seconds/years and this could lead to an issue if the exact time law must be followed.
- Some of the provided Router contracts had functions that weren't implemented (reverting by default), making it difficult to assess whether there could be potential issues arising from that, sinc ethese functions could be extended to in the future by others would be key to make them be in the know that they should be careful for the type of integration that's been added.
- Wouldn't be fair if i don't comment on the ASCII art on the contracts, it's a nice job:)

## Centralization Risks

- Upholding trust assumptions is pivotal for the project's longevity. Malicious or compromised ownership can drain liquidity from a vast portion of the project. This can be orchestrated by embedding malicious strategies (Gauges) within existing project reward components or by introducing harmful reward tokens.
- The above is also applicable to Bridge Agents, currently, while _adding/toggling_ them, there are absolutely no checks. Once activated, these agents possess unchecked authority to mint or burn hTokens as they see fit. So a safeguard here could be to empower the admin to exclusively invoke the `toggleBridgeAgent()` function on a Bridge Agent address recognized by the `BranchPort`. Another alternative might be to limit admin capabilities to only removing Bridge Agents, though i weigh more on the former.

## Systemic Risks

- Maia DAO has one of the most prolific development team I've seen, would put them on a scale A- if the Chainlink is on scale A+
  A major decider for my choice of grading is the fact that the core developers are relatively small. This results in a very low [project bus](https://en.wikipedia.org/wiki/Bus_factor) value. If anything happens to one of the team members, development will be drastically stalled or even project may be closed down, this can be sween to be validated based on my _Other recommendations_ section, cause during the duration of the contest, I could only get a hold of `0xbuzzlightyear` which means other security researchers most probably faced the same issue and if something might have happened to `0xbuzzlightyear` the whole experience would have been massively affected. Getting more team members should be prioritzed.
- There is a possibility of completely unexpected results if an L2 chain goes down.
- Each chain has some differences from one another, and it may be possible that some chains do not synchronize well with others due to differences in say block timing for example, this may be very problematic when managing funds across chains.
- From a token/liquidity point of view, I believe the protocol does not have any more of a system risk that other DeFi products. Its spread to integrating balancer composable pools, liquidity mining, staking, and strategies, which gives the project a better chance of surviving any black swan type events that may occur.

## Testability

The system's testability is commendable. While the test suite has imperfections – like a 69% coverage rate and occasionally intricate lengthy tests that are a bit hard to follow, the latter shouldn't be considered a flaw though, tcause once familiarized with the setup, devising tests and creating PoCs becomes a smoother endeavor. The developers have provided a high-quality testing sandbox for implementing, testing, and expanding ideas.

## Other Recommendations

### **Protocol Requires Additional Developers**

While the MaiaDAO developers have showcased commendable prowess, the rapid evolution of the system – notably _the shift from `anyCall` to `LayerZero` for cross-chain messaging_ – with a limited workforce has inevitably led to oversight in certain aspects. Evidence of this can be gleaned from codebase typos. Drawing from the [broken windows theory](https://en.wikipedia.org/wiki/Broken_windows_theory), such lapses may signify underlying code imperfections.

Another evident example is the fact that tests only cover 69% of the whole codebase among others, not to forget that during the contest most of the questions in the public contest's discord chat was answered by the OG `0xbuzzlightyear` , privately I only received answers from him too, all these reinforces the idea that the team needs more developers and eyes on the project.

### **Improve Testability**

As already stated in this report, whereas 69% is not a bad amount of coverage since it provides security researchers to build on the already available test suite, it still isn't enough and protocol should be aiming at nothing less than 95% of test coverage,

### **Leverage Additional Auditing Tools**

The majority of security researchers use Visual Studio Code with some specific plugins. A very popular one is the [Solidity Visual Developer](https://marketplace.visualstudio.com/items?itemName=tintinweb.solidity-visual-auditor) which does not integrate with this protocol, there are others but that I beleive is atleast a very good one that should be used.

### **Enhance Event Monitoring**

Pesent implementation subtly suggests that event tracking isn't maximized. Instances where events are missing or seem arbitrarily embedded have been observed. A shift towards a more purposeful utilization of events is recommended.

## Security Researcher Logistics

My attempt on reviewing the updated Maia DAO - Ulysses Codebase spanned 45 hours distributed over a week:

- 2 hours dedicated to writing this analysis.
- 5 hours exploring the previous MAIA contest on code4rena, viewable [here](https://github.com/code-423n4/2023-05-maia-findings).
- 3 hours were allocated for discussions with sponsors on the private discord group regarding potential vulnerabilities.
- The remaining time on finding issues and writing the report for each of them on the spot, later on _editing a few with more knowledge gained on the protocol as a whole, or downgrading them to QA reports_

## Conclusion

The codebase was a very great learning experience, though it was a pretty hard nut to crack, being that it's like an update contest and most _easy to catch_ bugs have already been spotted and mitigated from the previous contest.

During the course of my security review, I encountered only one _High_ severity finding related to how all deposits attached to a specific nonce could become stuck in the protocol for a user. The cause of this bug, in reality, would likely originate from one of the tokens whose deposits need to be redeemed. However, this would affect all the deposits attached to that specific nonce since the redemption is executed in a loop method; a reversion in one translates to a reversion in all.

Additionally, I submitted two potentially _Medium-worthy_ findings:

- One involves the current MulticallRootRouter, which could revert when a correct integration is made by a 3rd party dApp passing the supported function IDs, namely _0x04, 0x05, 0x06_.
- The second pertains to the potential for an attacker to block the LayerZero channel. This vulnerability arises from missing access control on the `callOut` prefixed functions and an absence of a minimum gas pass check when the execution reaches _performCall()_.


### Time spent:
42 hours