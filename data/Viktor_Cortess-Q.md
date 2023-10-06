### [L] The depositToPort() and withdrawFromPort() functions in the ArbitrumBranchBridgeAgent contract are marked as payable; however, they should not handle native tokens.

    function depositToPort(address underlyingAddress, uint256 amount) external payable lock {
        IArbPort(localPortAddress).depositToPort(msg.sender, msg.sender, underlyingAddress, amount); //@LOW payable
    }

     function withdrawFromPort(address localAddress, uint256 amount) external payable lock {
        IArbPort(localPortAddress).withdrawFromPort(msg.sender, msg.sender, localAddress, amount);
    }

If user A accidentally sends native tokens with a call to these functions, the tokens will get stuck in the BranchBridgeAgent contract and may be used in subsequent operations by another user (user B), resulting in a loss of funds for user A.

    BranchBridgeAgent.sol:

    function _execute(uint256 _settlementNonce, bytes memory _calldata) private {
        //Update tx state as executed
        executionState[_settlementNonce] = STATUS_DONE;

        //Try to execute the remote request
        (bool success,) = bridgeAgentExecutorAddress.call{value: address(this).balance}(_calldata);

        //  No fallback is requested revert allowing for settlement retry.
        if (!success) revert ExecutionFailure();
    }

Consider removing the payable parameter from these functions to prevent the unintended handling of native tokens.

### [L] There is a potential underflow in the _bridgeOut function in the BranchPort contract.

    514: function _bridgeOut(
        address _depositor,
        address _localAddress,
        address _underlyingAddress,
        uint256 _amount,
        uint256 _deposit
    ) internal virtual {
        // Cache hToken amount out
        uint256 _hTokenAmount = _amount - _deposit;//@LOW underflow

        // Check if hTokens are being bridged out
        if (_hTokenAmount > 0) {
            _localAddress.safeTransferFrom(_depositor, address(this), _hTokenAmount);
            ERC20hTokenBranch(_localAddress).burn(_hTokenAmount);
        }

        // Check if underlying tokens are being bridged out
        if (_deposit > 0) {
            _underlyingAddress.safeTransferFrom(_depositor, address(this), _deposit);
        }
    }

The _amount parameter represents the total sum of tokens, while _deposit represents the amount of underlying tokens. It should hold that _hTokenAmount + _deposit = _amount. It is advisable to include a require statement to ensure that _amount - _deposit is greater than or equal to zero for such calculations.

Add a require statement to check if _amount - _deposit is greater than or equal to zero for correctness.

### [L] The variable _dstChainId is defined as a uint16, but in several mappings and functions, the input parameter _dstChainId is declared as uint256.

There is a huge list of functions that have _dstChainId as an input parameter, so to avoid spamming with copypasta I'll show an example where _dstchain is encoded as uint256 and decoded as uint16:

    CoreBranchRouter.sol 43:

    function addGlobalToken(address _globalAddress, uint256 _dstChainId, GasParams[3] calldata _gParams)
        external
        payable
    {
        // Encode Call Data
        bytes memory params = abi.encode(msg.sender, _globalAddress, _dstChainId, [_gParams[1], _gParams[2]]);

        // Pack FuncId
        bytes memory payload = abi.encodePacked(bytes1(0x01), params);

        // Send Cross-Chain request (System Response/Request)
        IBridgeAgent(localBridgeAgentAddress).callOut{value: msg.value}(payable(msg.sender), payload, _gParams[0]); 
    }

    CoreRootRouter.sol 332:

     function execute(bytes calldata _encodedData, uint16) external payable override requiresExecutor {
        // Parse funcId
        bytes1 funcId = _encodedData[0];

        /// FUNC ID: 1 (_addGlobalToken)
        if (funcId == 0x01) {
            (address refundee, address globalAddress, uint16 dstChainId, GasParams[2] memory gasParams) =
                abi.decode(_encodedData[1:], (address, address, uint16, GasParams[2]));

            _addGlobalToken(refundee, globalAddress, dstChainId, gasParams);

            /// Unrecognized Function Selector
        } else {
            revert UnrecognizedFunctionId();
        }
    }

If the value of _dstChainid is more than the maximum value of uint16 this will call a revert. I'm not sure that there will be chainIDs with such a big value, so it's just a theoretical issue.

Verify and correct the data type consistency of _dstChainId across all functions and mappings to match the input parameter types accurately.

### [L] In the payableCall() function, a virtual account user cannot directly use native tokens from their balance for a payable call.

    function payableCall(PayableCall[] calldata calls) public payable returns (bytes[] memory returnData) { 
        uint256 valAccumulator;
        uint256 length = calls.length;
        returnData = new bytes[](length);
        PayableCall calldata _call;
        for (uint256 i = 0; i < length;) {
            _call = calls[i];
            uint256 val = _call.value;
            // Humanity will be a Type V Kardashev Civilization before this overflows - andreas
            // ~ 10^25 Wei in existence << ~ 10^76 size uint fits in a uint256
            unchecked {
                valAccumulator += val;
            }

            bool success;

            if (isContract(_call.target)) (success, returnData[i]) = _call.target.call{value: val}(_call.callData);

            if (!success) revert CallFailed();

            unchecked {
                ++i;
            }
        }

        // Finally, make sure the msg.value = SUM(call[0...i].value)
        if (msg.value != valAccumulator) revert CallFailed(); 
    }

If a virtual account functions as a wallet where users can hold ETH, they cannot directly use their ETH balance for a payable call. Instead, they would need to withdraw ETH from the virtual account first and then make the payable call to ensure that msg.value matches valAccumulator.

### [L] Variable _dailyManagementLimit doesn't have upper and lower limits.

The _dailyManagementLimit variable stores the amount of a Strategy Token a given Port Strategy can manage. For added safety, it is advisable to set minimum and maximum values for such variables.

Implement both minimum and maximum limits for the _dailyManagementLimit variable to enhance safety and control.

    BranchPort.sol:

    403: function updatePortStrategy(address _portStrategy, address _token, uint256 _dailyManagementLimit)
        external
        override
        requiresCoreRouter
    {
        strategyDailyLimitAmount[_portStrategy][_token] = _dailyManagementLimit; //@LOW minimum

        emit PortStrategyUpdated(_portStrategy, _token, _dailyManagementLimit);
    }

    