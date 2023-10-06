# Maia DAO Analysis

**1. Analysis of the Codebase:**

The Maia DAO project demonstrates a commitment to best practices and code quality. Several unique aspects and existing patterns can be observed:

- **Unique Aspects**:
    - Virtualized account creation and management.
    - Port strategy tokens and allow for the opportunity to earn a yield on locked tokens.
- **Existing Patterns**:
    - Cross-chain bridging through locking on the depositing chain and subsequently minting on the receiving chain.
    - Router and pool mechanism for executing functions and storing tokens similar to Uniswap
    - Cross-chain messaging using LayerZero Protocol

**2. Architecture Feedback:**

[Architecture of Maia DAO](https://github.com/zzzuhaibmohd/Code4rena/blob/main/Architecture%20of%20Maia%20DAO.png)

At the core of Branch architecture is the BranchPort, with one port being dedicated to each branch or “chain”. These BranchPorts store and lock the user's tokens for cross-chain messages. The CoreBranchRouter and BranchBridgAgent serve as crucial entry points for various operations within the system. 

The router and the BridgeAgent, are users’ entry points for users to interact with the system. Each branch has a BrigeAgentFactory that allows for the deployment of additional BridgeAgents. Each BridgeAgent is attached to a BranchRouter. This includes ArbitrumBranch as well.

The Root, only exists on Arbtrum. The is separate from the AbritrumBranch though which acts more as a separate Branch for Arbitrum. 

The Root contracts have one RootPort with one CoreRootRouter (with one RootBridgeAgent). There can also be multiple RootBridgeAgentFactories which can create multiple RootBridgeAgents. Systematically, each BridgeAgent is attached to one Root Router.

This architecture lends itself to be a bit confusing. With two different contracts in which the user can enter the system it unnecessarily overexposes the users to more systemic risk. It's not clear why this decision was made, if possible considering limiting entry points to only one contract similar to Uniswaps Router and Pool System.

The following diagram illustrates the execution flow for cross-chain deposits.

[Execution Flow of Adding a Bridge Agent](https://github.com/zzzuhaibmohd/Code4rena/blob/main/Execution%20Flow%20of%20Adding%20a%20Bridge%20Agent.png)

**Contracts Overview:**

***ArbitrumBranchBridgeAgent.sol***

- The `retrySettlement` is not currently implemented. As a standard practice, consider removing redundant code from the contracts and adding it back when needed in the future.

***BaseBranchRouter.sol***

- Perform sanity checks on the length of the array in the `_transferAndApproveMultipleTokens` function.
- Some ERC20 tokens (like USDT) do not work when changing the allowance from an existing non-zero allowance value. As a result, the function will revert if the current approval is not zero.

***BranchBridgeAgent.sol***

- In `callOutAndBridgeMultiple`, have a sanity check to make sure that the length of arrays of token does not exceed 255, since the payload restricts the length to be as a `uint8` type.
- Implement a return value check when making low-level function calls and external calls. For Example The `excessivelySafeCall` function lacks a return value check.
- Make sure **not to hardcode** values when interacting with LayerZero endpoints. There are two instances of this in the code. First is the `_zroPaymentAddress` is hardcoded to `address(0)` which prevents the Maia DAO contracts from using LayerZero tokens as fees in the future. The second example is the `_adapterParams` set to `bytes(0)` which is used for custom functionality.

***BranchPort.sol***

- The `bridgeInMultiple` function should include a sanity check to ensure that `length **>** **255**`, similar to the check present in `bridgeOutMultiple`.
- The `setCoreBranchRouter` function should include a check to verify if `_coreBranchBridgeAgent` is already added to the `bridgeAgents` array. Implement an `isBridgeAgent[_coreBranchBridgeAgent]` check within the function or else it may result in duplicate values.
- `addPortStrategy` can be used to toggle a port strategy back to true if the owner ends up being malicious. Consider the following, port strategy is added via `addPortStrategy` its then toggled off with `togglePortStrategy` it can then be toggled back on without calling the `togglePortStrategy` function as it calling `addPortStrategy` again with the same `_portStrategy` address will toggle the bool back to true. Consider adding a check that the port strategy doesn’t already exist in the `addPortStrategy` function.
- This is the same for `addBridgeAgent` and `toggleBrigdeAgent`.

 ***MulticallRootRouter.sol***

- `safeApprove` is deprecated. Instead, use `safeIncreaseAllowance` and `safeDecreaseAllowance` in the `_approveMultipleAndCallOut` and `_approveAndCallOut` functions.

***RootBridgeAgent.sol***

- When calling the `estimateFees` function of LayerZero, the `_payInZRO` parameter is hardcoded to `false`. To allow for payment via ZRO tokens in the future, consider passing it as an argument instead of hardcoding it.
- Similarly, when calling the `send` function via LayerZero endpoint, the `_zroPaymentAddress` parameter is hardcoded to `address(0)`. To accommodate future payments via ZRO tokens, consider passing this parameter as an argument.
- The best practice for integrating with LayerZero suggests that the `_adapterParams` parameter should not be set to `bytes(0)`. Instead of hardcoding it to `bytes(0)`, consider passing it as a function argument. This parameter is used for custom functionality that may not be required now but could be useful in the future.

***VirtualAccount.sol***

- Even though the contract accepts `ERC1155` tokens, there is currently no function provided to withdraw `ERC1155` tokens from the contract.

**3. Centralization Risks:**

While Maia DAO's design aims to maintain decentralization, there are centralization risks:

- **Permission Management**: The absence of robust permission management could potentially lead to controller exploitation if overlapping permissions are granted. Implementing checks and balances is recommended to mitigate this risk.

**4. Systemic Risks:**

Systemic risks relate to potential issues affecting the entire system. In Maia DAO, these include:

- **Payload Length Limits**: Ensuring that payload length limits, especially in **`uint8`** type variables, are not exceeded is crucial to prevent issues.
- **Return Value Checks**: Implementing return value checks when making low-level or external calls is necessary to ensure contract security.
- **Avoiding Hardcoded Values**: Hardcoding values for LayerZero endpoints can limit flexibility. Instead, parameters should be passed as arguments.

Highly recommend reviewing the LayerZero docs and reimplementing the endpoint using their contracts available via npm. An overwhelming majority of the issues we found we related to improper implementation leading to the failling of messages or blocking of channels. It is always recommended to follow the docs and suggested implementation as much as possible to avoid issues such as these. Asking for assistance in their Discord is also an option if you’re ensure about secure implementation.

**5. Other Recommendations:**

To enhance the Maia DAO project, the following recommendations are made:

- Ensure the **`ERC1155`** token withdrawal functionality is provided in the VirtualAccount.sol contract.
- Consider passing parameters as arguments rather than hardcoding them for flexibility in the future.
- Implement Gas Params in the **`addLocalToken`** function.
- Include sanity checks on array lengths in relevant functions.
- Implement checks to prevent overlapping permissions in the permission management system.

**6. Time Spent:**

The analysis was conducted within approximately 60 hours, ensuring a thorough examination of the codebase, architecture, risks, and recommendations.

**Overall Best Practices to Follow**

- When making low-level calls, always verify the return value of the call.
- As a best practice, employ a two-step ownership transfer via `Ownable2Step.sol`.
- Adhere to the Checks-Effects-Interaction Pattern.
- Prefer the OpenZeppelin `nonReentrant` modifier over a custom `lock` modifier.
- Take note that variables related to `chainId` are currently defined with a `uint16` data type (e.g., `localChainId`, `rootChainId`, etc.). The maximum value a `uint16` type can hold is 65535. If the project intends to deploy the contract on networks with a `chainId` greater than 65535, such as Aurora (1313161554) or Harmony Mainnet (1666600000), the deployment will fail due to overflow. As a best practice, consider changing the type to `uint256`.
- When declaring state variables as `immutable` and involving third-party components, conduct thorough research. In the context of the current contract, `lzEndpointAddress` is defined as `immutable`. If LayerZero plans to migrate to a new version due to a bug or security risk, all contracts could become unusable.
- For `payable` functions, always verify whether `msg.value > 0` to prevent transactions from failing and provide a better user experience.

### Time spent:
148 hours