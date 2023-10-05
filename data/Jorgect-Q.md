# LOW REPORT 

## [L-01] Consider check for the lzReceive calls.

The check value for the call in lzReceive function is not checked 

```
 function lzReceive(uint16, bytes calldata _srcAddress, uint64, bytes calldata _payload) public override {
        address(this).excessivelySafeCall(
            gasleft(),
            150,
            abi.encodeWithSelector(this.lzReceiveNonBlocking.selector, msg.sender, _srcAddress, _payload)
        );
    }
```
https://github.com/code-423n4/2023-09-maia/blob/c0dc3550e0754571b82d7bfd8f0282ac8fa5e42f/src/BranchBridgeAgent.sol#L578C4-L584C6

Consider check for this value, and keep it. User can now se clearly his transaccion fail.

## [L-02] Users can open whatever virtual acconunt

The fetchVirtualAccount function in the virtualAccount.sol contract allow open virtual account of whatever addres.

```
 function fetchVirtualAccount(address _user) external override returns (VirtualAccount account) {
        account = getUserAccount[_user];
        if (address(account) == address(0)) account = addVirtualAccount(_user);
    }
```
https://github.com/code-423n4/2023-09-maia/blob/c0dc3550e0754571b82d7bfd8f0282ac8fa5e42f/src/RootPort.sol#L350C4-L353C6

Consider allow user to open just his only account.