## G-01 Use `!= 0` instead of `> 0` for uints
[BaseBranchRouter.sol#L160-L177](https://github.com/code-423n4/2023-09-maia/blob/c0dc3550e0754571b82d7bfd8f0282ac8fa5e42f/src/BaseBranchRouter.sol#L160-L177)
```diff
    function _transferAndApproveToken(address _hToken, address _token, uint256 _amount, uint256 _deposit) internal {
        // Cache local port address
        address _localPortAddress = localPortAddress;

        // Check if the local branch tokens are being spent
-       if (_amount - _deposit > 0) {
+       if (_amount - _deposit != 0) {
            unchecked {
                _hToken.safeTransferFrom(msg.sender, address(this), _amount - _deposit);
                ERC20(_hToken).approve(_localPortAddress, _amount - _deposit);
            }
        }

        // Check if the underlying tokens are being spent
-       if (_deposit > 0) {
+       if (_deposit != 0) {
            _token.safeTransferFrom(msg.sender, address(this), _deposit);
            ERC20(_token).approve(_localPortAddress, _deposit);
        }
    }
```