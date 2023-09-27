# Amounts should be checked for `0` before calling a transfer

It is good practice always check for zero values before making any transfer in smart contracts because this can help save gas costs and avoid unwanted external calls.

Sending assets to an address with zero value will result in losing said assets.  

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L226

function withdraw(address _recipient, address _underlyingAddress, uint256 _deposit)
        public
        virtual
        override
        lock
        requiresBridgeAgent
    {
        _underlyingAddress.safeTransfer(_recipient, _deposit);
    }

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L160

function replenishReserves(address _token, uint256 _amount) external override lock {
        // Update Port Strategy Token Debt. Will underflow if not enough debt to repay.
        getPortStrategyTokenDebt[msg.sender][_token] -= _amount;

        // Update Strategy Token Global Debt. Will underflow if not enough debt to repay.
        getStrategyTokenDebt[_token] -= _amount;

        // Get current balance of _token
        uint256 currBalance = ERC20(_token).balanceOf(address(this));

        // Withdraw tokens from startegy
        IPortStrategy(msg.sender).withdraw(address(this), _token, _amount);

        // Check if _token balance has increased by _amount
        require(ERC20(_token).balanceOf(address(this)) - currBalance == _amount, "Port Strategy Withdraw Failed");

        // Emit DebtRepaid event
        emit DebtRepaid(msg.sender, _token, _amount);
    }

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L294

function bridgeToLocalBranchFromRoot(address _to, address _hToken, uint256 _amount)
        external
        override
        requiresLocalBranchPort
    {
        if (!isGlobalAddress[_hToken]) revert UnrecognizedToken();

        _hToken.safeTransfer(_to, _amount);
    }

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L304

function burn(address _from, address _hToken, uint256 _amount, uint256 _srcChainId)
        external
        override
        requiresBridgeAgent
    {
        if (!isGlobalAddress[_hToken]) revert UnrecognizedToken();
        ERC20hTokenRoot(_hToken).burn(_from, _amount, _srcChainId);
    }


