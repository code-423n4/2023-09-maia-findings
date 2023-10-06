
# Validation of tokens before adding them
the function allows the tokens to be added that can already exist on the chain, or not allowed.
## Instances 
CoreBranchRouter.sol #43
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L43
```javascript
function addGlobalToken(address _globalAddress, uint256 _dstChainId, GasParams[3] calldata _gParams)
        external
        payable
```
CoreBranchRouter.sol #166
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L166

BranchPort.sol #468
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L486

CoreBranchRouter.sol #67
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/CoreBranchRouter.sol#L62C1-L63C1

## Mitigation
* Check if the token already exists on the chain before adding a new token and when using an already existing token
* Check if the token is allowed on the chain

# Function is exposed to re-entrancy attack
functions not having the ```lock``` modifier are exposed to the re-entrancy attack, as they deal with external contracts
## Instances
CoreBranchRouter.sol #43
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L43
```javascript
 function addGlobalToken(address _globalAddress, uint256 _dstChainId, GasParams[3] calldata _gParams)
        external
        payable
   ```
BaseBranchRouter.sol #74
https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L74C25-L74C25
   ## Proof of concept
   since the function has the following line, which performs low-level external call, it is subjective to re-entrance attack 
   ```javascript
           IBridgeAgent(localBridgeAgentAddress).callOut{value: msg.value}(payable(msg.sender), payload, _gParams[0]);

```

## Mitigation
Consider adding the ```lock``` modifier to the function to guard it from re-entrancy attacks

# msg.value may be 0, leading to a break of logic
if the function is called with msg.value = 0 it may lead to non-expected, nonsanity behavior or worse, a complete break of code.
## Instances
CoreBranchRouter.sol #78
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/CoreBranchRouter.sol#L78C12-L78C12
```javascript
IBridgeAgent(localBridgeAgentAddress).callOutSystem{value: msg.value}(payable(msg.sender), payload, _gParams);
```
## Mitigation
Consider adding a checker that checks if  ```msg.value``` is greater than 0

# Initializing of already initialized contracts is allowed
contracts that are already initialized may be re-initialized which may lead to loss of data
## Instances 
* MultiCallRootRouter.sol https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L109
* BaseBranchRouter.sol https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L60

# Griefing possibility by bridge agents
## Instances
ArbitriumBranchPort.sol#50
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/ArbitrumBranchPort.sol#L50
```
    function depositToPort(address _depositor, address _recipient, address _underlyingAddress, uint256 _deposit)
        external
        override
        lock
        requiresBridgeAgent
```
## Proof of concept
The bridge agents have the griefing possibility, as they can deposit funds from any user at any time

# Code does not follow the comment
The code does not follow the instructions presented in the comments, and missing a statement
## Instances
MultiCallRootRouter.sol #565
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/MulticallRootRouter.sol#L565

```javascript
//Move output hTokens from Root to Branch and call 'clearTokens'.
        IBridgeAgent(_bridgeAgentAddress).callOutAndBridgeMultiple{value: msg.value}(
            payable(refundee),
            recipient,
            dstChainId,
            "",
            SettlementMultipleInput(outputTokens, amountsOut, depositsOut),
            gasParams,
            true
        );
   ```
   ## Mitigation
   implement the logic of calling ```clearTokens``` 




# The protocol is not pausable 
## Description
the protocol contracts do not implement pausing by any means, it can lead to major losses in the worst cases.
Pausing may be helpful in most scenarios, it is one of the major features to consider adding to the contracts
## Mitigation
Implement and make the contracts inherit ```pasuable```, then utilize the pause function across the function, and add roles for who can pause the contract.




# Branch Bridge may be created twice
## Instances
CoreBranchRouter.sol #216
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L216C8-L219C1
```
 address newBridgeAgent = IBridgeAgentFactory(_branchBridgeAgentFactory).createBridgeAgent(
            _newBranchRouter, _rootBridgeAgent, _rootBridgeAgentFactory
        );

```
## Proof of concept 
the method creates a branch bridge agent without checking if it already exists or not neither in the method nor in the factory, which may result in a clash or a failure of code.

## Mitigation
check if the branch bridge is already created or not before creating a new one, either in the method mentioned or in the factory creation method.
check if the branch bridge is not already created



# May return 0 fees on invalid data
## Instances
RootBridgeAgent.sol #140
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L140
```javascript
 (_fee,) = ILayerZeroEndpoint(lzEndpointAddress).estimateFees(
            _dstChainId,
            address(this),
            _payload,
            false,
            Abi.encodePacked(uint16(2), _gasLimit, _remoteBranchExecutionGas, getBranchBridgeAgent[_dstChainId])
        );
```
## Description
on invalid, nonexistent data, the function may return a 0 fee, instead of reverting 
## Mitigation
validate addresses, and check if chains exist before performing the external call

# Invalid  if-statement  will cause the function to not revert while it should 
## Instances

RootBridgeAgent.sol #430
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L430C1-L431C1
```javascript
if (!success) if (msg.sender == getBranchBridgeAgent[localChainId]) revert ExecutionFailure();
```
## Proof of concept 
On the condition that the not success and msg.sender does not equal getBranchBridgeAgent[localChainId] the function will not revert 

## Mitigation 
Change the code to be 
```javascript
if(!success || msg.sender == getBranchBridgeAgent[localChainId])  revert ExecutionFailure();
```
the code ensures correct logical execution, is easier to read, and is more predictable and versatile code

# Approve return value not checked 
## Description 
The approve method returns true on success and false on failure, the code mentioned below does not check the return value of the method approve, causing the code to not revert on failure.

## Instances
BaseBranchRouter.sol#168
 https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L168C1-L169C1
 ```javascript
 ERC20(_hToken).approve(_localPortAddress, _amount - _deposit);
```

BaseBranchRouter.sol #175
https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L175
```javascript
 ERC20(_token).approve(_localPortAddress, _deposit);
```
## Mitigation
check for the return value of the approve method, by adding a checker for the return value and revert on fail, while utilizing the use of error messages.

# Mint return value not checked 
## Description 
the mint method returns true on success and false on failure, the code mentioned below does not check the return value of the method approve, causing the code to not revert on failure.

## Instances
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L342
```
        if (!ERC20hTokenRoot(_hToken).mint(_to, _amount, localChainId)) revert UnableToMint();
```
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L503
```
        ERC20hTokenBranch(_localAddress).mint(_recipient, _amount);
```
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/ArbitrumBranchPort.sol#L69
```
        IRootPort(_rootPortAddress).mintToLocalBranch(_recipient, _globalToken, _deposit);
```

## Mitigation
check for the return value of the mint method, by adding a checker for the return value and revert on fail, while utilizing the use of error messages.



# Missing modifier on function
## Instances
RootBridgeAgentFactory.sol  #48
https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/RootBridgeAgentFactory.sol#L48C23-L48C23
```javascript
 function createBridgeAgent(address _newRootRouterAddress) external returns (address newBridgeAgent) {
        newBridgeAgent = address(
            new RootBridgeAgent(
                rootChainId, 
                lzEndpointAddress, 
                rootPortAddress, 
                _newRootRouterAddress
            )
        );

        IRootPort(rootPortAddress).addBridgeAgent(msg.sender, newBridgeAgent);
}
```
## Mitigation
Add a modifier on the function, or add a require block that checks for msg.sender to be the allowed addresses to create a bridge agent,


# Function will always revert due to invalid input 
## Instances
ArbitriumFactory.sol #41

https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ArbitrumBranchBridgeAgentFactory.sol#L37C8-L45C10
```javascript
        BranchBridgeAgentFactory(
            _rootChainId,
            _rootChainId,
            _rootBridgeAgentFactoryAddress,
            address(0),
            _localCoreBranchRouterAddress,
            _localPortAddress,
            _owner
        )
```
## Proof of concept 
The code references the BranchBridgeAgentFactory and passes address(0) as the fourth input, we can see below that the BranchBridgeAgentFactory constructor will revert on address(0) input.
https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/BranchBridgeAgentFactory.sol#L62C9-L65C11
```javascript
require(
            _lzEndpointAddress != address(0) || _rootChainId == _localChainId,
            "Layerzero Endpoint Address cannot be the zero address."
        );
```

## Mitigation
change the address(0) input into a valid address.




# Use safeTrasnferFrom 
## Instances
VirtualAccount.sol#L62
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/VirtualAccount.sol#L62
```javascript
ERC721(_token).transferFrom(address(this), msg.sender, _tokenId);
```

## Mitigation
Utilize the use of safeTransferFrom instead of transferFrom

# Decimals greater than 18 may break the contract
## Instances
CoreBranchRouter.sol #64
```javascript
  function addLocalToken(address _underlyingAddress, GasParams calldata _gParams) external payable virtual {
        //Get Token Info
        uint8 decimals = ERC20(_underlyingAddress).decimals();
```
CoreBranchRouter.sol #43

## Mitigation
Check if token decimals are smaller than or equal to 18, for more versatile and solid functionality

# Restructure of bulky functions
## Description
In the function mentioned below, there is a lot of bulky code with a lot of cases and external calls, as we can see the two methods mentioned below are nearly 200 lines each.
The code appears to be very complex and overwhelming making it hard to read and understand, even for the developer who wrote the code causing maintenance issues.

## Instances
BranchBridgeAgent.sol #343
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L343C14-L343C26

RootBridgeAgent.sol#434
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L434

## Mitigation
* Re-write the methods with separation of concerns type of architecture, by breaking down the logic of the function into multiple functions each with separate external call functions, and data handling functions instead of multiple if statements and processing the logic inside the if block 
* add more comments before making the external call and before handling the data inside the if statement, which describes why the if case is valid and what are the next steps and logic are inside the if block, These short and clear comments but rather effective and better for the eye.

# Add comments to describe bulky functions 
## Description
In the function mentioned below, the comments used in describing the bulky function do not mention anything about the functionality of the function logic, external calls, and functionality, what is present in the code base is a comment on the function signature, which gives no help at all.
## Instances

CoreBranchRouter.sol #86
https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L86
```
     /// _toggleBranchBridgeAgentFactory
        } else if (_params[0] == 0x03) {
            (address bridgeAgentFactoryAddress) = abi.decode(_params[1:], (address));

      _toggleBranchBridgeAgentFactory(bridgeAgentFactoryAddress);
```
another example 
```
  /// _removeBranchBridgeAgent
        } else if (_params[0] == 0x04) {
            (address branchBridgeAgent) = abi.decode(_params[1:], (address));

            _removeBranchBridgeAgent(branchBridgeAgent);

```

## Mitigation
Rather than using the same function signature as a comment, should add a couple more sentences describing the external call functionality and in what cases the external call gets executed 
example:
```
//When foo is found in bar, that means baz, and then a branch bridge agent is removed using _removeBranchBridgeAgent
 } else if (_params[0] == 0x04) {
            (address branchBridgeAgent) = abi.decode(_params[1:], (address));

            _removeBranchBridgeAgent(branchBridgeAgent);

```



















# Emit event when initialized and deployed 
## Instances 
ArbiriumBranchBridgeAgent.sol











# Address and values are used without validity checking
## Instances 
MultiCallRootRouter.sol #508
```javascript
function _approveAndCallOut(
        address refundee,
        address recipient,
        address outputToken,
        uint256 amountOut,
        uint256 depositOut,
        uint16 dstChainId,
        GasParams memory gasParams
    ) internal virtual 
  ```
  MultiCallRootRouter.sol #544

  ```javascript
function _approveMultipleAndCallOut(
        address refundee,
        address recipient,
        address[] memory outputTokens,
        uint256[] memory amountsOut,
        uint256[] memory depositsOut,
        uint16 dstChainId,
        GasParams memory gasParams
    )
   ```
 check if bridgeAgent is a valid address 
  CoreRootRouter.sol #83
  
check if brifgeAgentExecution addres is valid 
CoreRootRouter.sol #83

   check if _htokenFactory is valid
instances 
CoreRootRouter.sol #83

check if BranchBridge exists 
instances 
CoreRootRouter.sol #186

check if the refundee is a valid address 
instances 
CoreRootRouter.sol #186

check if the token is valid 
instances 
CoreRootRouter.sol #212

check if the refundee is a valid address 
instances 
CoreRootRouter.sol #212

check 0 address 
instances 
CoreRootRouter.sol #241

check if srcChainId is valid 
instances 
CoreRootRouter.sol #297

check for dstChainid , globalAdress is valid
instances 
CoreBranchRouter.sol #43

check if the address is valid 
instances 
CoreBranchRouter.sol #166

check if the chain ID is valid 
instances 
RootBridgeAgent.sol #105

check if the chain exists
instances 
RootBridgeAgent.sol #163

check valid address and chain 
instances 
RootBridgeAgent.sol #175

address 0 check 
instances 
RootBridgeAgent.sol #1072, 1073

check if the token address is valid 
instances 
BranchPort.sol #144, 167

check if msg.sender is a valid strategy 
instances 
BranchPort.sol #167

validate address 
instances 
BranchPort.sol #188

address validation at the constructor 
instances 
RootBridgeAgentFactory.sol 

address validation 
instances
ArbitriumFactory.sol #79


   ## Mitigation
 *  Check if the address is not equal to the zero address
*   Check if the address exists and valid
* Check if msg.values are greater than 0 and valid 














# Function reverts
## Instances 
RootBridgeAgentExecuter.sol #82, 150, 121



























