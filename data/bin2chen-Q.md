## Findings Summary

| Label | Description |
| - | - |
| [L-01](#l-01-payablecall-dont-support-target-is-eoas) | payableCall don't support target is EOAs|
| [L-02](#l-02-addbridgeagentfactory-bridgeagentfactories--may-be-duplicated) | addBridgeAgentFactory bridgeAgentFactories  may be duplicated|
| [L-03](#l-03-virtualaccount-miss-supportsinterface) | VirtualAccount miss supportsInterface|

## [L-01] payableCall don't support target is EOAs
The `payableCall()` can be used to execute the code and transfer eth to the `target`.
Currently, it determines if the `target` is a contract, but if it's not, it fails.
This doesn't support one case, transferring eth to EOAs.
Suggest removing the restriction to determine the contract

```diff
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
-           if (isContract(_call.target)) (success, returnData[i]) = _call.target.call{value: val}(_call.callData);
+           success, returnData[i] = _call.target.call{value: val}(_call.callData);

```

## [L-02] addBridgeAgentFactory bridgeAgentFactories  may be duplicated
`addBridgeAgentFactory()` executes `bridgeAgentFactories.push(_newBridgeAgentFactory);` every time
But disabling `BridgeAgentFactory` via `toggleBridgeAgentFactory()` does not remove the `factory`.
This way when `factory` if disabled by `enable` -> `disable` -> `enable`, bridgeAgentFactories will have duplicate `factory`s

Delete array elements after `disable`.  


## [L-03] VirtualAccount miss supportsInterface
Currently `VirtualAccount` supports ERC721, but does not implement `supportsInterface()`, in order to better integrate with third parties, it is recommended to implement this method.

```diff
contract VirtualAccount is IVirtualAccount, ERC1155Receiver {
...
+     function supportsInterface(bytes4 interfaceId) public view virtual override(ERC1155Receiver) returns (bool) {
+          return interfaceId == type(IERC721Receiver).interfaceId || super.supportsInterface(interfaceId);
+     }   
```