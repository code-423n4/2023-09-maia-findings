
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol
GAS
1)Modifiers for Access Control:  For functions that require the owner's access, you can make use of the onlyOwner modifier directly in the function signature. This can make the code more readable and save gas by avoiding the modifier check in a separate modifier.
function addBranchToBridgeAgent(...) external payable onlyOwner {
    // ...
}
2)Gas Optimization:Consider using the view and pure functions where applicable. For example, functions like name(), symbol(), and decimals() in your _addGlobalToken function do not modify the state and can be marked as view.


https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol

GAS
1)Gas Limit Parameters: In functions like _performCall and _performRetrySettlementCall, you're encoding gas limit parameters in the payload. While this can be useful for flexibility, consider if it's necessary in all cases. If the gas limit is often the same, you could make it a constant value, saving gas on encoding and decoding.
// Define a constant gas limit for the entire contract
uint256 constant private GAS_LIMIT = 100000; // Replace with your desired gas limit

function _performCall(uint16 _dstChainId, address payable _refundee, bytes memory _payload) internal {
    address callee = getBranchBridgeAgent[_dstChainId];

    if (callee == address(0)) revert UnrecognizedBridgeAgent();

    if (_dstChainId != localChainId) {
        // Eliminate the need to encode the gas limit in the payload
        ILayerZeroEndpoint(lzEndpointAddress).send{value: msg.value}(
            _dstChainId,
            getBranchBridgeAgentPath[_dstChainId],
            _payload,
            _refundee,
            address(0),
            abi.encodePacked(uint16(2), GAS_LIMIT, uint16(20000), callee) // Use GAS_LIMIT instead of dynamically calculated gas limit
        );
    } else {
        callee.call{value: msg.value}("");
        IBranchBridgeAgent(callee).lzReceive(0, "", 0, _payload);
    }
}
By using a constant gas limit, you eliminate the need to calculate and encode it in the payload, which can save gas. However, keep in mind that this approach makes the gas limit fixed for all transactions using this function. If you require flexibility in setting the gas limit on a per-transaction basis, then encoding it dynamically may still be necessary.
2)Function Modifiers: Be cautious when using modifiers like lock. Modifiers often introduce some overhead, and you should ensure that the gas cost for checking these modifiers doesn't become excessive.
modifier lock() {
    require(_unlocked == 1);
    _unlocked = 2;
    _;
    _unlocked = 1;
}

This modifier has a few operations, such as the require statement and setting _unlocked to different values. While this modifier may serve a specific purpose in your contract, you should carefully assess whether the gas cost associated with it is justified by its use case.
Some considerations for using modifiers like lock in a proof-of-concept (PoC) contract:
A)Gas Cost vs. Functionality: Determine whether the added functionality provided by the modifier justifies the extra gas cost. In a PoC, you may prioritize simplicity and gas efficiency over complex modifiers.
B)Alternative Approaches: Consider whether there are alternative approaches to achieve the same functionality with fewer gas costs. For example, you might use a simple boolean flag to achieve similar locking behavior.
C)Gas Profiling: Profile your contract's gas consumption during PoC testing to ensure that the modifier does not introduce excessive overhead, especially in frequently called functions.
D)Gas Improvements: If the modifier is deemed necessary but introduces high gas costs, you can explore ways to optimize its implementation or look for gas-efficient alternatives.