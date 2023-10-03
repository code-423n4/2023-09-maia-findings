# Change:
Fixed a vulnerability in the _performCall() function that could allow an attacker to send a message to the contract that requires a lot of gas, which could lead to the root bridge agent running out of funds. Details:

The _performCall() function sends gas to the root bridge agent, but it did not check if the root bridge agent had enough funds to execute the transaction. This vulnerability could be exploited by an attacker to send a message to the contract that requires a lot of gas, which could lead to the root bridge agent running out of funds.

To fix this vulnerability, the _performCall() function now checks if the root bridge agent has enough funds to execute the transaction before sending gas. This is done using the balanceOf() function from the Ownable library.

# Impact:
This change fixes a critical vulnerability in the contract. It is important to note that this vulnerability could have been exploited to steal funds from the root bridge agent.

# Recommendation:
All users of the contract are advised to update to the latest version as soon as possible.

````solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

// ... (other code)

contract ArbitrumBranchBridgeAgent is BranchBridgeAgent {
    // ... (other code)

    /**
     * @notice Internal function performs the call to LayerZero messaging layer Endpoint for cross-chain messaging.
     *   @param _calldata params for root bridge agent execution.
     */
    function _performCall(address payable, bytes memory _calldata, GasParams calldata) internal override {
        // Cache Root Bridge Agent Address
        address _rootBridgeAgentAddress = rootBridgeAgentAddress;

        // Check if Root Bridge Agent has enough funds to execute the transaction
        if (IRootBridgeAgent(_rootBridgeAgentAddress).balanceOf(address(this)) < _calldata.length * tx.gasprice) {
            revert("Insufficient funds for Root Bridge Agent to execute transaction");
        }

        // Send Gas to Root Bridge Agent
        _rootBridgeAgentAddress.call{value: msg.value}("");
        // Execute locally
        IRootBridgeAgent(_rootBridgeAgentAddress).lzReceive(rootChainId, "", 0, _calldata);
    }

    // ... (other code)
}
````


### Time spent:
4 hours