# Summary



# Low Risk Issues    
no | Issue |Instances||
|-|:-|:-:|:-:|
| [L-01] |  NO check for contract accounts | 1 |

# Non-critical Issues
no | Issue |Instances||
|-|:-|:-:|:-:|
| [N-01] |  Misspelled word | 1 |
| [N-02] |  if statement syntax not correctly implemented | 2 |
| [N-03] |  no implementation of events | 5 |


## [L-01] No check for contract accounts
The function does not include verification of whether the supplied addresses are Externally Owned Accounts (EOAs) or contract addresses. If there is no check for contract accounts the function will accept EOA accounts and continue to work with them as if they were contract addresses which will lead to potential problems.
To resolve this issue, you should add checks to verify whether the supplied addresses are Externally Owned Accounts (EOAs) or contract addresses. This can be done by using the `extcodesize` opcode in Solidity. Here is an example of how you can implement this:

```solidity
function isContract(address _addr) private view returns (bool) {
    uint32 size;
    assembly {
        size := extcodesize(_addr)
    }
    return (size > 0);
}
```

Then, you can use this function in your `initialize` and `initializeCore` functions to check if the supplied addresses are contract addresses:

```solidity
function initialize(address _bridgeAgentFactory, address _coreRootRouter) external onlyOwner {
    require(_bridgeAgentFactory != address(0), "Bridge Agent Factory cannot be 0 address.");
    require(_coreRootRouter != address(0), "Core Root Router cannot be 0 address.");
    require(_setup, "Setup ended.");
    require(isContract(_bridgeAgentFactory), "Bridge Agent Factory must be a contract.");
    require(isContract(_coreRootRouter), "Core Root Router must be a contract.");
    _setup = false;

    isBridgeAgentFactory[_bridgeAgentFactory] = true;
    bridgeAgentFactories.push(_bridgeAgentFactory);

    coreRootRouterAddress = _coreRootRouter;
}
```

This will ensure that only contracts can be set as the Bridge Agent Factory and Core Root Router, which can prevent potential attacks or misuse of these roles.

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L129


## [N-01] Misspelled word
Inccorectly spelled word.

```solidity
 mapping(VirtualAccount acount => mapping(address router => bool allowed)) public isRouterApproved;
 ```

 https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L54


## [N-02] if statement syntax not correctly implemented
The "if" statement does not adhere to the correct syntax or layout. In Solidity, an if statement should be written as `if (condition) { code }`. 

In the functions, the if statement on line 74 is written as `if (isContract(_call.target)) (success, returnData[i]) = _call.target.call{value: val}(_call.callData);`. 

This is not the correct syntax for an if statement in Solidity. The code that should be executed if the condition is true should be enclosed in curly braces `{}`. 

Here is the corrected code:

```solidity
if (isContract(_call.target)) {
    (success, returnData[i]) = _call.target.call{value: val}(_call.callData);
}
```

This change will ensure that the if statement is correctly formatted and will execute as expected.

There are 2 instances of this
```solidity
 function call(Call[] calldata calls) external override requiresApprovedCaller returns (bytes[] memory returnData) {
        uint256 length = calls.length;
        returnData = new bytes[](length);

        for (uint256 i = 0; i < length;) {
            bool success;
            Call calldata _call = calls[i];

            if (isContract(_call.target)) (success, returnData[i]) = _call.target.call(_call.callData);

            if (!success) revert CallFailed();

            unchecked {
                ++i;
            }
        }
    }
```

```solidity
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
```

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/VirtualAccount.sol#L74C8-L74C8

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/VirtualAccount.sol#L101


## [N-03] no implementation of events
Contract Event Emission Deficiency: The contract under review does not emit any events. Although not a mandatory requirement, the emission of events is instrumental for off-chain tracking of contract activities.
To resolve this issue, you should consider adding event emissions at key points in your contract. Events allow light clients and web interfaces to react to on-chain transactions and activities. 

For example, you could add an event in the `withdrawNative`, `withdrawERC20`, and `withdrawERC721` functions to track when withdrawals occur. Here's how you could do it for `withdrawNative`:

```solidity
event WithdrawnNative(address indexed to, uint256 amount);

function withdrawNative(uint256 _amount) external override requiresApprovedCaller {
    msg.sender.safeTransferETH(_amount);
    emit WithdrawnNative(msg.sender, _amount);
}
```

You can do similar for `withdrawERC20` and `withdrawERC721`. Also consider adding events in the `call` and `payableCall` functions to track when these functions are called.

```solidity
 function withdrawNative(uint256 _amount) external override requiresApprovedCaller {
        msg.sender.safeTransferETH(_amount);
    }
```
```solidity
 function withdrawERC20(address _token, uint256 _amount) external override requiresApprovedCaller {
        _token.safeTransfer(msg.sender, _amount);
    }
```
```solidity
function withdrawERC721(address _token, uint256 _tokenId) external override requiresApprovedCaller {
        ERC721(_token).transferFrom(address(this), msg.sender, _tokenId);
    }
```
```solidity
 function call(Call[] calldata calls) external override requiresApprovedCaller returns (bytes[] memory returnData) {
        uint256 length = calls.length;
        returnData = new bytes[](length);

        for (uint256 i = 0; i < length;) {
            bool success;
            Call calldata _call = calls[i];

            if (isContract(_call.target)) (success, returnData[i]) = _call.target.call(_call.callData);

            if (!success) revert CallFailed();

            unchecked {
                ++i;
            }
        }
    }
```
```solidity
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
```

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/VirtualAccount.sol#L51
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/VirtualAccount.sol#L56
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/VirtualAccount.sol#L61
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/VirtualAccount.sol#L66
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/VirtualAccount.sol#L85