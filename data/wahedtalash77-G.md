|     |     |
| --- | --- |
| ID  | Description |
| \[G‑01\] | Save gas by preventing zero amount in mint() and burn() |
| \[G-02\] | Use hardcode address instead `address(this)` |
| \[G-03\] | Use assembly to validate `msg.sender` |
| \[G-04\] | Amounts should be checked for 0 before calling a transfer |
| \[G-05\] | Use assembly to write address storage values |
| \[G-06\] | Move `if()`checks to the top |
| \[G-07\] | State variables should be cached in stack variables rather than re-reading them from storage |
| \[G-08\] | Duplicated `require()/if()` checks should be refactored to a modifier or function |
| \[G-09\] | Use `selfbalance()` instead of `address(this).balance` |
| \[G-10\] | Use != 0 instead of > 0 |
| \[G-11\] | Use a temporary variable to cache repetitive calculation |
| \[G-12\] | uint8 is not always cheaper than uint256 |
| \[G-13\] | Usage of "UINTS", "INTS" smaller than 32 Bytes (256 bits) results in Increased Gas Consumption. |
| \[G-14\] | Caching global variables is expensive than using the variable itself |
| \[G-15\] | `abi.encode()` is less efficient than `abi.encodePacked()` |
| \[G-16\] | Use assembly in place of `abi.decode` to extract calldata values more efficiently |
| \[G-17\] | When possible, use assembly instead of `unchecked{}` |
| \[G-18\] | DEFAULT VALUE ASSIGNMENT TO VARIABLES CAN BE OMITTED TO SAVE GAS |
| \[G-19\] | Expensive operation inside a for loop |
| \[G-20\] | The result of a function call should be cached rather than re-calling the function |
| \[G-21\] | Make fewer external calls. |
| \[G-22\] | Rearranging order of state variable declarations to pack them will save storage slots and gas |
| \[G-23\] | Empty Blocks Should Be Removed Or Emit Something |
| \[G-24\] | Using storage instead of memory for structs/arrays saves gas |
| \[G-25\] | Multiple accesses of a mapping/array should use a storage pointer |
| \[G-26\] | `_hTokens.length` should be cached |
| \[G-27\] | use Mappings Instead of Arrays |
| \[G-28\] | Use bytes32 rather than string/bytes. |
| \[G-29\] | you can save storage by reordering Holding struct fields |
| \[G‑30\] | Use a more recent version of Solidity |
| \[ 0 \] | CONCLUSION |

## \[G‑01\] Save gas by preventing zero amount in mint() and burn()

```
29:	 function mint(address account, uint256 amount) external override onlyOwner returns (bool) {
   +       require(amount != 0, "invalid amount);
           _mint(account, amount);
          return true;
    }

    /// @inheritdoc IERC20hTokenBranch
    function burn(uint256 amount) public override onlyOwner {
    +       require(amount != 0, "invalid amount);
            _burn(msg.sender, amount);
    }
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/token/ERC20hTokenBranch.sol#L29C4-L37C6

```
function mint(address to, uint256 amount, uint256 chainId) external onlyOwner returns (bool) {
+       require(amount != 0, "invalid amount);
        getTokenBalance[chainId] += amount;  
        _mint(to, amount); 
        return true;
    }

    /**
     * @notice Burns new tokens and updates the total supply for the given chain.
     * @param from Address to burn tokens from.
     * @param amount Amount of tokens to burn.
     * @param chainId Chain Id of the chain to burn tokens to.
     */
    function burn(address from, uint256 amount, uint256 chainId) external onlyOwner {
  +       require(amount != 0, "invalid amount);
        getTokenBalance[chainId] -= amount; 
        _burn(from, amount); 
    }
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/token/ERC20hTokenRoot.sol#L57C4-L72C6

## \[G-02\] Use hardcode address instead `address(this)`

Instead of using `address(this)`, it is more gas-efficient to pre-calculate and use the hardcoded `address`. Foundry’s script.sol and solmate’s `LibRlp.sol` contracts can help achieve this.

References: <ins>https://book.getfoundry.sh/reference/forge-std/compute-create-address</ins>

<ins>https://twitter.com/transmissions11/status/1518507047943245824</ins>

```
126: if (msg.sender != address(this)) revert LayerZeroUnauthorizedEndpoint();
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchBridgeAgent.sol#L126

```
66:  _underlyingAddress.safeTransferFrom(_depositor, address(this), _deposit);
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol#L66

```
167:	_hToken.safeTransferFrom(msg.sender, address(this), _amount - _deposit);
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L167

```
174:	_token.safeTransferFrom(msg.sender, address(this), _deposit);
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L174

```
424:	if (getDeposit[_depositNonce].owner != msg.sender) revert NotDepositOwner();
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L424

```
441:	if (deposit.owner != msg.sender) revert NotDepositOwner();
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L441

```
938:	 if (msg.sender != address(this)) revert LayerZeroUnauthorizedEndpoint();
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L938

```
175:	uint256 currBalance = ERC20(_token).balanceOf(address(this));

178:	        IPortStrategy(msg.sender).withdraw(address(this), _token, _amount); 

181:	        require(ERC20(_token).balanceOf(address(this)) - currBalance == _amount, "Port Strategy Withdraw Failed"); 

210:	IPortStrategy(_strategy).withdraw(address(this), _token, amountToWithdraw); 

214:	            ERC20(_token).balanceOf(address(this)) - currBalance == amountToWithdraw, "Port Strategy Withdraw Failed"

526:	            _localAddress.safeTransferFrom(_depositor, address(this), _hTokenAmount);
```

[https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol\[\](https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L605)](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol%5B%5D%28https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L605%29 "https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol%5B%5D(https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L605)")

```
424:	(bool success,) = address(this).excessivelySafeCall(
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L424

```
1205:	if (msg.sender != address(this)) revert LayerZeroUnauthorizedEndpoint();
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1205

```
1233:	if (msg.sender != IPort(localPortAddress).getBridgeAgentManager(address(this))) {
```

[https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1233\[\](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L178)](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1233%5B%5D%28https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L178%29 "https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1233%5B%5D(https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L178)")

```
301:	 _hToken.safeTransferFrom(_from, address(this), _amount);
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L301

```
61:	    function withdrawERC721(address _token, uint256 _tokenId) external override requiresApprovedCaller {
        ERC721(_token).transferFrom(address(this), msg.sender, _tokenId);
    }
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L61C5-L63C6

## \[G-03\] Use assembly to validate `msg.sender`

We can use assembly to efficiently validate msg.sender for the didPay and uniswapV3SwapCallback functions with the least amount of opcodes necessary. Additionally, we can use xor() instead of iszero(eq()), saving 3 gas. We can also potentially save gas on the unhappy path by using scratch space to store the error selector, potentially avoiding memory expansion costs.

```
126: if (msg.sender != address(this)) revert LayerZeroUnauthorizedEndpoint();
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchBridgeAgent.sol#L126

```
207:	if (msg.sender != bridgeAgentExecutorAddress) revert UnrecognizedBridgeAgentExecutor();
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L207

```
354:	if (deposit.owner != msg.sender) revert NotDepositOwner();
```

[https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L354\[\](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L441)](https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L354%5B%5D%28https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L441%29 "https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L354%5B%5D(https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L441)")

```
938:	 if (msg.sender != address(this)) revert LayerZeroUnauthorizedEndpoint();
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L938

```
954:	if (msg.sender != bridgeAgentExecutorAddress) revert UnrecognizedBridgeAgentExecutor();
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L954

```
542:	 if (msg.sender != coreBranchRouterAddress) revert UnrecognizedCore();
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L531

```
112:	if (msg.sender != IPort(rootPortAddress).getBridgeAgentManager(_rootBridgeAgent)) {
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L112

```
278:	require(msg.sender == rootPortAddress, "Only root port can call");
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L278

```
512:	 if (msg.sender != bridgeAgentExecutorAddress) revert UnrecognizedBridgeAgentExecutor();
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L512

```
605:	if (msg.sender != bridgeAgentExecutorAddress) revert UnrecognizedBridgeAgentExecutor();
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L605

https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/BranchBridgeAgentFactory.sol#L121

```
File:	src/RootBridgeAgent.sol

247:	if (msg.sender != settlementReference.owner) {  
            if (msg.sender != address(IPort(localPortAddress).getUserAccount(settlementReference.owner))) {
            
285:	 if (msg.sender != settlementOwner) {
            if (msg.sender != address(IPort(localPortAddress).getUserAccount(settlementOwner))) {
            
311:	 if (msg.sender != settlementOwner) {
            if (msg.sender != address(IPort(localPortAddress).getUserAccount(settlementOwner))) {
            
430:	 if (!success) if (msg.sender == getBranchBridgeAgent[localChainId]) revert ExecutionFailure();
```

[https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L247\[\](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L424)](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L247%5B%5D%28https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L424%29 "https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L247%5B%5D(https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L424)")

```
1205:	if (msg.sender != address(this)) revert LayerZeroUnauthorizedEndpoint();


1221:	if (msg.sender != bridgeAgentExecutorAddress) revert UnrecognizedExecutor();

1227:	if (msg.sender != localPortAddress) revert UnrecognizedPort();

1233:	if (msg.sender != IPort(localPortAddress).getBridgeAgentManager(address(this))) {
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1205

```
570:	 if (msg.sender != coreRootRouterAddress) revert UnrecognizedCoreRootRouter();
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L570

```
576:	 if (msg.sender != localBranchPortAddress) revert UnrecognizedLocalBranchPort();
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L576

```
162:	if (msg.sender != userAddress)
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L162

https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenBranchFactory.sol#L113

https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenRootFactory.sol#L97C9-L98C49

https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ArbitrumBranchBridgeAgentFactory.sol#L85

## \[G-04\] Amounts should be checked for 0 before calling a transfer

It is considered a best practice to verify for zero values before executing any transfers within smart contract functions. This approach helps prevent unnecessary external calls and can effectively reduce gas costs.

This practice is particularly crucial when dealing with the transfer of tokens or ether, as sending these assets to an address with a zero value will result in the loss of those assets.

In Solidity, you can determine whether a value is zero by utilizing the == operator. Here's an example demonstrating how to check for a zero value before performing a transfer:

```
//example 
function transfer(address payable recipient, uint256 amount) public {
    require(amount > 0, "Amount must be greater than zero");
    recipient.transfer(amount);
}
```

```
94:  _underlyingAddress.safeTransfer(_recipient, _amount);
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol#L94

```
160:	_token.safeTransfer(msg.sender, _amount);
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L160

```
178:	IPortStrategy(msg.sender).withdraw(address(this), _token, _amount);
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L178

```
301:	 _hToken.safeTransferFrom(_from, address(this), _amount);
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L301

```
311:	        _hToken.safeTransfer(_to, _amount);
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L311

```
51:	function withdrawNative(uint256 _amount) external override requiresApprovedCaller {
        msg.sender.safeTransferETH(_amount);
    }

    /// @inheritdoc IVirtualAccount
    function withdrawERC20(address _token, uint256 _amount) external override requiresApprovedCaller {
        _token.safeTransfer(msg.sender, _amount);
    }
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L51C5-L58C6

## \[G-05\] Use assembly to write address storage values

By using assembly to write to address storage values, you can bypass some of these operations and lower the gas cost of writing to storage. Assembly code allows you to directly access the Ethereum Virtual Machine (EVM) and perform low-level operations that are not possible in Solidity.

example of using assembly to write to address storage values:

```
contract MyContract {
    address private myAddress;
    
    function setAddressUsingAssembly(address newAddress) public {
        assembly {
            sstore(0, newAddress)
        }
    }
}
```

```
File: src/ArbitrumBranchPort.sol

57: address _rootPortAddress = rootPortAddress;
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol#L57

```
94:	address _localPortAddress = localPortAddress;
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumCoreBranchRouter.sol#L94

## \[G-06\] Move `if()`checks to the top

The function would be more gas efficient if this check is moved to the top of the contract so that in scenarios where this check fails/true the function could revert without performing more gas consuming operations.

```
412:	if (payload.length == 0) revert DepositRetryUnavailableUseCallout();
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L412

## \[G-07\] State variables should be cached in stack variables rather than re-reading them from storage

The instances below point to the second+ access of a state variable within a function. Caching of a state variable replace each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

```
@audit localChainId is a state variable

83:	 if (!IRootPort(_rootPortAddress).isGlobalToken(_globalAddress, localChainId)) revert UnknownGlobalToken();
                                    
        // Get the underlying token address from the root port
        address _underlyingAddress =
            IRootPort(_rootPortAddress).getUnderlyingTokenFromLocal(_globalAddress, localChainId);
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol#L83C8-L87C99

## \[G-08\] Duplicated `require()/if()` checks should be refactored to a modifier or function

to reduce code duplication and improve readability. • Modifiers can be used to perform additional checks on the function inputs or state before it is executed. By defining a modifier to perform a specific check, we can reuse it across multiple functions that require the same check. • A function can also be used to perform a specific check and return a boolean value indicating whether the check has passed or failed. This can be useful when the check is more complex and cannot be performed easily in a modifier.

```
File:	src/RootBridgeAgent.sol

247:	if (msg.sender != settlementReference.owner) {  
            if (msg.sender != address(IPort(localPortAddress).getUserAccount(settlementReference.owner))) {
            
285:	 if (msg.sender != settlementOwner) {
            if (msg.sender != address(IPort(localPortAddress).getUserAccount(settlementOwner))) {
            
311:	 if (msg.sender != settlementOwner) {
            if (msg.sender != address(IPort(localPortAddress).getUserAccount(settlementOwner))) {
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L247

### \[GAS-09\] Use `selfbalance()` instead of `address(this).balance`

Use assembly when getting a contract's balance of ETH.

You can use `selfbalance()` instead of `address(this).balance` when getting your contract's balance of ETH to save gas. Additionally, you can use `balance(address)` instead of `address.balance()` when getting an external contract's balance of ETH.

```
92:	_recipient.safeTransferETH(address(this).balance);
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgentExecutor.sol#L92

```
126:	  _recipient.safeTransferETH(address(this).balance);
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgentExecutor.sol#L126

```
737:	(bool success,) = bridgeAgentExecutorAddress.call{value: address(this).balance}(_calldata);
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L737

```
787:	ILayerZeroEndpoint(lzEndpointAddress).send{value: address(this).balance}(
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L787

```
940:	 ILayerZeroEndpoint(lzEndpointAddress).send{value: address(this).balance}(
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L940

## \[G-10\] Use != 0 instead of > 0

Using `> 0` uses slightly more gas than using `!= 0`. Use `!= 0` when comparing `uint` variables to zero, which cannot hold values below zero.

```
if (_deposit > 0) {
            _underlyingAddress.safeTransferFrom(_depositor, address(this), _deposit);
        }

        //Burn hTokens if any are being used
        if (_amount - _deposit > 0) {
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol#L127C9-L132C38

```
165:	if (_amount - _deposit > 0) {
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L165

```
173:	if (_deposit > 0) {
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L173

```
906:	 if (_amount - _deposit > 0) {
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L906

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L912

```
266:	if (_deposits[i] > 0) {
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L266

```
259:	if (_amounts[i] - _deposits[i] > 0) {
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L259

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L531

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L531

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L363

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L370

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1142

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1149

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1156

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L284

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L290

## \[G-11\] Use a temporary variable to cache repetitive calculation

the result of the `_amount - _deposit` should be cached.

```
132:	if (_amount - _deposit > 0) {
            unchecked {
                IRootPort(rootPortAddress).bridgeToRootFromLocalBranch(_depositor, _localAddress, _amount - _deposit);
                }
          }
```

```
165:	if (_amount - _deposit > 0) {
            unchecked {
                _hToken.safeTransferFrom(msg.sender, address(this), _amount - _deposit);
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L165C9-L167C89

```
96:	if (_amount - _deposit > 0) {
            unchecked {
                IPort(localPortAddress).bridgeIn(_recipient, _hToken, _amount - _deposit);
            }
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L906

```
259:	if (_amounts[i] - _deposits[i] > 0) {
                unchecked {
                    _bridgeIn(_recipient, _localAddresses[i], _amounts[i] - _deposits[i]);
                }
            }
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L259

```
284:	 if (_amount - _deposit > 0) {
            unchecked {
                _hToken.safeTransfer(_recipient, _amount - _deposit);
            }
        }
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L284

## \[G-12\] uint8 is not always cheaper than uint256

The EVM only operates on 32 bytes/ 256 bits at a time. This means that if you use uint8, EVM has to first convert it uint256 to work on it and the conversion costs extra gas! You may wonder, What were the devs thinking? Why did they create smaller variables then? The answer lies in packing. In solidity, you can pack multiple small variables into one slot, but if you are defining a lone variable and can’t pack it, it’s optimal to use a uint256 rather than uint8.

```
501:	uint8 numOfAssets = uint8(bytes1(_sParams[0]));
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L501

```
64:	        uint8 decimals = ERC20(_underlyingAddress).decimals();
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L64

```
13:	 uint8 internal constant STATUS_READY = 0;

    uint8 internal constant STATUS_DONE = 1;

    uint8 internal constant STATUS_RETRIEVE = 2;

    // Settlement / Deposit Redeeem Status

    uint8 internal constant STATUS_FAILED = 1;

    uint8 internal constant STATUS_SUCCESS = 0;
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/BridgeAgentConstants.sol#L13C4-L23C48

```
96:	            ) = abi.decode(_params[1:], (address, string, string, uint8, address, GasParams));
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L96

```
243:	uint8(_dParams.hTokens.length),
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L243

```
359:	if (uint8(deposit.hTokens.length) == 1) {
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L359

```
273:	uint8 numOfAssets = uint8(bytes1(_dParams[0]));
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L273

```
96:	 function createToken(string memory _name, string memory _symbol, uint8 _decimals, bool _addPrefix)
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenBranchFactory.sol#L96

## \[G-13\] Usage of "UINTS", "INTS" smaller than 32 Bytes (256 bits) results in Increased Gas Consumption.

📌 Using Elements that are: Lower than 32 Bytes = Higher Gas Usage BECAUSE "EVM operates on 32 Bytes at a time & if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size"

```
uint16 public immutable localChainId;

    /// @notice Root Chain Id.
    uint16 public immutable rootChainId;
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/BranchBridgeAgentFactory.sol#L21C3-L24C41

```
uint16 public immutable rootChainId;
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/RootBridgeAgentFactory.sol#L13

```
uint24 public immutable localChainId;
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenBranchFactory.sol#L14

## \[G-14\] Caching global variables is expensive than using the variable itself

```
115:	factoryAddress = msg.sender;
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L115

## \[G-15\] `abi.encode()` is less efficient than `abi.encodePacked()`

In terms of efficiency, `abi.encodePacked()` is generally considered to be more gas-efficient than `abi.encode(),` because it skips the step of adding function signatures and other metadata to the encoded data. However, this comes at the cost.

```
53:	bytes memory params = abi.encode(
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumCoreBranchRouter.sol#L53

```
112:	bytes memory data = abi.encode(newBridgeAgent, _rootBridgeAgent);
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumCoreBranchRouter.sol#L112

```
48:	        bytes memory params = abi.encode(msg.sender, _globalAddress, _dstChainId, [_gParams[1], _gParams[2]]);
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L48

## \[G-16\] Use assembly in place of `abi.decode` to extract calldata values more efficiently

Instead of using abi.decode, we can use assembly to decode our desired calldata values directly. This will allow us to avoid decoding calldata values that we will not use.

```
File: src/ArbitrumCoreBranchRouter.sol

134:	) = abi.decode(_data[1:], (address, address, address, address, address, GasParams));

147:	(address bridgeAgentFactoryAddress) = abi.decode(_data[1:], (address));

153:	(address branchBridgeAgent) = abi.decode(_data[1:], (address));

158:	(address underlyingToken, uint256 minimumReservesRatio) = abi.decode(_data[1:], (address, uint256));

163:	(address portStrategy, address underlyingToken, uint256 dailyManagementLimit, bool isUpdateDailyLimit) =
                abi.decode(_data[1:], (address, address, uint256, bool));
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumCoreBranchRouter.sol#L134

[https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumCoreBranchRouter.sol#L163](https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumCoreBranchRouter.sol#L164)

```
96:	            ) = abi.decode(_params[1:], (address, string, string, uint8, address, GasParams));
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L96

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L116

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L122

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L128

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L135

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L178

## \[G-17\] When possible, use assembly instead of `unchecked{}`

You can also use unchecked{} for even more gas savings but this will not check to see if i overflows. For best gas savings, use inline assembly, however, this limits the functionality you can achieve.

```
195:	 unchecked {
                ++i;
            }
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L195C12-L197C14

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L450

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L308

*all loops are included.*

## \[G-18\] DEFAULT VALUE ASSIGNMENT TO VARIABLES CAN BE OMITTED TO SAVE GAS

Inside the `for` loop where the default value of `0` assigned to the `i` variable. Since `i` is by default initialzied to `0` it is not required explicitly assign the `0` value to `i` variable inside the `for` loops. This will save gas.

```
192: 	for (uint256 i = 0; i < _hTokens.length;) {
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L192

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L447

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L513

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L257

## \[G-19\] Expensive operation inside a for loop

```
518:	  _hTokens[i] = address(
                uint160(
                    bytes20(
                        bytes32(
                            _sParams[
                                PARAMS_TKN_START + (PARAMS_ENTRY_SIZE * i) + ADDRESS_END_OFFSET:
                                    PARAMS_TKN_START + (PARAMS_ENTRY_SIZE * currentIterationOffset)
                            ]
                        )
                    )
                )
            );

            _tokens[i] = address(
            ...
            
            
561:
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L513C8-L561C15

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L281

## \[G-20\] The result of a function call should be cached rather than re-calling the function

*External calls are expensive. External calls are expensive. Results of external function calls should be cached rather than call them multiple times. Consider caching the following*

```
570:	return SettlementMultipleParams(numOfAssets, _recipient, nonce, _hTokens, _tokens, _amounts, _deposits);
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L570

## \[G-21\] Make fewer external calls.

Every call to an external contract costs a decent amount of gas. For optimization of gas usage, It’s better to call one function and have it return all the data you need rather than calling a separate function for every piece of data. This might go against the best coding practices for other languages, but solidity is special.

## \[G-22\] Rearranging order of state variable declarations to pack them will save storage slots and gas

```Solidity
uint16 public immutable rootChainId;

    /// @notice Chain Id for Local Chain.
    uint16 public immutable localChainId;

    /// @notice Address for Bridge Agent who processes requests submitted for the Root Router Address
    ///         where cross-chain requests are executed in the Root Chain.
    address public immutable rootBridgeAgentAddress;

    /// @notice Layer Zero messaging layer path for Root Bridge Agent Address where cross-chain requests
    ///         are sent to the Root Chain Router.
    bytes private rootBridgeAgentPath;

    /// @notice Local Layerzero Endpoint Address where cross-chain requests are sent to the Root Chain Router.
    address public immutable lzEndpointAddress;

    /// @notice Address for Local Router used for custom actions for different hApps.
    address public immutable localRouterAddress;

    /// @notice Address for Local Port Address
    ///         where funds deposited from this chain are kept, managed and supplied to different Port Strategies.
    address public immutable localPortAddress;

    /// @notice Address for Bridge Agent Executor used for executing cross-chain requests.
    address public immutable bridgeAgentExecutorAddress;

    /*///////////////////////////////////////////////////////////////
                            DEPOSITS STATE
    //////////////////////////////////////////////////////////////*/

    /// @notice Deposit nonce used for identifying the transaction.
    uint32 public depositNonce;
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L53C4-L84C32

## \[G-23\] Empty Blocks Should Be Removed Or Emit Something

The code should be refactored such that they no longer exist, or the block should do something useful, such as emitting an event or reverting. If the contract is meant to be extended, the contract should be abstract and the function signatures be added without any default implementation. If the block is an empty if-statement block to avoid doing subsequent checks in the else-if/else conditions, the else-if/else conditions should be nested under the negation of the if-statement, because they involve different classes of checks, which may lead to the introduction of errors when the code is later modified (if(x){}else if(y){…}else{…} => if(!x){if(y){…}else{…}})

```
149:	receive() external payable {}
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L149

```
128:	receive() external payable {}
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L128

```
44:	receive() external payable {}
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L44

## \[G-24\] Using storage instead of memory for structs/arrays saves gas

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

```
507:	address[] memory _hTokens = new address[](numOfAssets);
        address[] memory _tokens = new address[](numOfAssets);
        uint256[] memory _amounts = new uint256[](numOfAssets);
        uint256[] memory _deposits = new uint256[](numOfAssets);
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L507C8-L510C65

```
827:	 address[] memory addressArray = new address[](1);
        uint256[] memory uintArray = new uint256[](1);
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L827

```
276:	 address[] memory hTokens = new address[](numOfAssets);
        address[] memory tokens = new address[](numOfAssets);
        uint256[] memory amounts = new uint256[](numOfAssets);
        uint256[] memory deposits = new uint256[](numOfAssets);
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L276

## \[G-25\] Multiple accesses of a mapping/array should use a storage pointer

Caching a mapping’s value in a storage pointer when the value is accessed multiple times saves ~40 gas per access due to not having to perform the same offset calculation every time. Help the Optimizer by saving a storage variable’s reference instead of repeatedly fetching it.

```
file:	src/RootBridgeAgent.sol

449: if (executionState[_srcChainId][nonce] != STATUS_READY) {

472:	 if (executionState[_srcChainId][nonce] != STATUS_READY) {

495:	 if (executionState[_srcChainId][nonce] != STATUS_READY) {

518:	 if (executionState[_srcChainId][nonce] != STATUS_READY) {

544:	 if (executionState[_srcChainId][nonce] != STATUS_READY) {

582:	 if (executionState[_srcChainId][nonce] != STATUS_READY) {

622:	 if (executionState[_srcChainId][nonce] != STATUS_READY) {

704:	if (executionState[_srcChainId][nonce] == STATUS_DONE) {

708:	if (executionState[_srcChainId][nonce] == STATUS_READY) {  @audit mapping
709:        executionState[_srcChainId][nonce] = STATUS_RETRIEVE;
                }
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L449

```
734:	executionState[_settlementNonce] = STATUS_DONE;

744:	executionState[_settlementNonce] = STATUS_RETRIEVE;
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L734

## \[G-26\] `_hTokens.length` should be cached

```
869:	  if (_hTokens.length > MAX_TOKENS_LENGTH) revert InvalidInput();
        if (_hTokens.length != _tokens.length) revert InvalidInput();
        if (_tokens.length != _amounts.length) revert InvalidInput();
        if (_amounts.length != _deposits.length) revert InvalidInput();
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L869

## \[G-27\]use Mappings Instead of Arrays

Mappings, are useful when you need to store and access data based on a key, rather than an index. Mappings have a lower gas cost for read and write operations, especially when the size of the mapping is large, since Solidity can perform these operations based on the key directly, without needing to iterate over the entire data structure.

```
File: src/BranchPort.sol

/// @notice Branch Routers deployed in branch chain.
35:    address[] public bridgeAgents;

45:	address[] public bridgeAgentFactories;

55:	 address[] public strategyTokens;

71:	 address[] public portStrategies;
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol

```
32:	address[] outputTokens;
    // Amounts of output hTokens to send.
    uint256[] amountsOut;
    // Amounts of output underlying tokens to send.
    uint256[] depositsOut;
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L32

```
File:	src/RootPort.sol

67:	address[] public bridgeAgents;

80:	address[] public bridgeAgentFactories;
```

[https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#](https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol "https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#")

```
23:	ERC20hTokenBranch[] public hTokens;
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenBranchFactory.sol#L23

## \[G-28\] Use bytes32 rather than string/bytes.

If you can fit your data in 32 bytes, then you should use bytes32 datatype rather than bytes or strings as it is much cheaper in solidity. Basically, Any fixed size variable in solidity is cheaper than variable size.

```
26:	 string public chainName;

    /// @notice Symbol of the chain for token symbol prefix.
29:    string public chainSymbol;
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenBranchFactory.sol#L26C5-L29C31

```
96:	            ) = abi.decode(_params[1:], (address, string, string, uint8, address, GasParams));
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L96

```
308:	(address underlyingAddress, address localAddress, string memory name, string memory symbol, uint8 decimals)
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L308

```
96:	 function createToken(string memory _name, string memory _symbol, uint8 _decimals, bool _addPrefix)
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenBranchFactory.sol#L96

## \[G-29\] you can save storage by reordering Holding struct fields

```
struct DepositMultipleParams {
    //Deposit Info
    uint8 numberOfAssets; //Number of assets to deposit.
    uint32 depositNonce; //Deposit nonce.
    address[] hTokens; //Input Local hTokens Address.
    address[] tokens; //Input Native / underlying Token Address.
    uint256[] amounts; //Amount of Local hTokens deposited for interaction.
    uint256[] deposits; //Amount of native tokens deposited for interaction.
}

struct DepositParams {
    //Deposit Info
    uint32 depositNonce; //Deposit nonce.
    address hToken; //Input Local hTokens Address.
    address token; //Input Native / underlying Token Address.
    uint256 amount; //Amount of Local hTokens deposited for interaction.
    uint256 deposit; //Amount of native tokens deposited for interaction.
}

struct Settlement {
    uint16 dstChainId; //Destination chain for interaction.
    uint80 status; //Status of the settlement
    address owner; //Owner of the settlement
    address recipient; //Recipient of the settlement.
    address[] hTokens; //Input Local hTokens Addresses.
    address[] tokens; //Input Native / underlying Token Addresses.
    uint256[] amounts; //Amount of Local hTokens deposited for interaction.
    uint256[] deposits; //Amount of native tokens deposited for interaction.
}

struct SettlementInput {
    address globalAddress; //Input Global hTokens Address.
    uint256 amount; //Amount of Local hTokens deposited for interaction.
    uint256 deposit; //Amount of native tokens deposited for interaction.
}
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/BridgeAgentStructs.sol#L38C1-L72C2

```
13:	struct PayableCall {
    address target;
    bytes callData;
    uint256 value;
}
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/IVirtualAccount.sol#L13

## \[G‑30\] Use a more recent version of Solidity

Use a Solidity version of at least 0.8.2 to get simple compiler automatic inlining.

Use a Solidity version of at least 0.8.3 to get better struct packing and cheaper multiple storage reads.

Use a Solidity version of at least 0.8.4 to get custom errors, which are cheaper at deployment than revert()/require() strings.

Use a Solidity version of at least 0.8.10 to have external calls skip contract existence checks if the external call has a return value.

```
2	pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchBridgeAgent.sol#L2

```
3	pragma solidity ^0.8.0;
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol#L3

> all contest

## CONCLUSION

As you move forward with implementing the suggested optimizations, we urge you to exercise caution and conduct meticulous testing. It is essential to verify that these changes do not introduce any new vulnerabilities and effectively deliver the desired performance improvements. Carefully review the code modifications and perform rigorous testing to validate the security and effectiveness of the refactored code.