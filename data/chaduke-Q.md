QA1. ``addStrategyToken()`` fails to check whether the strategyToken to be added has already been added. 

[https://github.com/code-423n4/2023-09-maia/blob/c0dc3550e0754571b82d7bfd8f0282ac8fa5e42f/src/BranchPort.sol#L362-L372](https://github.com/code-423n4/2023-09-maia/blob/c0dc3550e0754571b82d7bfd8f0282ac8fa5e42f/src/BranchPort.sol#L362-L372)

Mitigation: we need to check whether the strategyToken to be added has already been added. 

```diff
 function addStrategyToken(address _token, uint256 _minimumReservesRatio) external override requiresCoreRouter {

+       if(isStrategyToken[_token])  revert AlreadyAddedStrategyToken();

        if (_minimumReservesRatio >= DIVISIONER || _minimumReservesRatio < MIN_RESERVE_RATIO) {
            revert InvalidMinimumReservesRatio();
        }

        strategyTokens.push(_token);
        getMinimumTokenReserveRatio[_token] = _minimumReservesRatio;
        isStrategyToken[_token] = true;

        emit StrategyTokenAdded(_token, _minimumReservesRatio);
    }
```

QA2. ``updatePortStrategy()`` fails to check whether the input PortStrategy has been added or not:

[https://github.com/code-423n4/2023-09-maia/blob/c0dc3550e0754571b82d7bfd8f0282ac8fa5e42f/src/BranchPort.sol#L403-L411](https://github.com/code-423n4/2023-09-maia/blob/c0dc3550e0754571b82d7bfd8f0282ac8fa5e42f/src/BranchPort.sol#L403-L411)

Mitigation: we need to check whether the input PortStrategy has been added or not. 

QA3. ``RootPort.addEcosystemToken()`` fails to ``getUnderlyingTokenFromLocal`` and ``getLocalTokenFromUnderlying ``. Both mappings need to be updated by this function.

[https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L483-L506](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L483-L506)

Mitigation:

```diff
 function addEcosystemToken(address _ecoTokenGlobalAddress) external override onlyOwner {
        // Check if token already added
        if (isGlobalAddress[_ecoTokenGlobalAddress]) revert AlreadyAddedEcosystemToken();

        // Check if token is already a underlying token in current chain
        if (getUnderlyingTokenFromLocal[_ecoTokenGlobalAddress][localChainId] != address(0)) {
            revert AlreadyAddedEcosystemToken();
        }

        // Check if token is already a local branch token in current chain
        if (getLocalTokenFromUnderlying[_ecoTokenGlobalAddress][localChainId] != address(0)) {
            revert AlreadyAddedEcosystemToken();
        }

        // Update State
        // 1. Add new global token to global address mapping
        isGlobalAddress[_ecoTokenGlobalAddress] = true;

+       getUnderlyingTokenFromLocal[_ecoTokenGlobalAddress][localChainId] = _ecoTokenGlobalAddress;
+       getLocalTokenFromUnderlying[_ecoTokenGlobalAddress][localChainId] = _ecoTokenGlobalAddress;


        // 2. Add new branch local token address to global token mapping
        getGlobalTokenFromLocal[_ecoTokenGlobalAddress][localChainId] = _ecoTokenGlobalAddress;
        // 3. Add new global token to branch local token address mapping
        getLocalTokenFromGlobal[_ecoTokenGlobalAddress][localChainId] = _ecoTokenGlobalAddress;

        emit EcosystemTokenAdded(_ecoTokenGlobalAddress);
    }
```

QA4. ``RootPort.setLocalAddress()`` fails to reset the old mapping for ``_globalAddress`` and ``_localAddress``. For example, suppose we had L1 <-> G1, but now we need to revise it to L2 <-> G1, then it is important to delete  getLocalTokenFromGlobal[L1][_srcChainId] so that L1 will not linked to G1.
In the same way, if we had L1 <-> G1, but we need to change it to L1 <-> G2, then it is important to delete getGlobalTokenFromLocal[G1][_srcChainId] so that G1 will not be linked to L1 anymore.  

Moreover, it fails to set the oldG to false and the new ``_globalAddress`` to true in ``isGlobalAddress``:

Similar unlinking needs to be done for function ``setAddresses()`` as well.

[https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L259C14-L270](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L259C14-L270)

Mitigation: we need to unlink the existing mapping first before setting the new mapping: 

```diff
function setLocalAddress(address _globalAddress, address _localAddress, uint256 _srcChainId)
        external
        override
        requiresCoreRootRouter
    {
        if (_localAddress == address(0)) revert InvalidLocalAddress();
        
+        address oldG = getGlobalTokenFromLocal[_localAddress][_srcChainId];
+        address oldL = getLocalTokenFromGlobal[_globalAddress][_srcChainId];
+        delete getGlobalTokenFromLocal[oldL][_srcChainId]; 
+        delete getLocalTokenFromGlobal[oldG][_srcChainId];

+        delete isGlobalAddress[olG];
+        isGlobalAddress[_globalAddress]  = true;

        getGlobalTokenFromLocal[_localAddress][_srcChainId] = _globalAddress;
        getLocalTokenFromGlobal[_globalAddress][_srcChainId] = _localAddress;

        emit GlobalTokenAdded(_localAddress, _globalAddress, _srcChainId);
    }
```