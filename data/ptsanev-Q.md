[L-01] - use ``abi.encodeCall`` instead of ``abi.encodeWithSelector`` for more type safety.

[L-02] - use ``create2`` or add ``msg.sender`` as the salt in factory contracts during deployment to achieve determinicity and avoid re-org attacks, mainly on Polygon

[L-03] - direct usage of ``callOutAndBridge`` and ``callOutAndBridgeMultiple`` from the branch agents would revert due to no allowance and eat user gas. Put some kind of access control so that these functions are only callable by routers, which do the necessary approvals beforehand.

[L-04] - in ``VirtualAccount`` calling ``payableCall`` with more native token will revert instead of simply refunding the excess, being unfavorable for the caller who will lose his gas, since the revert occurs after the loop

[L-05] - no 0 address checks for the ``_refundee`` addresses 