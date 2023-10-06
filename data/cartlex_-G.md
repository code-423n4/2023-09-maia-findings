[GAS-01] Cache arrays length to safe gas.

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L300-L302

```
function bridgeOutMultiple(
        address _depositor,
        address[] memory _localAddresses,
        address[] memory _underlyingAddresses,
        uint256[] memory _amounts,
        uint256[] memory _deposits
    ) external override lock requiresBridgeAgent {
        // Cache Length
        uint256 length = _localAddresses.length;

        // Sanity Check input arrays
        if (length > 255) revert InvalidInputArrays();
        if (length != _underlyingAddresses.length) revert InvalidInputArrays();
        if (_underlyingAddresses.length != _amounts.length) revert InvalidInputArrays(); // @audit-issue != lenght
        if (_amounts.length != _deposits.length) revert InvalidInputArrays(); // @audit-issue != lenght

        // Loop through token inputs and bridge out
        for (uint256 i = 0; i < length;) {
            _bridgeOut(_depositor, _localAddresses[i], _underlyingAddresses[i], _amounts[i], _deposits[i]);

            unchecked {
                i++;
            }
        }
    }
```

## Recommendation

```
function bridgeOutMultiple(
        address _depositor,
        address[] memory _localAddresses,
        address[] memory _underlyingAddresses,
        uint256[] memory _amounts,
        uint256[] memory _deposits
    ) external override lock requiresBridgeAgent {
        // Cache Length
        uint256 length = _localAddresses.length;

        // Sanity Check input arrays
        if (length > 255) revert InvalidInputArrays();
        if (length != _underlyingAddresses.length) revert InvalidInputArrays();
++      if (length != _amounts.length) revert InvalidInputArrays();
++      if (length != _deposits.length) revert InvalidInputArrays();

        // Loop through token inputs and bridge out
        for (uint256 i = 0; i < length;) {
            _bridgeOut(_depositor, _localAddresses[i], _underlyingAddresses[i], _amounts[i], _deposits[i]);

            unchecked {
                i++;
            }
        }
    }
```