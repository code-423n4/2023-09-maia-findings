[L-01] - use ``abi.encodeCall`` instead of ``abi.encodeWithSelector`` for more type safety.

[L-02] - use ``create2`` or add ``msg.sender`` as the salt in factory contracts during deployment to achieve determinicity and avoid re-org attacks, mainly on Polygon