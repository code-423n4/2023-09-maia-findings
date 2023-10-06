## Summary

### Gas Optimization

no | Issue |Instances||
|-|:-|:-:|:-:|
| [G-01] | Using fixed bytes is cheaper than using string | 7 | - |
| [G-02] | Not using the named return variable when a function returns, wastes deployment gas | 2 | - |
| [G-03] | Can make the variable outside the loop to save gas | 3 | - |
| [G-04] | Use assembly to check for address(0) | 8 | - |
| [G-05] | Before transfer of  some functions, we should check some variables for possible gas save | 5 | - |
| [G-06] | State Variable can be packed into fewer storage slots | 1 | - |
| [G-07] | Using storage instead of memory for structs/arrays saves gas | 6 | - |
| [G-08] | With assembly, .call (bool success)  transfer can be done gas-optimized | 5 | - |
| [G-09] | Caching global variables is more expensive than using the actual variable (use msg.sender or block.timestamp instead of caching it) | 1 | - |
| [G-10] | Using ERC721A instead of ERC721 for more gas-efficient | 2 | - |
| [G-11] | Use hardcode address instead address(this) | 4 | - |
| [G-12] | Use assembly for math (add, sub, mul, div) | 2 | - |
| [G-13] | Don't apply the same value to state variables | 2 | - |
| [G-14] | Use assembly to validate msg.sender | 4 | - |
| [G-15] | Make 3 event parameters indexed when possible | 4 | - |

## Gas Optimizations  

## [G-01] Using fixed bytes is cheaper than using string

As a rule of thumb, use bytes for arbitrary-length raw byte data and string for arbitrary-length string (UTF-8) data.
If you can limit the length to a certain number of bytes, always use one of bytes1 to bytes32 because they are much cheaper.

```solidity
file: /src/factories/ERC20hTokenBranchFactory.sol

///@audit istead of string use bytes if possible on line 44, 45

26    string public chainName;

29    string public chainSymbol;

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenBranchFactory.sol#L26-L29


```solidity
file: /src/CoreRootRouter.sol

///@audit ' tring memory name, string memory symbol '
  
308            (address underlyingAddress, address localAddress, string memory name, string memory symbol, uint8 decimals)


455        string memory _name,

456        string memory _symbol,

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L308


```solidity
file: /src/RootPort.sol

441        string memory _wrappedGasTokenName,

442        string memory _wrappedGasTokenSymbol,

```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L441-L442


## [G-02] Not using the named return variable when a function returns, wastes deployment gas

When you execute a function that returns values in Solidity, the EVM still performs the necessary operations to execute and return those values. This includes the cost of allocating memory and packing the return values. If the returned values are not utilized, it can be seen as wasteful since you are incurring gas costs for operations that have no effect.

```solidity
file: /src/RootBridgeAgentExecutor.sol

/// @audit the address data  type is no need in this place, because its declared in function return. 

15        return address(new RootBridgeAgentExecutor(_rootBridgeAgent));

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L15


```solidity
file: /src/BranchBridgeAgentExecutor.sol

16    function deploy() external returns (address) {
17        return address(new BranchBridgeAgentExecutor());

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgentExecutor.sol#L16-L17


## [G-03] Can make the variable outside the loop to save gas

Consider making the stack variables before the loop which gonna save gas

```solidity
file: /src/RootBridgeAgentExecutor.sol

283            uint256 currentIterationOffset = PARAMS_START + i;

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L283


```solidity
file: /src/RootBridgeAgent.sol

320            address _hToken = settlement.hTokens[i];

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L320


```solidity
file: /src/VirtualAccount.sol

71            bool success;

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L71


## [G-04] Use assembly to check for address(0)

  Save 6 gas per instance.


```solidity
file: /src/RootBridgeAgent.sol

818        if (callee == address(0)) revert UnrecognizedBridgeAgent();

910        if (callee == address(0)) revert UnrecognizedBridgeAgent();

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L818


```solidity
file: /src/ArbitrumBranchPort.sol

63        if (_globalToken == address(0)) revert UnknownGlobalToken();

90        if (_underlyingAddress == address(0)) revert UnknownUnderlyingToken();

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol#L63


```solidity
file: /src/CoreRootRouter.sol

120        if (IBridgeAgent(_rootBridgeAgent).getBranchBridgeAgent(_dstChainId) != address(0)) revert InvalidChainId();

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L120


```solidity
file: /src/RootPort.sol

245        if (_globalAddress == address(0)) revert InvalidGlobalAddress();

246        if (_localAddress == address(0)) revert InvalidLocalAddress();

247        if (_underlyingAddress == address(0)) revert InvalidUnderlyingAddress();

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L245-L247


## [G-05]  Before transfer of  some functions, we should check some variables for possible gas save

Before transfer, we should check for amount being 0 so the function doesn't run when its not gonna do anything:

```solidity
file: /src/BranchPort.sol

233        _underlyingAddress.safeTransfer(_recipient, _deposit);

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L233


## [G-06] State Variable can be packed into fewer storage slots 

If variables occupying the same slot are both written the same function or by the constructor, avoids a separate Gsset (20000 gas). Reads of the variables are also cheaper.

```solidity
file: /src/RootBridgeAgent.sol

/// @audit the 40 and 75 number line should come consecutive, it will save gas , which can used on slot.

40    uint16 public immutable localChainId;

    /// @notice Bridge Agent Factory Address.
43    address public immutable factoryAddress;

    /// @notice Local Core Root Router Address
46    address public immutable localRouterAddress;

    /// @notice Local Port Address where funds deposited from this chain are stored.
49    address public immutable localPortAddress;

    /// @notice Local Layer Zero Endpoint Address for cross-chain communication.
52    address public immutable lzEndpointAddress;

    /// @notice Address of Root Bridge Agent Executor.
55    address public immutable bridgeAgentExecutorAddress;

    /*///////////////////////////////////////////////////////////////
                        BRANCH BRIDGE AGENTS STATE
    //////////////////////////////////////////////////////////////*/

    /// @notice Chain -> Branch Bridge Agent Address. For N chains, each Root Bridge Agent Address has M =< N Branch Bridge Agent Address.
62    mapping(uint256 chainId => address branchBridgeAgent) public getBranchBridgeAgent;

    /// @notice Message Path for each connected Branch Bridge Agent as bytes for Layzer Zero interaction = localAddress + destinationAddress abi.encodePacked()
65    mapping(uint256 chainId => bytes branchBridgeAgentPath) public getBranchBridgeAgentPath;

    /// @notice If true, bridge agent manager has allowed for a new given branch bridge agent to be synced/added.
68    mapping(uint256 chainId => bool allowed) public isBranchBridgeAgentAllowed;

    /*///////////////////////////////////////////////////////////////
                            SETTLEMENTS STATE
    //////////////////////////////////////////////////////////////*/

    /// @notice Deposit nonce used for identifying transaction.
75    uint32 public settlementNonce;

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L40-L76


## [G-07]  Using storage instead of memory for structs/arrays saves gas

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

```solidity
file: /src/MulticallRootRouter.sol

236            Call[] memory calls = abi.decode(_decode(encodedData[1:]), (Call[]));

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L236

```solidity
file: /src/RootBridgeAgentExecutor.sol

276        address[] memory hTokens = new address[](numOfAssets);

277        address[] memory tokens = new address[](numOfAssets);

278        uint256[] memory amounts = new uint256[](numOfAssets);

279        uint256[] memory deposits = new uint256[](numOfAssets);

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L276-L279


```solidity
file: /src/VirtualAccount.sol

66    function call(Call[] calldata calls) external override requiresApprovedCaller returns (bytes[] memory returnData) {

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L66


## [G-08] With assembly, .call (bool success)  transfer can be done gas-optimized

return data (bool success,) has to be stored due to EVM architecture, but in a usage like below, ‘out’ and ‘outsize’ values are given (0,0), this storage disappears and gas optimization is provided.
Did you know that the normal "(bool success, bytes memory returnData) = http://target.call()" automatically copies the return data to memory even if you omit the returnData variable? If a relayer executes transactions with such calls it can lead to a gas-griefing attack.

```solidity
file: /src/RootBridgeAgent.sol

424        (bool success,) = address(this).excessivelySafeCall(
            gasleft(),
            150,
            abi.encodeWithSelector(this.lzReceiveNonBlocking.selector, msg.sender, _srcChainId, _srcAddress, _payload)
428        );

754        (bool success,) = bridgeAgentExecutorAddress.call{value: address(this).balance}(_calldata);

779        (bool success,) = bridgeAgentExecutorAddress.call{value: address(this).balance}(_calldata);

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L424-L428

```solidity
file: /src/BranchBridgeAgent.sol

717        (bool success,) = bridgeAgentExecutorAddress.call{value: address(this).balance}(_calldata);

737        (bool success,) = bridgeAgentExecutorAddress.call{value: address(this).balance}(_calldata);

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L717


## [G-09]  Caching global variables is more expensive than using the actual variable (use msg.sender or block.timestamp instead of caching it)

```solidity
file: /src/RootBridgeAgent.sol

115        factoryAddress = msg.sender;

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L115


## [G-10] Using ERC721A instead of ERC721 for more gas-efficient 

ERC721A is a proposed extension to the ERC721 standard that aims to improve gas efficiency and reduce the cost of deploying and using non-fungible tokens (NFTs) on the Ethereum blockchain. 

Reference 1: https://nextrope.com/erc721-vs-erc721a-2/

Reference 2 :https://github.com/chiru-labs/ERC721A#about-the-project

 
```solidity
file: /src/VirtualAccount.sol

62        ERC721(_token).transferFrom(address(this), msg.sender, _tokenId);

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L62


```solidity
file: /src/interfaces/IVirtualAccount.sol

4     import {IERC721Receiver} from "@openzeppelin/contracts/token/ERC721/IERC721Receiver.sol";

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/IVirtualAccount.sol#L4


## [G-11] Use hardcode address instead address(this)
 
 Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.
References: https://book.getfoundry.sh/reference/forge-std/compute-create-address

an example :
```solidity
contract MyContract {
    address constant public CONTRACT_ADDRESS = 0x1234567890123456789012345678901234567890;
    
    function getContractAddress() public view returns (address) {
        return CONTRACT_ADDRESS;
    }
}
```
 
```solidity
file: /src/ArbitrumBranchPort.sol

66        _underlyingAddress.safeTransferFrom(_depositor, address(this), _deposit);

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol#L66


```solidity
file: /src/RootBridgeAgent.sol

940        ILayerZeroEndpoint(lzEndpointAddress).send{value: address(this).balance}(

1182        getBranchBridgeAgentPath[_branchChainId] = abi.encodePacked(_newBranchBridgeAgent, address(this));

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L940


```solidity
file: /src/BranchPort.sol
 
210        IPortStrategy(_strategy).withdraw(address(this), _token, amountToWithdraw);

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L210


## [G-12] Use assembly for math (add, sub, mul, div)

Use assembly for math instead of Solidity. You can check for overflow/underflow in assembly to ensure safety. If using Solidity versions < 0.8.0 and you are using Safemath, you can gain significant gas savings by using assembly to calculate values and checking for overflow/underflow.

```solidity
file: /src/BranchPort.sol

155        getStrategyTokenDebt[_token] = _strategyTokenDebt + _amount;

207        getStrategyTokenDebt[_token] = strategyTokenDebt - amountToWithdraw;

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L155


## [G-13] Don't apply the same value to state variables

Assigning the same value to a state variable repeatedly can cause unnecessary gas costs, because each assignment operation incurs a gas cost. Additionally, if the value being assigned is the same as the current value of the state variable, the assignment operation does not actually change anything, which can waste gas.

```solidity
file: /src/interfaces/BridgeAgentConstants.sol
 
13    uint8 internal constant STATUS_READY = 0;

15    uint8 internal constant STATUS_DONE = 1;

21    uint8 internal constant STATUS_FAILED = 1;

23    uint8 internal constant STATUS_SUCCESS = 0;

27    uint256 internal constant PARAMS_START = 1;

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/BridgeAgentConstants.sol#L13-L27


## [G-14] Use assembly to validate msg.sender

We can use assembly to efficiently validate msg.sender for the didPay and uniswapV3SwapCallback functions with the least amount of opcodes necessary. Additionally, we can use xor() instead of iszero(eq()), saving 3 gas. We can also potentially save gas on the unhappy path by using scratch space to store the error selector, potentially avoiding memory expansion costs.

```solidity
file: /src/ArbitrumBranchBridgeAgent.sol

126        if (msg.sender != address(this)) revert LayerZeroUnauthorizedEndpoint();

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchBridgeAgent.sol#L126


```solidity
file: /src/ArbitrumCoreBranchRouter.sol
 
65        IBridgeAgent(localBridgeAgentAddress).callOutSystem(payable(msg.sender), payload, GasParams(0, 0));

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumCoreBranchRouter.sol#L65


```solidity
file: /src/CoreRootRouter.sol

512        if (msg.sender != bridgeAgentExecutorAddress) revert UnrecognizedBridgeAgentExecutor();

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L512


```solidity
file: /src/BranchBridgeAgent.sol

441        if (deposit.owner != msg.sender) revert NotDepositOwner();

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L441


## [G-15] Make 3 event parameters indexed when possible

It's the most gas efficient to make up to 3 event parameters indexed. If there are less than 3 parameters, you need to make all parameters indexed.

```solidity
file: /src/interfaces/IRootBridgeAgent.sol

337    event LogExecute(uint256 indexed depositNonce, uint256 indexed srcChainId);

338    event LogFallback(uint256 indexed settlementNonce, uint256 indexed dstChainId);

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/IRootBridgeAgent.sol#L337-L338


```solidity
file: /src/interfaces/IBranchPort.sol

220    event DebtCreated(address indexed _strategy, address indexed _token, uint256 _amount);

221    event DebtRepaid(address indexed _strategy, address indexed _token, uint256 _amount);

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/IBranchPort.sol#L220-L221