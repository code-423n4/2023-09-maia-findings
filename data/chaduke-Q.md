QA1. setAddress() fails to delete the ``getGlobalTokenFromLocal[oldL][_srcChainId]``
        ``getLocalTokenFromGlobal[oldG][_srcChainId]``. As a result there might be two global tokens that are mapped to one local token. For example, suppose we had L1 <-> G1, and now we need to set L2 <-> G2, 
then we need to set L1 so that it points to zero; otherwise, both L1 and L2 will point to G2. 

Similar problem for ``setLocalAddress()``.

[https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L239-L256](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L239-L256)



Mitigation: 
 function setAddresses(
        address _globalAddress,
        address _localAddress,
        address _underlyingAddress,
        uint256 _srcChainId
    ) external override requiresCoreRootRouter {
        if (_globalAddress == address(0)) revert InvalidGlobalAddress();
        if (_localAddress == address(0)) revert InvalidLocalAddress();
        if (_underlyingAddress == address(0)) revert InvalidUnderlyingAddress();

+        address oldG = getGlobalTokenFromLocal[_localAddress][_srcChainId];
+        address oldL = getLocalTokenFromGlobal[_globalAddress][_srcChainId];
+        address oldU = getUnderlyingTokenFromLocal[_localAddress][_srcChainId];

+        delete isGlobalAddress[oldG];
+        delete getGlobalTokenFromLocal[oldL][_srcChainId];
+        delete getLocalTokenFromGlobal[oldG][_srcChainId];
+        delete getLocalTokenFromUnderlying[oldU][_srcChainId];
+        delete getUnderlyingTokenFromLocal[_localAddress][_srcChainId];

        isGlobalAddress[_globalAddress] = true;
        getGlobalTokenFromLocal[_localAddress][_srcChainId] = _globalAddress;
        getLocalTokenFromGlobal[_globalAddress][_srcChainId] = _localAddress;
        getLocalTokenFromUnderlying[_underlyingAddress][_srcChainId] = _localAddress;
        getUnderlyingTokenFromLocal[_localAddress][_srcChainId] = _underlyingAddress;

        emit LocalTokenAdded(_underlyingAddress, _localAddress, _globalAddress, _srcChainId);
    }

GA2: 