# 1. Instead of doing same calculation again and again inside a function, consider saving it into a memory variable and then use it
`_amount - _deposit` is used at many places in the codebase and everywhere it is calculated. Like in the following code. Same calculation will cost more gas, better to cache it into a variable first and then use it multiple times.
```solidity
        if (_amount - _deposit > 0) {
            unchecked {
                ///@audit-issue G Multiple same calculation will cost more gas, better to cache it into a variable first 
                _hToken.safeTransferFrom(msg.sender, address(this), _amount - _deposit);
                ERC20(_hToken).approve(_localPortAddress, _amount - _deposit);
            }
        }
```
#### Optimize the code like this:
```solidity
        uint256 calc = _amount - _deposit;

        // Check if the local branch tokens are being spent
        if (calc > 0) {
            unchecked {
                ///@audit-issue G Multiple same calculation will cost more gas, better to cache it into a variable first 
                _hToken.safeTransferFrom(msg.sender, address(this), calc);
                ERC20(_hToken).approve(_localPortAddress, calc);
            }
        }
```

There are more than 10 instances where this optimization needs to be implemented.