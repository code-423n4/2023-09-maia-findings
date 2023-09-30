### Gas Optimizations List


|no    |issue|number of issue|
|------|-----|---------------|
|[G-01]|Avoid contract existence checks by using low level calls|24|
|[G-02]|Can make the variable outside of the loop to save gas|1|
|[G-03]|Make 3 event parameters indexed when possible|5|
|[G-04]|>= costs less gas than >|1|
|[G-05]|State variables should be cached in stack variables rather than re-reading them from storage|1|
|[G-06]|abi.encode() is less efficient than abi.encodePacked()|2|
|[G-07]|Use hardcode address instead of address(this)|4|
|[G-08]|A modifier used only once and not being inherited should be inlined to save gas|3|
|[G-09]|Amounts should be checked for 0 before calling a transfer|3|
|[G-10]|Use assembly to hash instead of solidity|1|
|[G-11]|Loop best practice to save gas|3|
|[G-12]|Gas savings can be achieved by changing the model for assigning value to the structure|5|
|[G-13]|Use assembly for math (add, sub, mul, div)|3|
|[G-14]|Access mappings directly rather than using accessor functions|1|
|[G-15]|Use mappings instead of arrays|6|
|[G-16]|Use Short-Circuiting rules to your advantage|1|
|[G-17]|Use ERC721A instead ERC721|-|
|[G-18]|Add unchecked {} for subtractions where the operands cannot underflow because of a previous require() or if-statement|1|



Total 11 issues

## [G‑01] Avoid contract existence checks by using low level calls

Prior to 0.8.10, the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.

```solidity

70        IArbPort(localPortAddress).depositToPort(msg.sender, msg.sender, underlyingAddress, amount);

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchBridgeAgent.sol#L70

```solidity

80        IArbPort(localPortAddress).withdrawFromPort(msg.sender, msg.sender, localAddress, amount);

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchBridgeAgent.sol#L80

```solidity

105        IRootBridgeAgent(_rootBridgeAgentAddress).lzReceive(rootChainId, "", 0, _calldata);

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchBridgeAgent.sol#L105

```solidity

114        IRootBridgeAgent(rootBridgeAgentAddress).lzReceive(
115            rootChainId, "", 0, abi.encodePacked(bytes1(0x09), _settlementNonce)
116        );

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchBridgeAgent.sol#L114C1-L116C11

```solidity

60        address _globalToken = IRootPort(_rootPortAddress).getLocalTokenFromUnderlying(_underlyingAddress, localChainId);

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol#L60

```solidity

66        _underlyingAddress.safeTransferFrom(_depositor, address(this), _deposit);

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol#L66
```solidity

69        IRootPort(_rootPortAddress).mintToLocalBranch(_recipient, _globalToken, _deposit);

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol#L69

```solidity

83        if (!IRootPort(_rootPortAddress).isGlobalToken(_globalAddress, localChainId)) revert UnknownGlobalToken();

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol#L83

```solidity

86        address _underlyingAddress =
87            IRootPort(_rootPortAddress).getUnderlyingTokenFromLocal(_globalAddress, localChainId);

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol#L86-L87

```solidity

92        IRootPort(_rootPortAddress).burnFromLocalBranch(_depositor, _globalAddress, _amount);

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol#L92

```solidity

94        _underlyingAddress.safeTransfer(_recipient, _amount);

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol#L94

```solidity

108        IRootPort(rootPortAddress).bridgeToLocalBranchFromRoot(_recipient, _localAddress, _amount);

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol#L108

```solidity

128            _underlyingAddress.safeTransferFrom(_depositor, address(this), _deposit);

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol#L128

```solidity

134            IRootPort(rootPortAddress).bridgeToRootFromLocalBranch(_depositor, _localAddress, _amount - _deposit);

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol#L134

```solidity

65        IBridgeAgent(localBridgeAgentAddress).callOutSystem(payable(msg.sender), payload, GasParams(0, 0));

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumCoreBranchRouter.sol#L65

```solidity

97        if (!IPort(_localPortAddress).isBridgeAgentFactory(_branchBridgeAgentFactory)) {

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumCoreBranchRouter.sol#L97

```solidity

102        address newBridgeAgent = IBridgeAgentFactory(_branchBridgeAgentFactory).createBridgeAgent(
103            _newBranchRouter, _rootBridgeAgent, _rootBridgeAgentFactory
104        );

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumCoreBranchRouter.sol#L102-L104

```solidity

107        if (!IPort(_localPortAddress).isBridgeAgent(newBridgeAgent)) {

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumCoreBranchRouter.sol#L107

```solidity

118        IBridgeAgent(localBridgeAgentAddress).callOutSystem(payable(_refundee), payload, _gParams);

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumCoreBranchRouter.sol#L118

```solidity

113        bridgeAgentExecutorAddress = IBridgeAgent(_bridgeAgentAddress).bridgeAgentExecutorAddress();

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L113

```solidity

251         IVirtualAccount(userAccount).withdrawERC20(outputParams.outputToken, outputParams.amountOut);

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L251

```solidity

255                IVirtualAccount(userAccount).userAddress(),

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L255

```solidity

279                 IVirtualAccount(userAccount).withdrawERC20(outputParams.outputTokens[i], outputParams.amountsOut[i]);

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L279

```solidity

340                 IVirtualAccount(userAccount).withdrawERC20(outputParams.outputToken, outputParams.amountOut);

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L340



## [G‑02] Can make the variable outside of the loop to save gas

When you declare a variable inside a loop, Solidity creates a new instance of the variable for each iteration of the loop. This can lead to unnecessary gas costs; especially if the loop is executed frequently or iterates over a large number of elements.

By declaring the variable outside of the loop, you can avoid the creation of multiple instances of the variable and reduce the gas cost of your contract.

Here’s an example:

```solidity
contract MyContract {
    function sum(uint256[] memory values) public pure returns (uint256) {
        uint256 total = 0;
        
        for (uint256 i = 0; i < values.length; i++) {
            total += values[i];
        }
        
        return total;
    }
}


```
```solidity

320            address _hToken = settlement.hTokens[i];

```
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L320

## [G‑03] Make 3 event parameters indexed when possible

It’s the most gas efficient to make up to 3 event parameters indexed. If there are less than 3 parameters, you need to make all parameters indexed.

```solidity
223    event StrategyTokenAdded(address indexed _token, uint256 indexed _minimumReservesRatio);
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/IBranchPort.sol#L223

```solidity
229    event PortStrategyToggled(address indexed _portStrategy, address indexed _token);
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/IBranchPort.sol#L229

```solidity
239    event CoreBranchSet(address indexed _coreBranchRouter, address indexed _coreBranchBridgeAgent);
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/IBranchPort.sol#L239

```solidity
337    event LogExecute(uint256 indexed depositNonce, uint256 indexed srcChainId);
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/IRootBridgeAgent.sol#L337

```solidity
338    event LogFallback(uint256 indexed settlementNonce, uint256 indexed dstChainId);
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/IRootBridgeAgent.sol#L338

## [G‑04] >= costs less gas than >

The compiler uses opcodes for solidity code that uses >, but only requires for >=, which saves 3 gas.



```solidity

396        if (length > MAX_TOKENS_LENGTH)
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L396


## [G‑05] State variables should be cached in stack variables rather than re-reading them from storage
The instances below point to the second+ access of a state variable within a function. Caching of a state variable replaces each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include: having local memory caches of state variable structs or having local caches of state variable contracts/addresses.

```solidity

466        address newToken = address(IFactory(hTokenFactoryAddress).createToken(_name, _symbol, _decimals));
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreRootRouter.sol#L466

## [G‑06] abi.encode() is less efficient than abi.encodePacked()
In terms of efficiency, abi.encodePacked() is generally considered to be more gas-efficient than abi.encode() because it skips the step of adding function signatures and other metadata to the encoded data. However, this comes at the cost of reduced safety, as abi.encodePacked() does not perform any type of checking or padding of data.

```solidity

72      bytes memory params = abi.encode(_underlyingAddress, newToken, newToken.name(), newToken.symbol(), decimals);
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L72

```solidity

48     bytes memory params = abi.encode(msg.sender, _globalAddress, _dstChainId, [_gParams[1], _gParams[2]]);
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L48
        
## [G‑07] Use hardcode address instead of address(this)
It can be more gas-efficient to use a hardcoded address instead of the address(this) expression, especially if you need to use the same address multiple times in your contract.

The reason for this, is that using address(this) requires an additional EXTCODESIZE operation to retrieve the contract’s address from its bytecode, which can increase the gas cost of your contract. By pre-calculating and using a hardcoded address, you can avoid this additional operation and reduce the overall gas cost of your contract.

Here’s an example of how you can use a hardcoded address instead of address(this):


```solidity
contract MyContract {
    address public myAddress = 0x1234567890123456789012345678901234567890;
    
    function doSomething() public {
        // Use myAddress instead of address(this)
        require(msg.sender == myAddress, "Caller is not authorized");
        
        // Do something
    }
}
```
In the above example, we have a contract (MyContract) with a public address variable myAddress. Instead of using address(this) to retrieve the contract’s address, we have pre-calculated and hardcoded the address in the variable. This can help to reduce the gas cost of our contract and make our code more efficient.

```solidity

126        if (msg.sender != address(this)) revert LayerZeroUnauthorizedEndpoint();
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchBridgeAgent.sol#L126

```solidity

66        _underlyingAddress.safeTransferFrom(_depositor, address(this), _deposit);
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol#L66

```solidity

128            _underlyingAddress.safeTransferFrom(_depositor, address(this), _deposit);
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol#L128

```solidity

938        if (msg.sender != address(this)) revert LayerZeroUnauthorizedEndpoint();
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L938

## [G‑08] A modifier used only once and not being inherited should be inlined to save gas
When a modifier is used only once and is not inherited by any other contracts, inlining it can reduce gas costs. Inlining means that the modifier’s code is directly inserted into the function where it is applied, rather than creating a separate function call for the modifier.

By inlining the modifier, you avoid the overhead of the additional function call, which results in lower gas consumption. This optimization is especially useful when the modifier’s code is short or simple.

```solidity

947    modifier requiresRouter() {
        if (msg.sender != localRouterAddress) revert UnrecognizedRouter();
        _;
    }
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L947

```solidity

553    modifier requiresBridgeAgentFactory() {
        if (!isBridgeAgentFactory[msg.sender]) revert UnrecognizedBridgeAgentFactory();
        _;
    }
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L553C1-L556C6

```solidity
559    modifier requiresPortStrategy(address _token) {
        if (!isStrategyToken[_token]) revert UnrecognizedStrategyToken();
        if (!isPortStrategy[msg.sender][_token]) revert UnrecognizedPortStrategy();
        _;
    }
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L559

## [G‑09] Amounts should be checked for 0 before calling a transfer

It is generally a good practice to check for zero values before making any transfers in smart contract functions. This can help to avoid unnecessary external calls and save gas costs.

Checking for zero values is especially important when transferring tokens or ether, as sending these assets to an address with a zero value will result in the loss of those assets.

In Solidity, you can check whether a value is zero by using the == operator. Here’s an example of how you can check for a zero value before making a transfer:

```solidity
function transfer(address payable recipient, uint256 amount) public {
    require(amount > 0, "Amount must be greater than zero");
    recipient.transfer(amount);
}

In the above example, we check to make sure that the amount parameter is greater than zero before making the transfer to the recipient address. If the amount is zero or negative, the function will revert and the transfer will not be made.
```

```solidity
94        _underlyingAddress.safeTransfer(_recipient, _amount);
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol#L94

```solidity
160        _token.safeTransfer(msg.sender, _amount);
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L160

```solidity
311        _hToken.safeTransfer(_to, _amount);
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L311

## [G‑10] Use assembly to hash instead of solidity
```solidity
function solidityHash(uint256 a, uint256 b) public view {
    //unoptimized
    keccak256(abi.encodePacked(a, b));
}
```
Gas: 313

```solidity
function assemblyHash(uint256 a, uint256 b) public view {
    //optimized
    assembly {
        mstore(0x00, a)
        mstore(0x20, b)
        let hashedVal := keccak256(0x00, 0x40)
    }
} 
```
Gas:231

```solidity
362        newAccount = new VirtualAccount{salt: keccak256(abi.encode(_user))}(_user, address(this));
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L362

## [G‑11] Loop best practice to save gas
```solidity
function Plusi() public view {
		for(uint i=0; i<10;) {
			console.log("Number ==",i);
			unchecked{
				++i;
			}
		}
	}
best practice
-----------------------------------------------------
for (uint i = 0; i < length; i = unchecked_inc(i)) {
    // do something that doesn't change the value of i
}
function unchecked_inc(uint i) returns (uint) {
    unchecked {
        return i + 1;
    }
}
```

```solidity
195            unchecked {
                ++i;
            }
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L195

```solidity
450            unchecked {
                ++i;
            }
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L450

```solidity
563            unchecked {
                ++i;
            }
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L563

## [G‑12] Gas savings can be achieved by changing the model for assigning value to the structure
Here’s an example to illustrate this:
```solidity
struct MyStruct {
    uint256 a;
    uint256 b;
    uint256 c;
}
function assignValuesToStruct(uint256 _a, uint256 _b, uint256 _c) public {
    MyStruct memory myStruct = MyStruct(_a, _b, _c);
    // Do something with myStruct
}
```
In this example, we have a simple MyStruct data structure with three uint256 fields. The assignValuesToStruct function takes three input parameters _a, _b, and _c, and assigns them to the corresponding fields in a new myStruct variable. This is done using the struct constructor syntax, which creates a new instance of the MyStruct struct with the specified field values.

This approach can be more efficient than assigning values to the struct fields one by one, like this:
```solidity
function assignValuesToStruct(uint256 _a, uint256 _b, uint256 _c) public {
    MyStruct memory myStruct;
    myStruct.a = _a;
    myStruct.b = _b;
    myStruct.c = _c;
    // Do something with myStruct
}
```
In this example, the values of _a, _b, and _c are assigned to the corresponding fields of the myStruct variable one by one. This can be less efficient than using the struct constructor syntax, because it requires more memory operations to initialize the struct fields.

By using the struct constructor syntax to assign values to the struct fields, we can save gas by reducing the number of memory operations required to create the struct.

```solidity
72        SettlementParams memory sParams = SettlementParams({
            settlementNonce: uint32(bytes4(_payload[PARAMS_START_SIGNED:PARAMS_TKN_START_SIGNED])),
            recipient: _recipient,
            hToken: address(uint160(bytes20(_payload[PARAMS_TKN_START_SIGNED:45]))),
            token: address(uint160(bytes20(_payload[45:65]))),
            amount: uint256(bytes32(_payload[65:97])),
            deposit: uint256(bytes32(_payload[97:PARAMS_SETTLEMENT_OFFSET]))
        });
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgentExecutor.sol#L72

```solidity
402                DepositParams({
                    hToken: _dParams.hTokens[i],
                    token: _dParams.tokens[i],
                    amount: _dParams.amounts[i],
                    deposit: _dParams.deposits[i],
                    depositNonce: 0
                })
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L402

```solidity
88        DepositParams memory dParams = DepositParams({
            depositNonce: uint32(bytes4(_payload[PARAMS_START:PARAMS_TKN_START])),
            hToken: address(uint160(bytes20(_payload[PARAMS_TKN_START:PARAMS_TKN_START_SIGNED]))),
            token: address(uint160(bytes20(_payload[PARAMS_TKN_START_SIGNED:45]))),
            amount: uint256(bytes32(_payload[45:77])),
            deposit: uint256(bytes32(_payload[77:PARAMS_TKN_SET_SIZE]))
        })
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L88

```solidity
173        DepositParams memory dParams = DepositParams({
            depositNonce: uint32(bytes4(_payload[PARAMS_START_SIGNED:PARAMS_TKN_START_SIGNED])),
            hToken: address(uint160(bytes20(_payload[PARAMS_TKN_START_SIGNED:45]))),
            token: address(uint160(bytes20(_payload[45:65]))),
            amount: uint256(bytes32(_payload[65:97])),
            deposit: uint256(bytes32(_payload[97:PARAMS_SETTLEMENT_OFFSET]))
        })
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L173

```solidity
338        dParams = DepositMultipleParams({
            numberOfAssets: numOfAssets,
            depositNonce: nonce,
            hTokens: hTokens,
            tokens: tokens,
            amounts: amounts,
            deposits: deposits
        })
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L338

## [G‑13] Use assembly for math (add, sub, mul, div)
Use assembly for math instead of Solidity. You can check for overflow/underflow in assembly to ensure safety. If using Solidity versions < 0.8.0 and you are using Safemath, you can gain significant gas savings by using assembly to calculate values and checking for overflow/underflow.
```solidity
Addition:

//addition in Solidity
function addTest(uint256 a, uint256 b) public pure {
    uint256 c = a + b;
}
Gas: 303

//addition in assembly
function addAssemblyTest(uint256 a, uint256 b) public pure {
    assembly {
        let c := add(a, b)
        if lt(c, a) {
            mstore(0x00, "overflow")
            revert(0x00, 0x20)
        }
    }
}
Gas: 263

Subtraction:

//subtraction in Solidity
function subTest(uint256 a, uint256 b) public pure {
  uint256 c = a - b;
}
Gas: 300

//subtraction in assembly
function subAssemblyTest(uint256 a, uint256 b) public pure {
    assembly {
        let c := sub(a, b)
        if gt(c, a) {
            mstore(0x00, "underflow")
            revert(0x00, 0x20)
        }
    }
}
Gas: 263

Multiplication:

//multiplication in Solidity
function mulTest(uint256 a, uint256 b) public pure {
    uint256 c = a * b;
}
Gas: 325

//multiplication in assembly
function mulAssemblyTest(uint256 a, uint256 b) public pure {
    assembly {
        let c := mul(a, b)
        if lt(c, a) {
            mstore(0x00, "overflow")
            revert(0x00, 0x20)
        }
    }
}
Gas: 265

Division:

//division in Solidity
function divTest(uint256 a, uint256 b) public pure {
    uint256 c = a * b;
}
Gas: 325

//division in assembly
function divAssemblyTest(uint256 a, uint256 b) public pure {
    assembly {
        let c := div(a, b)
        if gt(c, a) {
            mstore(0x00, "underflow")
            revert(0x00, 0x20)
        }
    }
}
Gas: 265

```solidity
111        uint256 settlementEndOffset = PARAMS_START_SIGNED + PARAMS_TKN_START + assetsOffset;
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgentExecutor.sol#L111

```solidity
155        getStrategyTokenDebt[_token] = _strategyTokenDebt + _amount;
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L155

```solidity
473        return ((_currBalance + _strategyTokenDebt) * getMinimumTokenReserveRatio[_token]) / DIVISIONER;
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L473


## [G‑14] Access mappings directly rather than using accessor functions
Saves having to do two JUMP instructions, along with stack setup.
```solidity
136        return getSettlement[_settlementNonce];
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L136

## [G‑15] Use mappings instead of arrays
Arrays are useful when you need to maintain an ordered list of data that can be iterated over, but they have a higher gas cost for read and write operations; especially when the size of the array is large. This is because Solidity needs to iterate over the entire array to perform certain operations, such as finding a specific element or deleting an element.

Mappings, on the other hand, are useful when you need to store and access data based on a key, rather than an index. Mappings have a lower gas cost for read and write operations, especially when the size of the mapping is large, since Solidity can perform these operations based on the key directly, without needing to iterate over the entire data structure.
```solidity
35    address[] public bridgeAgents;
45    address[] public bridgeAgentFactories;
55    address[] public strategyTokens;
71    address[] public portStrategies;
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L35

```solidity
67    address[] public bridgeAgents;
80    address[] public bridgeAgentFactories;
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L67

## [G‑16] Use Short-Circuiting rules to your advantage
When using logical disjunction (||) or logical conjunction (&&), make sure to order your functions correctly for optimal gas usage. In logical disjunction (OR), if the first function resolves to true, the second one won’t be executed and hence, save you gas. In logical disjunction (AND), if the first function evaluates to false, the next function won’t be evaluated. Therefore, you should order your functions accordingly in your solidity code to reduce the probability of needing to evaluate the second function.

```solidity
363        if (_minimumReservesRatio >= DIVISIONER || _minimumReservesRatio < MIN_RESERVE_RATIO) {
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L363


## [G‑17] Use ERC721A instead ERC721
The ERC721A is an improvement standard for ERC721 tokens. It was proposed by the Azuki team and used for developing their NFT collection. Compared with ERC721, ERC721A is a more gas-efficient standard to mint a lot of NFTs simultaneously. It allows developers to mint multiple NFTs at the same gas price. This has been a great improvement due to Ethereum’s sky-rocketing gas fee.

https://nextrope.com/erc721-vs-erc721a-2/



## [G‑18]  Add unchecked {} for subtractions where the operands cannot underflow because of a previous require() or if-statement

require(a <= b); x = b - a => require(a <= b); unchecked { x = b - a }.

```solidity
173        if (_deposit > 0) {
174            _token.safeTransferFrom(msg.sender, address(this), _deposit);
175            ERC20(_token).approve(_localPortAddress, _deposit);
176        }
```

https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L173