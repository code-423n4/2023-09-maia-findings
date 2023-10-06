## [G-01] Move storage pointer to top of function to avoid offset calculation
We can avoid unnecessary offset calculations by moving the storage pointer to the top of the function.

```
  function _createDepositMultiple(
        uint32 _depositNonce,
        address payable _refundee,
        address[] memory _hTokens,
        address[] memory _tokens,
        uint256[] memory _amounts,
        uint256[] memory _deposits
    ) internal {
        // Validate Input
        if (_hTokens.length > MAX_TOKENS_LENGTH) revert InvalidInput();
        if (_hTokens.length != _tokens.length) revert InvalidInput();
        if (_tokens.length != _amounts.length) revert InvalidInput();
        if (_amounts.length != _deposits.length) revert InvalidInput();

        // Update Deposit Nonce
        depositNonce = _depositNonce + 1;

        // Deposit / Lock Tokens into Port
        IPort(localPortAddress).bridgeOutMultiple(msg.sender, _hTokens, _tokens, _amounts, _deposits);

        // Update State
        Deposit storage deposit = getDeposit[_depositNonce];
        deposit.owner = _refundee;
        deposit.hTokens = _hTokens;
        deposit.tokens = _tokens;
        deposit.amounts = _amounts;
        deposit.deposits = _deposits;
        deposit.status = STATUS_SUCCESS;
    }
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L860C3-L888C6

```
  function _createDeposit(
        uint32 _depositNonce,
        address payable _refundee,
        address _hToken,
        address _token,
        uint256 _amount,
        uint256 _deposit
    ) internal {
        // Update Deposit Nonce
        depositNonce = _depositNonce + 1;

        // Deposit / Lock Tokens into Port
        IPort(localPortAddress).bridgeOut(msg.sender, _hToken, _token, _amount, _deposit);

        // Cast to Dynamic
        address[] memory addressArray = new address[](1);
        uint256[] memory uintArray = new uint256[](1);

        // Save deposit to storage
        Deposit storage deposit = getDeposit[_depositNonce];
        deposit.owner = _refundee;

        addressArray[0] = _hToken;
        deposit.hTokens = addressArray;

        addressArray[0] = _token;
        deposit.tokens = addressArray;

        uintArray[0] = _amount;
        deposit.amounts = uintArray;
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L812C3-L841C37

## [G-02] Use hardcode address instead address(this)
Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.

```
 _underlyingAddress.safeTransferFrom(_depositor, address(this), _deposit);

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol#L66


``` 
_underlyingAddress.safeTransferFrom(_depositor, address(this), _deposit);
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol#L128C11-L128C86

```
   function lzReceive(uint16 _srcChainId, bytes calldata _srcAddress, uint64, bytes calldata _payload) public {
        (bool success,) = address(this).excessivelySafeCall(
            gasleft(),
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L423C2-L425C23

```         
  ILayerZeroEndpoint(lzEndpointAddress).send{value: address(this).balance}(
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L940C7-L940C82

```
      if (msg.sender != IPort(localPortAddress).getBridgeAgentManager(address(this))) {
            revert UnrecognizedBridgeAgentManager();
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1233C3-L1234C53

```
   if (_hTokenAmount > 0) {
            _localAddress.safeTransferFrom(_depositor, address(this), _hTokenAmount);
            ERC20hTokenBranch(_localAddress).burn(_hTokenAmount);
        }

        // Check if underlying tokens are being bridged out
        if (_deposit > 0) {
            _underlyingAddress.safeTransferFrom(_depositor, address(this), _deposit);
        }
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L525C6-L533C10

```
  uint256 currBalance = ERC20(_token).balanceOf(address(this));
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L436C7-L436C70

```
   // Withdraw tokens from startegy
        IPortStrategy(_strategy).withdraw(address(this), _token, amountToWithdraw);

        // Check if _token balance has increased by _amount
        require(
            ERC20(_token).balanceOf(address(this)) - currBalance == amountToWithdraw, "Port Strategy Withdraw Failed"
        );

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L209C6-L216C1

```
       uint256 currBalance = ERC20(_token).balanceOf(address(this));
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L193C2-L193C70

```
   // Get current balance of _token
        uint256 currBalance = ERC20(_token).balanceOf(address(this));

        // Withdraw tokens from startegy
        IPortStrategy(msg.sender).withdraw(address(this), _token, _amount);

        // Check if _token balance has increased by _amount
        require(ERC20(_token).balanceOf(address(this)) - currBalance == _amount, "Port Strategy Withdraw Failed");

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L174C6-L182C1

```
     if (msg.sender != address(this)) revert LayerZeroUnauthorizedEndpoint();
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L938C4-L938C81

```
     (bool success,) = bridgeAgentExecutorAddress.call{value: address(this).balance}(_calldata);
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L717C4-L717C100

```
  if (_amount - _deposit > 0) {
            unchecked {
                _hToken.safeTransferFrom(msg.sender, address(this), _amount - _deposit);
                ERC20(_hToken).approve(_localPortAddress, _amount - _deposit);
            }
        }

        // Check if the underlying tokens are being spent
        if (_deposit > 0) {
            _token.safeTransferFrom(msg.sender, address(this), _deposit);
            ERC20(_token).approve(_localPortAddress, _deposit);
        }
    }
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L165C7-L177C6

```
    _hToken.safeTransferFrom(_from, address(this), _amount);
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L301C5-L301C65

```
address(this).balance
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L695C17-L695C38

## [G‑03] Multiple accesses of a mapping/array should use a local variable cache
Caching a mapping’s value in a local storage or calldata variable when the value is accessed multiple times saves ~42 gas per access due to not having to perform the same offset calculation every time. Help the Optimizer by saving a storage variable’s reference instead of repeatedly fetching it.

```
                IVirtualAccount(userAccount).withdrawERC20(outputParams.outputTokens[i], outputParams.amountsOut[i]);
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L279C1-L279C118

```
  IVirtualAccount(userAccount).withdrawERC20(outputParams.outputTokens[i], outputParams.amountsOut[i]);
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L368C15-L368C118

```
   IVirtualAccount(userAccount).withdrawERC20(outputParams.outputTokens[i], outputParams.amountsOut[i]);
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L456C14-L456C118

## [G-04] Using storage instead of memory for structs/arrays saves gas
Using a memory pointer for a storage struct/array will effectively load all the fields of that data type from storage (SLOAD) into memory (MSTORE). Using a storage pointer will allow you to read specific fields from storage as you need them. If you are not going to use all of the fields of your data type then you should use a storage pointer so that you don’t incur extra Gcoldsload (2100 gas) for fields that you will never use.

```
Call[] memory calls = abi.decode(_decode(encodedData[1:]), (Call[]));
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L413C13-L413C82

```
 DepositParams memory dParams = DepositParams({
            depositNonce: uint32(bytes4(_payload[PARAMS_START:PARAMS_TKN_START])),
            hToken: address(uint160(bytes20(_payload[PARAMS_TKN_START:PARAMS_TKN_START_SIGNED]))),
            token: address(uint160(bytes20(_payload[PARAMS_TKN_START_SIGNED:45]))),
            amount: uint256(bytes32(_payload[45:77])),
            deposit: uint256(bytes32(_payload[77:PARAMS_TKN_SET_SIZE]))
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L88C8-L93C72

```
  DepositMultipleParams memory dParams = _bridgeInMultiple(
            _router,
            _payload[
                PARAMS_START:
                    PARAMS_END_OFFSET + uint256(uint8(bytes1(_payload[PARAMS_START]))) * PARAMS_TKN_SET_SIZE_MULTIPLE
            ],
            _srcChainId
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L121C7-L127C24

```
  bytes memory params = abi.encode(msg.sender, _globalAddress, _dstChainId, [_gParams[1], _gParams[2]]);
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L48C7-L48C111

## [G-05] Use do while loops instead of for loops
A do while loop will cost less gas since the condition is not being checked for the first iteration.

```
   for (uint256 i = 0; i < outputParams.outputTokens.length;) {
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L278C10-L278C73

```
       for (uint256 i = 0; i < outputParams.outputTokens.length;) {
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L367C6-L367C73

```
   for (uint256 i = 0; i < outputParams.outputTokens.length;) {
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L455C10-L455C73

```
     for (uint256 i = 0; i < outputTokens.length;) {
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L557C4-L557C56

```
     for (uint256 i = 0; i < settlement.hTokens.length;) {
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L318C4-L318C62

```
     for (uint256 i = 0; i < hTokens.length;) {
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1070C4-L1070C51

```
   for (uint256 i = 0; i < deposit.tokens.length;) {
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L447C6-L447C58

## [G-06] Arithmatic operations can be unchecked, since there is no underflow or overflow

```
 IRouter(_router).executeDepositMultiple{value: msg.value}(
                _payload[PARAMS_END_OFFSET + uint256(numOfAssets) * PARAMS_TKN_SET_SIZE_MULTIPLE:], dParams, _srcChainId
            );
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L136C12-L138C15

```
} else if (_payload[0] & 0x7F == 0x06) {
            // Parse deposit nonce
            nonce = uint32(bytes4(_payload[PARAMS_START_SIGNED + PARAMS_START:PARAMS_START_SIGNED + PARAMS_TKN_START]));
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L617C9-L619C121

```
 if (_payload.length > settlementEndOffset) {
            // Execute the remote request
            IRouter(_router).executeSettlementMultiple{value: msg.value}(
                _payload[PARAMS_END_SIGNED_OFFSET + assetsOffset:], sParams
            );
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgentExecutor.sol#L119C8-L123C15

## [G-07] += Costs More Gas Than =+ For State Variables

```
    getPortStrategyTokenDebt[msg.sender][_token] += _amount;
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L157C5-L157C65

```
  getTokenBalance[chainId] += amount;
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/token/ERC20hTokenRoot.sol#L58C7-L58C44

## [G-08] Public Functions To External
The following functions could be set external to save gas and improve code quality.
External call cost is less expensive than of public functions.
```
 function payableCall(PayableCall[] calldata calls) public payable returns (bytes[] memory returnData) {
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L85C4-L85C108

```
 function bridgeIn(address _recipient, DepositParams memory _dParams, uint256 _srcChainId)
        public
        override
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L351C4-L353C17

```
 function lzReceive(uint16 _srcChainId, bytes calldata _srcAddress, uint64, bytes calldata _payload) public {
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L423C4-L423C113

```
 function lzReceiveNonBlocking(
        address _endpoint,
        uint16 _srcChainId,
        bytes calldata _srcAddress,
        bytes calldata _payload
    ) public override requiresEndpoint(_endpoint, _srcChainId, _srcAddress) {
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L434C4-L439C78
```
  /// @inheritdoc ILayerZeroReceiver
    function lzReceive(uint16, bytes calldata _srcAddress, uint64, bytes calldata _payload) public override {
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L577C3-L578C110

```
  function lzReceiveNonBlocking(address _endpoint, bytes calldata _srcAddress, bytes calldata _payload)
        public
        override
        requiresEndpoint(_endpoint, _srcAddress)
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L587C3-L590C49

```
  function burn(uint256 amount) public override onlyOwner {
        _burn(msg.sender, amount);
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/token/ERC20hTokenBranch.sol#L35C3-L36C35


## [G-09] Redundant zero initialization

Solidity does not recognize null as a value, so uint variables are initialized to zero. Setting a uint variable to zero is redundant and can waste gas.

There are several places where an int is initialized to zero, which looks like:

```
  uint8 internal constant STATUS_READY = 0;
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/BridgeAgentConstants.sol#L13C3-L13C46

```
  uint8 internal constant STATUS_SUCCESS = 0;
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/BridgeAgentConstants.sol#L23C3-L23C48

## [G-10] Use the Latest Version of Solidity to Reduce gas.

```
pragma solidity >=0.5.0;
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/ILayerZeroEndpoint.sol#L3C1-L3C25

```
pragma solidity >=0.5.0;
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/ILayerZeroReceiver.sol#L3C1-L3C25