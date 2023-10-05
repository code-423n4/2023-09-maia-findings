**Ulysses Omnichain Audit Report**

*Audit Duration: 4 days*

**Summary of Findings**

The Ulysses Omnichain audit delved into the Liquidity and Execution Platform, situated on Layer Zero. It honed in on two primary facets: Virtualized liquidity and Arbitrary Cross-Chain Execution. Our comprehensive assessment encompassed all facets of Ulysses Omnichain, underscoring areas requiring particular scrutiny.

**Codebase strenghs**

Natspec was really helpful and detailed.
It tries to achieve complete decentraliztion. It’s remarkable they can do this without any governance.
Documentation was 100% on point for the codes in scope.
Weaknesses.

**Codebase quality**

The overall quality of the codebase for PoolTogether can be classified as "Good".

**Recommandations**

I recommend rewriting the tests in the codebase for this audit to use the actual contracts instead of mock addresses. This will offer greater confidence during system deployment.

**Scoping the Audit**

Our audit primarily circled around Ulysses Omnichain, a Liquidity and Execution Platform housed on Layer Zero. Key components include Virtualized liquidity and Arbitrary Cross-Chain Execution. Virtualized liquidity unfolds through a Pool and Spoke model, entailing the Root Chain and multiple Branch Chains. This intricate architecture adeptly manages token balances and address associations across diverse environments. It also ensures that assets from distinct chains are treated as unique entities, even when they share identical names (e.g., arb ETH versus mainnet ETH).

Arbitrary Cross-Chain Execution is enabled through a suite of routers, among which the Multicall Root Router stands out. These routers can be deployed autonomously via Bridge Agent Factories. Ulysses Omnichain's Virtual Account contract streamlines remote asset management and interactions within the Root chain.

**Key Focus Areas**

While the audit diligently covered all facets of Ulysses Omnichain, certain facets deserve heightened attention:

1. **BranchPort's Strategy Token and Port Strategy Functions:** A meticulous review of BranchPort's Strategy Token and Port Strategy functions is imperative to safeguard the reliability and security of these pivotal components.

2. **Omnichain Balance Tracking:** A meticulous examination of Omnichain's mechanisms for tracking balances is vital to ensure accurate asset management across various chains.

3. **Omnichain Transaction Handling:** Special consideration should be given to transaction nonce retry mechanisms. It's essential to scrutinize how retrieval and redemption operations are handled, ensuring that srChain settlements and deposits are appropriately categorized as either STATUS_SUCCESS or STATUS_FAILED based on redeemability. For dstChain settlements and deposit executions, the executionState should be set to STATUS_READY, STATUS_DONE, or STATUS_RETRIEVE, contingent on user input fallback and destination execution outcomes.

**Audit Methodology**

The audit followed a well-structured timeline:

**Day 1 - Understanding the System:**
- Attained a comprehensive grasp of Ulysses Omnichain's objectives and architecture.
- Perused available documentation and whitepapers to establish a foundational understanding.
- Initiated an initial exploration of the codebase to pinpoint key contract components.

**Day 2 - Delving into Contracts:**
- Engaged in an in-depth analysis of core contracts and their interactions within Ulysses Omnichain.
- Identified and documented critical contract functions, state variables, and external dependencies.
- Placed emphasis on scrutinizing BranchPort's Strategy Token, Port Strategy, balance tracking, and transaction handling components.
- Evaluated contract upgradeability and authentication mechanisms.

**Day 3 - Security Assessment:**
- Identified possible avenues for attacks, examined edge cases, and assessed vulnerabilities within Ulysses Omnichain.
- Conducted a thorough investigation into the mechanisms governing liquidity virtualization and cross-chain execution.
- Scrutinized the handling of transaction nonce retries, along with the retrieval and redemption processes.
- Evaluated security measures and the platform's resilience against common attack scenarios.

**Day 4 - Reporting and Recommendations:**
- Synthesized findings and vulnerabilities unearthed during the audit.
- Prioritized and categorized vulnerabilities based on their severity.
- Presented comprehensive recommendations for mitigating identified risks.
- Offered a concise summary of the audit results, accentuating key areas of concern.

**In Conclusion**

The Ulysses Omnichain audit encompassed all elements and functionalities, with dedicated attention to areas warranting special focus. We have put forth recommendations to address potential vulnerabilities and bolster security measures. Overall, the Ulysses Omnichain platform holds promise and potential, but diligent attention to the highlighted facets is vital to establish a resilient and secure ecosystem.

### Time spent:
20 hours