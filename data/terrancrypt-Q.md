# 1. `address(0)` and `amount > 0` is not checked in many functions in `RootPort.sol`

These are functions that use address as a parameter but do not check `address(0)`. Although this may not be financially harmful, it is still a top priority to check these parameters.

## Proof Of Concept

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L259-L270

No check for the `_globalAddress` parameter
```
    function setLocalAddress(address _globalAddress, address _localAddress, uint256 _srcChainId)
        external
        override
        requiresCoreRootRouter
    {
        if (_localAddress == address(0)) revert InvalidLocalAddress();

        getGlobalTokenFromLocal[_localAddress][_srcChainId] = _globalAddress;
        getLocalTokenFromGlobal[_globalAddress][_srcChainId] = _localAddress;

        emit GlobalTokenAdded(_localAddress, _globalAddress, _srcChainId);
    }
```

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L294-L302

Do not check the address in the `_from` parameter. In the previous function `bridgeToRoot`, I also mentioned this, but here we use `safeTransfer` of solady and because it is transferring from outside to this contract, we can check for exceptions in this case. So the previous function I evaluated the risk level as Medium.

```
    function bridgeToRootFromLocalBranch(address _from, address _hToken, uint256 _amount)
        external
        override
        requiresLocalBranchPort
    {
        if (!isGlobalAddress[_hToken]) revert UnrecognizedToken();

        _hToken.safeTransferFrom(_from, address(this), _amount);
    }
```

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L314-L322
```
    function burn(address _from, address _hToken, uint256 _amount, uint256 _srcChainId)
        external
        override
        requiresBridgeAgent
    {
        if (!isGlobalAddress[_hToken]) revert UnrecognizedToken();
        ERC20hTokenRoot(_hToken).burn(_from, _amount, _srcChainId);
    }
```

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L325-L333
No check for the `_from` and `_amount` parameters
```
    function burnFromLocalBranch(address _from, address _hToken, uint256 _amount)
        external
        override
        requiresLocalBranchPort
    {
        if (!isGlobalAddress[_hToken]) revert UnrecognizedToken();

        ERC20hTokenRoot(_hToken).burn(_from, _amount, localChainId);
    }
```

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L336-L343

No check for the `_to` and `_amount` parameters. 
```
    function mintToLocalBranch(address _to, address _hToken, uint256 _amount)
        external
        override
        requiresLocalBranchPort
    {
        if (!isGlobalAddress[_hToken]) revert UnrecognizedToken();
        if (!ERC20hTokenRoot(_hToken).mint(_to, _amount, localChainId)) revert UnableToMint();
    }
```

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L382-L390
No check for the `_manager` and `brigeAgent` parameters
```
    function addBridgeAgent(address _manager, address _bridgeAgent) external override requiresBridgeAgentFactory {
        if (isBridgeAgent[_bridgeAgent]) revert AlreadyAddedBridgeAgent();

        bridgeAgents.push(_bridgeAgent);
        getBridgeAgentManager[_bridgeAgent] = _manager;
        isBridgeAgent[_bridgeAgent] = true;

        emit BridgeAgentAdded(_bridgeAgent, _manager);
    }
```

## Recommendation
You should check `address(0)` before executing any function.  

My recommendation is to just add a simple modifier for checking `address(0)` and a modifier for checking `amount > 0`. And use these modifiers for all functions that need them.

# 2. Add `@inheritdoc` for function comments in `RootPort.sol`

## Proof Of Concept
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L304
```
    function bridgeToLocalBranchFromRoot(address _to, address _hToken, uint256 _amount)
        external
        override
        requiresLocalBranchPort
    {
    }
```

## Recommentdation

```
    /// @inheritdoc IRootPort
    function bridgeToLocalBranchFromRoot(address _to, address _hToken, uint256 _amount)
        external
        override
        requiresLocalBranchPort
    {
    }
```

# 3. Add an hyphens underscore before the name of the internal function

In your entire code, I understand that it is necessary to add a hyphen below before the name of the internal function. But here it is not there, you should add that in your code.

## Proof Of Concept

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L359C3-L359C3

```
    function addVirtualAccount(address _user) internal returns (VirtualAccount newAccount) {
    ...
    }
```

## Recommendedation

```
    function _addVirtualAccount(address _user) internal returns (VirtualAccount newAccount) {
    ...
    }
```

# 4. Exceptions should be checked in functions that are not internal functions, and internal functions should only be functions that execute within a contract.

## Proof Of Concept
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L349-L366
```
    /// @inheritdoc IRootPort
    function fetchVirtualAccount(address _user) external override returns (VirtualAccount account) {
        account = getUserAccount[_user];
        if (address(account) == address(0)) account = addVirtualAccount(_user);
    }

    /**
     * @notice Creates a new virtual account for a user.
     * @param _user address of the user to associate a virtual account with.
     */
    function addVirtualAccount(address _user) internal returns (VirtualAccount newAccount) {
        if (_user == address(0)) revert InvalidUserAddress();

        newAccount = new VirtualAccount{salt: keccak256(abi.encode(_user))}(_user, address(this));
        getUserAccount[_user] = newAccount;

        emit VirtualAccountCreated(_user, address(newAccount));
    }
```

## Recommendation
```
   /// @inheritdoc IRootPort
    function fetchVirtualAccount(
        address _user
    ) external override returns (VirtualAccount account) {
        if (_user == address(0)) revert InvalidUserAddress();

        account = getUserAccount[_user];
        if (address(account) == address(0)) {
            account = _addVirtualAccount(_user);
            emit VirtualAccountCreated(_user, address(newAccount));
        }
    }

    /**
     * @notice Creates a new virtual account for a user.
     * @param _user address of the user to associate a virtual account with.
     */
    function _addVirtualAccount(
        address _user
    ) internal returns (VirtualAccount newAccount) {
        newAccount = new VirtualAccount{salt: keccak256(abi.encode(_user))}(
            _user,
            address(this)
        );
        getUserAccount[_user] = newAccount;
    }
```

# 4. Code Duplication In `RootBridgeAgentExecutor.sol`

There is code repetition in parsing `_dParams`. This makes the code error-prone and difficult to maintain.

## Proof Of Concept

```
hTokens[i] = address(
    uint160(
        bytes20(
            bytes32(
                _dParams[
                    PARAMS_TKN_START + (PARAMS_ENTRY_SIZE * i) + ADDRESS_END_OFFSET:
                    PARAMS_TKN_START + (PARAMS_ENTRY_SIZE * currentIterationOffset)
                ]
            )
        )
    )
);

tokens[i] = address(
    uint160(
        bytes20(
            bytes32(
                _dParams[
                    PARAMS_TKN_START + PARAMS_ENTRY_SIZE * (i + numOfAssets) + ADDRESS_END_OFFSET:
                    PARAMS_TKN_START + PARAMS_ENTRY_SIZE * (currentIterationOffset + numOfAssets)
                ]
            )
        )
    )
);
```

## Recomendation

You should consider creating helper functions to avoid repeating code. For example, you could create a separate function to convert addresses from `_dParams` data to reduce repetition and make the code more readable and maintainable.

# 5. Overflow/Underflow in `_bridgeInMultiple` in `RootBridgeAgentExecutor.sol`

In the `_bridgeInMultiple` function, overflow or underflow can occur in some variables if the numOfAssets value is not handled properly.

## Proof Of Concept
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgentExecutor.sol#L273
Here, `numOfAssets` is read from the input `_dParams` and converted to type `uint8`. However, if the value in `_dParams` exceeds the limit of `uint8` (i.e. between 0 and 255), an overflow may occur.
```
    uint8 numOfAssets = uint8(bytes1(_dParams[0]));
```

## Recommendation
To fix this, you need to add more checking and processing before using the `numOfAssets` value.

For example:
```
if (numOfAssets > 0) revert();
```

When you use `numOfAssets` to iterate over an array or perform other calculations, make sure that you check for and handle any possible cases to avoid overflow or underflow.

