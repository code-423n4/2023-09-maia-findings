# [G-01] Ordering of require statements can be changed in `RootPort.sol`

Performing checks inside `require` statements costs gas. This implies, that checks which are more likely to revert should be execute before checks which are less likely to revert.
If first `require` reverts, then the 2nd one won't be executed (thus there will be no gas paid for executing the 2nd `require`)

Both `initialize` and `initializeCore` can be called only once, thus checking that `initialize` and `initializeCore` had already been initialized - is the most likely scenario.
This implies, that `require(_setup, "Setup ended.");` and `require(_setupCore, "Core Setup ended.");`, should be the first `require` statement.

The code-base after the fix should look as below:

[File: RootPort.sol](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L129-L133)
```
 function initialize(address _bridgeAgentFactory, address _coreRootRouter) external onlyOwner {
        require(_setup, "Setup ended.");
        require(_bridgeAgentFactory != address(0), "Bridge Agent Factory cannot be 0 address.");
        require(_coreRootRouter != address(0), "Core Root Router cannot be 0 address.");
        _setup = false;
```

[File: RootPort.sol](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L1-L1)
```
  function initializeCore(
        address _coreRootBridgeAgent,
        address _coreLocalBranchBridgeAgent,
        address _localBranchPortAddress
    ) external onlyOwner {
        require(_setupCore, "Core Setup ended.");
        require(_coreRootBridgeAgent != address(0), "Core Root Bridge Agent cannot be 0 address.");
        require(_coreLocalBranchBridgeAgent != address(0), "Core Local Branch Bridge Agent cannot be 0 address.");
        require(_localBranchPortAddress != address(0), "Local Branch Port Address cannot be 0 address.");
        require(isBridgeAgent[_coreRootBridgeAgent], "Core Bridge Agent doesn't exist.");
        
```


# [G-02] Updating Port Strategy Token Debt in `BranchPort.sol` can be unchecked

[File: BranchPort.sol](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L202-L205)
```
        uint256 amountToWithdraw = portStrategyTokenDebt < reservesLacking ? portStrategyTokenDebt : reservesLacking;

        // Update Port Strategy Token Debt
        getPortStrategyTokenDebt[_strategy][_token] = portStrategyTokenDebt - amountToWithdraw;
```

Because of the condition: `portStrategyTokenDebt < reservesLacking ? portStrategyTokenDebt : reservesLacking`, the expression: `getPortStrategyTokenDebt[_strategy][_token] = portStrategyTokenDebt - amountToWithdraw;` will never underflow, thus it can be unchecked.


# [G-03] Length of arrays are calculated twice
As calculated length of the array is a costly operation - doing it twice is unnecessary. The more efficient approach would be to cache length of `_hTokens`, `_tokens`, `_amounts`, instead of calculating it twice:

[File: BranchBridgeAgent.sol](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L869-L872)
```
        if (_hTokens.length > MAX_TOKENS_LENGTH) revert InvalidInput();
        if (_hTokens.length != _tokens.length) revert InvalidInput();
        if (_tokens.length != _amounts.length) revert InvalidInput();
        if (_amounts.length != _deposits.length) revert InvalidInput();
```

The length of `_globalAddresses` is calculated 4 times. The length of `_amounts` - twice. The more efficient approach would be to cache the length, instead of calculating it multiple of times.

[File: RootBridgeAgent.sol](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L1057-1068)
```
if (_globalAddresses.length > MAX_TOKENS_LENGTH) revert InvalidInputParamsLength();

// Check if valid length
if (_globalAddresses.length != _amounts.length) revert InvalidInputParamsLength();
if (_amounts.length != _deposits.length) revert InvalidInputParamsLength();

//Update Settlement Nonce
settlementNonce = _settlementNonce + 1;

// Create Arrays
address[] memory hTokens = new address[](_globalAddresses.length);
address[] memory tokens = new address[](_globalAddresses.length);
```

# [G-04] Redundant check in `redeemDeposit`

In `redeemDeposit`, we firstly check if `deposit.owner` is not `address(0)` and then if `deposit.owner` is `msg.sender`.
The first check is unessesary. `msg.sender` cannot be `address(0)`, so  `deposit.owner == msg.sender` implies that `deposit.owner != address(0)`.

[File: BranchBridgeAgent.sol](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L440-L441)
```
        if (deposit.owner == address(0)) revert DepositRedeemUnavailable();
        if (deposit.owner != msg.sender) revert NotDepositOwner();
```
Line `if (deposit.owner == address(0)) revert DepositRedeemUnavailable()` can be removed.


# [G-05] Calculation for additional calldata in the payload in `executeWithDepositMultiple` can be done once

[File: RootBridgeAgentExecutor.sol](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgentExecutor.sol#L219-L229)
```
219:        if (
220:            _payload.length
221:                > PARAMS_END_SIGNED_OFFSET
222:                    + uint256(uint8(bytes1(_payload[PARAMS_START_SIGNED]))) * PARAMS_TKN_SET_SIZE_MULTIPLE
223:        ) {
224:            //Execute remote request
225:            IRouter(_router).executeSignedDepositMultiple{value: msg.value}(
226:                _payload[
227:                    PARAMS_END_SIGNED_OFFSET
228:                        + uint256(uint8(bytes1(_payload[PARAMS_START_SIGNED]))) * PARAMS_TKN_SET_SIZE_MULTIPLE:
229:               ],
```
`PARAMS_END_SIGNED_OFFSET + uint256(uint8(bytes1(_payload[PARAMS_START_SIGNED]))) * PARAMS_TKN_SET_SIZE_MULTIPLE` can be calculated once, the result cached in the variable and then used in line 221-222 and 227-228, instead of being calculated twice.

[File: RootBridgeAgentExecutor.sol](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgentExecutor.sol#L134-138)
```
134:        if (length > PARAMS_END_OFFSET + (numOfAssets * PARAMS_TKN_SET_SIZE_MULTIPLE)) {
135:            //Try to execute remote request
136:            IRouter(_router).executeDepositMultiple{value: msg.value}(
137:                _payload[PARAMS_END_OFFSET + uint256(numOfAssets) * PARAMS_TKN_SET_SIZE_MULTIPLE:], dParams, _srcChainId
138:            );
```

`PARAMS_END_OFFSET + (numOfAssets * PARAMS_TKN_SET_SIZE_MULTIPLE)` can be calculated once, the result cached in the variable and then used in line 134 and 137, instead of being calculated twice.

# [G-06] Calculations on constants can be pre-calculated

[File: RootBridgeAgent.sol](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L619)
```
nonce = uint32(bytes4(_payload[PARAMS_START_SIGNED + PARAMS_START:PARAMS_START_SIGNED + PARAMS_TKN_START]));
```

`PARAMS_START_SIGNED + PARAMS_START` and `PARAMS_START_SIGNED + PARAMS_TKN_START` can be calculated before the compilation time.


# [G-07] `_performRetrySettlementCall` in `RootBridgeAgent.sol` can be optimized for gas usage

There are multiple of issues which should be done to optimize this function:

* Getting destination Branch Bridge Agent and checking if valid destination has been returned should be moved on top

[File: RootBridgeAgent.sol](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L907-L910)
```
        address callee = getBranchBridgeAgent[_dstChainId];

        // Check if valid destination
        if (callee == address(0)) revert UnrecognizedBridgeAgent();
```

Those line should be moved on top of the function. If `callee` returns `address(0)`, we'll be saving gas, due to reverting immediately.

* `_hTokens.length` should be calculated once and cached

`_hTokens.length` is being checked in multiple of `if`'s conditions. Cache the `_hTokens.length` into a variable and use that variable, instead of calculating the length every time.

* `else if` (line 891) should be changed to `else`
Firstly we're comparing: `if (_hTokens.length == 0)`. If it is - function reverts with `SettlementRetryUnavailableUseCallout()`
Then, another `if` condition occurs. We're comparing: `if (_hTokens.length == 1)`.
If it's not, then, we're comparing: `else if (_hTokens.length > 1)`.

Please notice, that since `_hTokens.length` is not 0 (we've checked that at line 871) and if it's not 1 (we've checked that at line 877), then `_hTokens.length` has to be greater than 1.
This implies, that `else if (_hTokens.length > 1)` can be changed to a simple: `else`.


# [G-08] Parsing payload can be better optimized

To understand this issue more clearly, please take a look at function `execute` in `MulticallRootRouter.sol`.
Firstly, it extracts the first byte (parse funcId): `bytes1 funcId = encodedData[0];` and then - it compares that `funcId` in `if`-`else` branches.
This is more optimized version than accessing `encodedData[0]` ins every `if`-`else` statement.

To prove that, you can check these two example functions:

```
  function aaa(bytes calldata a) public {
        uint x;

        uint s = gasleft();
        if (a[0] == 0x00) x = 0;
        else if (a[0] == 0x01) x = 1;
        else if (a[0] == 0x02) x = 2;
        else if (a[0] == 0x03) x = 3;
        else x = 4;
        console.log(s - gasleft());
    }

      function bbb(bytes calldata a) public {
        uint x;

        uint s = gasleft();
        bytes1 z = a[0];
        
        if (z == 0x00) x = 0;
        else if (z == 0x01) x = 1;
        else if (z == 0x02) x = 2;
        else if (z == 0x03) x = 3;
        else x = 4;
        console.log(s - gasleft());
    }
```

Calling `bbb()` (237 gas usage) is more optimized than `aaa()` (382 gas usage)

However, in multiple of instances, first byte of payload is not cached in the variable, but it's being compared in every `if`-`else` branch:

* [File: ArbitrumCoreBranchRouter.sol](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/ArbitrumCoreBranchRouter.sol#L127-L172)
```
./src/ArbitrumCoreBranchRouter.sol:127:        if (_data[0] == 0x02) {
./src/ArbitrumCoreBranchRouter.sol:146:        } else if (_data[0] == 0x03) {
./src/ArbitrumCoreBranchRouter.sol:152:        } else if (_data[0] == 0x04) {
./src/ArbitrumCoreBranchRouter.sol:157:        } else if (_data[0] == 0x05) {
./src/ArbitrumCoreBranchRouter.sol:162:        } else if (_data[0] == 0x06) {

```

* [File: CoreBranchRouter.sol](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/CoreBranchRouter.sol#L86-L148)
```
./src/CoreBranchRouter.sol:88:        if (_params[0] == 0x01) {
./src/CoreBranchRouter.sol:100:        } else if (_params[0] == 0x02) {
./src/CoreBranchRouter.sol:115:        } else if (_params[0] == 0x03) {
./src/CoreBranchRouter.sol:121:        } else if (_params[0] == 0x04) {
./src/CoreBranchRouter.sol:127:        } else if (_params[0] == 0x05) {
./src/CoreBranchRouter.sol:133:        } else if (_params[0] == 0x06) {
./src/CoreBranchRouter.sol:140:        } else if (_params[0] == 0x07) {
```

* [File: RootBridgeAgent.sol](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L434-737)
```
./src/RootBridgeAgent.sol:444:        if (_payload[0] == 0x00) {
./src/RootBridgeAgent.sol:467:        } else if (_payload[0] == 0x01) {
./src/RootBridgeAgent.sol:490:        } else if (_payload[0] == 0x02) {
./src/RootBridgeAgent.sol:513:        } else if (_payload[0] == 0x03) {
./src/RootBridgeAgent.sol:539:        } else if (_payload[0] == 0x04) {
./src/RootBridgeAgent.sol:577:        } else if (_payload[0] & 0x7F == 0x05) {
./src/RootBridgeAgent.sol:600:                _payload[0] == 0x85,
./src/RootBridgeAgent.sol:617:        } else if (_payload[0] & 0x7F == 0x06) {
./src/RootBridgeAgent.sol:640:                _payload[0] == 0x86,
./src/RootBridgeAgent.sol:657:        } else if (_payload[0] & 0x7F == 0x07) {
./src/RootBridgeAgent.sol:684:                _payload[0] == 0x87,
./src/RootBridgeAgent.sol:699:        } else if (_payload[0] == 0x08) {
./src/RootBridgeAgent.sol:718:        } else if (_payload[0] == 0x09) {
```

Instead of extracting the first byte every time, do it once, assign the result to the variable, and then use that variable. See how it's done in the previously mentioned `MulticallRootRouter.sol`.

# [G-09] `retryDeposit` in `BranchBridgeAgent.sol` should not calculate `deposit.hTokens.length` twice

`uint8(deposit.hTokens.length)` is being calculated twice. Firstly, in the first condition, and then, when that condition fails - in the `else`-`if`:

[File: BranchBridgeAgent.sol](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L359)
```
 if (uint8(deposit.hTokens.length) == 1) {

 (...)

   } else if (uint8(deposit.hTokens.length) > 1) {
```

Instead of calculating the length of `deposit.hTokens` twice - cache the result in the variable and use that variable in the conditional branches.

# [G-10] Do-while loops are cheaper than for loops

Using do-while loops are better for saving gas than for-loops.

Multiple of for-loops can be rewritten to do-while loops.

```
./src/BaseBranchRouter.sol:        for (uint256 i = 0; i < _hTokens.length;) {
./src/BranchBridgeAgent.sol:        for (uint256 i = 0; i < deposit.tokens.length;) {
./src/BranchBridgeAgent.sol:        for (uint256 i = 0; i < numOfAssets;) {
./src/BranchPort.sol:        for (uint256 i = 0; i < length;) {
./src/BranchPort.sol:        for (uint256 i = 0; i < length;) {
./src/MulticallRootRouter.sol:            for (uint256 i = 0; i < outputParams.outputTokens.length;) {
./src/MulticallRootRouter.sol:            for (uint256 i = 0; i < outputParams.outputTokens.length;) {
./src/MulticallRootRouter.sol:            for (uint256 i = 0; i < outputParams.outputTokens.length;) {
./src/MulticallRootRouter.sol:        for (uint256 i = 0; i < outputTokens.length;) {
./src/RootBridgeAgent.sol:        for (uint256 i = 0; i < settlement.hTokens.length;) {
./src/RootBridgeAgent.sol:        for (uint256 i = 0; i < length;) {
./src/RootBridgeAgent.sol:        for (uint256 i = 0; i < hTokens.length;) {
./src/RootBridgeAgentExecutor.sol:        for (uint256 i = 0; i < uint256(uint8(numOfAssets));) {
./src/VirtualAccount.sol:        for (uint256 i = 0; i < length;) {
./src/VirtualAccount.sol:        for (uint256 i = 0; i < length;) {

```

# [G-11] uint256(uint8(numOfAssets)) should not be looked up in every loop of a `for`-loop 

[File: RootBridgeAgentExecutor.sol](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgentExecutor.sol#L281)
```
for (uint256 i = 0; i < uint256(uint8(numOfAssets));) {
```

Every iteration of loop calculates `uint256(uint8(numOfAssets))`. This operation can be cached in local variable instead of looking it up in every `for`-loop iteration:

```
uint256 numOfAssetLoop = uint256(uint8(numOfAssets));
for (uint256 i = 0; i < numOfAssetLoop;) {

```

# [G-12] Make 3 event parameters indexed when possible

The most gas efficient is to make up to 3 event parameters indexed. If there are less than 3 parameters, all of them should be indexed.

```
./src/interfaces/IBranchPort.sol:220:    event DebtCreated(address indexed _strategy, address indexed _token, uint256 _amount);
./src/interfaces/IBranchPort.sol:221:    event DebtRepaid(address indexed _strategy, address indexed _token, uint256 _amount);
./src/interfaces/IRootPort.sol:380:    event VirtualAccountCreated(address indexed user, address account);
```