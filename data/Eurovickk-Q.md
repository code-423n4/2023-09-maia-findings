https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol

1)Function Order: In the constructor, you set the localChainId and rootPortAddress before calling the parent constructor. This is fine as long as the parent constructor does not rely on these values during its execution. Ensure that the order of operations in the constructor is correct.
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol

1)Code Reusability: You have some code duplication in functions like executeNoDeposit, executeWithDeposit, executeSignedNoDeposit, and executeSignedWithDeposit. You can potentially create a private helper function to handle common functionality to reduce redundancy.
2)Modifiers for Access Control: You're using the onlyOwner modifier for access control, which is good. However, consider using msg.sender == owner() directly in functions where applicable to improve code readability.
A)In your executeSystemRequest function:
function executeSystemRequest(address _router, bytes calldata _payload, uint16 _srcChainId)
    external
    payable
{
    require(msg.sender == owner(), "Only owner can call this function");
    IRouter(_router).executeResponse(_payload[PARAMS_TKN_START:], _srcChainId);
}

B)In your executeNoDeposit function:
function executeNoDeposit(address _router, bytes calldata _payload, uint16 _srcChainId)
    external
    payable
{
    require(msg.sender == owner(), "Only owner can call this function");
    IRouter(_router).execute{value: msg.value}(_payload[PARAMS_TKN_START:], _srcChainId);
}
C)In your executeWithDeposit function:
function executeWithDeposit(address _router, bytes calldata _payload, uint16 _srcChainId)
    external
    payable
{
    require(msg.sender == owner(), "Only owner can call this function");
    DepositParams memory dParams = _parseDepositParams(_payload);

    _bridgeIn(_router, dParams, _srcChainId);

    if (_payload.length > PARAMS_TKN_SET_SIZE) {
        IRouter(_router).executeDepositSingle{value: msg.value}(
            _payload[PARAMS_TKN_SET_SIZE:], dParams, _srcChainId
        );
    }
}

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol

1)Parameter Validation: Validate function parameters explicitly before processing. It's essential to check whether the inputs are valid, and if not, revert with a clear error message. For example, ensure that _dParams.amount is greater than or equal to _dParams.deposit.
2)The excessivelySafeCall function has the intention of adding safety checks is commendable, the implementation of excessivelySafeCall is not provided. The security of this mechanism depends on how it's implemented. Therefore, it's crucial to thoroughly review the implementation of excessivelySafeCall to ensure it doesn't introduce any security vulnerabilities.
Here are some security considerations to keep in mind:
Reentrancy: Ensure that the excessivelySafeCall function effectively guards against reentrancy attacks. It should correctly manage state changes, especially when interacting with external contracts.
Gas Limit: The usage of a gas limit (150 in this case) is meant to prevent excessive gas consumption. Make sure this limit is well-calibrated to prevent attacks while still allowing legitimate transactions to execute.

3)In the lzReceiveNonBlocking function there is some code duplication for executing different types of transactions. You might consider refactoring the common logic into separate internal functions to reduce duplication and improve readability.
function lzReceiveNonBlocking(
    address _endpoint,
    uint16 _srcChainId,
    bytes calldata _srcAddress,
    bytes calldata _payload
) public override requiresEndpoint(_endpoint, _srcChainId, _srcAddress) {
    // Parse deposit nonce
    uint32 nonce = parseDepositNonce(_payload);

    // Check if tx has already been executed
    require(executionState[_srcChainId][nonce] == STATUS_READY, "AlreadyExecutedTransaction");

    // Avoid stack too deep
    uint16 srcChainId = _srcChainId;

    if (_payload[0] == 0x00) {
        // DEPOSIT FLAG: 0 (System request / response)
        executeSystemRequest(nonce, _payload, srcChainId);
    } else if (_payload[0] == 0x01) {
        // DEPOSIT FLAG: 1 (Call without Deposit)
        executeNoDeposit(nonce, _payload, srcChainId);
    } else if (_payload[0] == 0x02) {
        // DEPOSIT FLAG: 2 (Call with Deposit)
        executeWithDeposit(nonce, _payload, srcChainId);
    } else if (_payload[0] == 0x03) {
        // DEPOSIT FLAG: 3 (Call with multiple asset Deposit)
        executeWithDepositMultiple(nonce, _payload, srcChainId);
    } else if (_payload[0] == 0x04) {
        // DEPOSIT FLAG: 4 (Call without Deposit + msg.sender)
        executeSignedNoDeposit(nonce, _payload, srcChainId);
    } else if (_payload[0] & 0x7F == 0x05) {
        // DEPOSIT FLAG: 5 (Call with Deposit + msg.sender)
        executeSignedWithDeposit(nonce, _payload, srcChainId);
    } else if (_payload[0] & 0x7F == 0x06) {
        // DEPOSIT FLAG: 6 (Call with multiple asset Deposit + msg.sender)
        executeSignedWithDepositMultiple(nonce, _payload, srcChainId);
    } else if (_payload[0] & 0x7F == 0x07) {
        // DEPOSIT FLAG: 7 (retrySettlement)
        executeRetrySettlement(nonce, _payload, srcChainId);
    } else if (_payload[0] == 0x08) {
        // DEPOSIT FLAG: 8 (retrieveDeposit)
        executeRetrieveDeposit(nonce, _payload, srcChainId);
    } else if (_payload[0] == 0x09) {
        // DEPOSIT FLAG: 9 (Fallback)
        reopenSettlementForRedemption(nonce, srcChainId);
    } else {
        revert("UnknownFlag");
    }

    emit LogExecute(nonce, srcChainId);
}

function parseDepositNonce(bytes calldata _payload) internal pure returns (uint32) {
    return uint32(bytes4(_payload[PARAMS_START:PARAMS_TKN_START]));
}

function executeSystemRequest(uint32 nonce, bytes calldata payload, uint16 srcChainId) internal {
    RootBridgeAgentExecutor(bridgeAgentExecutorAddress).executeSystemRequest(localRouterAddress, payload, srcChainId);
}

function executeNoDeposit(uint32 nonce, bytes calldata payload, uint16 srcChainId) internal {
    RootBridgeAgentExecutor(bridgeAgentExecutorAddress).executeNoDeposit(localRouterAddress, payload, srcChainId);
}

function executeWithDeposit(uint32 nonce, bytes calldata payload, uint16 srcChainId) internal {
    RootBridgeAgentExecutor(bridgeAgentExecutorAddress).executeWithDeposit(localRouterAddress, payload, srcChainId);
}

function executeWithDepositMultiple(uint32 nonce, bytes calldata payload, uint16 srcChainId) internal {
    RootBridgeAgentExecutor(bridgeAgentExecutorAddress).executeWithDepositMultiple(localRouterAddress, payload, srcChainId);
}

function executeSignedNoDeposit(uint32 nonce, bytes calldata payload, uint16 srcChainId) internal {
    address userAccount = getUserAccountFromPayload(payload);
    toggleVirtualAccountForExecution(userAccount);
    RootBridgeAgentExecutor(bridgeAgentExecutorAddress).executeSignedNoDeposit(userAccount, localRouterAddress, payload, srcChainId);
    toggleVirtualAccountForExecution(userAccount);
}

function executeSignedWithDeposit(uint32 nonce, bytes calldata payload, uint16 srcChainId) internal {
    address userAccount = getUserAccountFromPayload(payload);
    toggleVirtualAccountForExecution(userAccount);
    RootBridgeAgentExecutor(bridgeAgentExecutorAddress).executeSignedWithDeposit(userAccount, localRouterAddress, payload, srcChainId);
    toggleVirtualAccountForExecution(userAccount);
}

function executeSignedWithDepositMultiple(uint32 nonce, bytes calldata payload, uint16 srcChainId) internal {
    address userAccount = getUserAccountFromPayload(payload);
    toggleVirtualAccountForExecution(userAccount);
    RootBridgeAgentExecutor(bridgeAgentExecutorAddress).executeSignedWithDepositMultiple(userAccount, localRouterAddress, payload, srcChainId);
    toggleVirtualAccountForExecution(userAccount);
}

function executeRetrySettlement(uint32 nonce, bytes calldata payload, uint16 srcChainId) internal {
    (address owner, bytes memory params, GasParams memory gParams) = abi.decode(payload[PARAMS_START:], (address, bytes, GasParams));
    Settlement storage settlementReference = getSettlement[nonce];
    require(settlementReference.owner != address(0), "SettlementRetryUnavailable");
    if (owner != settlementReference.owner) {
        require(owner == address(IPort(localPortAddress).getUserAccount(settlementReference.owner)), "NotSettlementOwner");
    }
    settlementReference.status = STATUS_SUCCESS;
    performRetrySettlementCall(payload[0] == 0x87, settlementReference, params, nonce, srcChainId);
}

function executeRetrieveDeposit(uint32 nonce, bytes calldata payload, uint16 srcChainId) internal {
    uint32 existingState = executionState[srcChainId][nonce];
    if (existingState == STATUS_DONE) {
        revert("AlreadyExecutedTransaction");
    }
    if (existingState == STATUS_READY) {
        executionState[srcChainId][nonce] = STATUS_RETRIEVE;
    }
    address recipient = getUserAccountFromPayload(payload);
    performFallbackCall(recipient, nonce, srcChainId);
}

function reopenSettlementForRedemption(uint32 nonce, uint16 srcChainId) internal {
    getSettlement[nonce].status = STATUS_FAILED;
    emit LogFallback(nonce, srcChainId);
}

function getUserAccountFromPayload(bytes calldata payload) internal pure returns (address) {
    return address(uint160(bytes20(payload[PARAMS_START:PARAMS_START_SIGNED])));
}

function toggleVirtualAccountForExecution(address userAccount) internal {
    VirtualAccount account = IPort(localPortAddress).fetchVirtualAccount(userAccount);
    IPort(localPortAddress).toggleVirtualAccountApproved(account, localRouterAddress);
}
