#Low 

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/ArbitrumBranchPort.sol#L50-L51

1- ` ArbitrumBranchPort.sol`  function `depositToPort` takes the input from `ArbitrumBranchBridgeAgent` during the deposit to port,  inputs aren't validated it is important to validate the input parameters to mitigate any unforeseen issues, parameters like  `_underlyingAddress` should be validated to ensure the token address are correct, not zero address and aren't malicious addresses before making the call to transfer tokens to the contract
```
_underlyingAddress.safeTransferFrom(_depositor, address(this), _deposit);
```

##FIX 
Validate the address isn't zero address by adding a check 

```
require (_underlyingAddress !=  Address(0))
```
This would ensure the address passed isn't a zero address and the token transfer can proceed.




2-
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L264-L271

 in `Rootport.sol` when setting the local address the `_localAddress` input is validated not to be a null/zero address this ensures that the _`localAddress` parameter is valid, However the address of the `_globalAddress` isn't validated.

This can lead to issues where if the `_globalAddress` input is null/zero it will lead to false retrieval of the Addresses

```
if (_localAddress == address(0)) revert InvalidLocalAddress();

        getGlobalTokenFromLocal[_localAddress][_srcChainId] = _globalAddress;
        getLocalTokenFromGlobal[_globalAddress][_srcChainId] = _localAddress;
```        
##FIX 
Add a condition to check if the `_globalAddress` is zero/null

```
if (_globalAddress == address(0)) revert InvalidLocalAddress();
```

3-
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L276-L282

 Lack of Input Validation for ` _recipient`  Address in ` bridgeToRoot`  Function

In the `RootPort` contract the   `bridgeToRoot` function is called by a `BridgAgent`, this function is responsible for  bridging tokens from a source chain to the root chain.
However the  `bridgeToRoot`  function lacks proper input validation for the  `_recipient`  address parameter. This omission poses a risk and may result in the loss of tokens or unintended transfers.

```
Function bridgeToRoot(address _recipient, address _hToken, uint256 _amount, uint256 _deposit, uint256 _srcChainId)
```
Without proper validation, tokens could be inadvertently sent to an address that is not intended to receive them. This could result in the permanent loss of tokens or make them irretrievable.

##FIX 
it is recommended to implement proper input validation for the  `_recipient`  address. This can be achieved by adding a validation step to ensure that the  `_recipient`  address is address and is not zero or empty.

``` 
// Validate _recipient address
require(_recipient != address(0), "Invalid recipient address");
```

4- 
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L294-L299

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L304-L309

Lack of Validation for  _amount  Parameter in bridgeToRootFromLocalBranch  and bridgeToLocalBranchFromRoot 


The  `bridgeToRootFromLocalBranch`  and  `bridgeToLocalBranchFromRoot`  functions in the `RootPort` contract lack proper validation for the  `_amount`  parameter.

In the  `bridgeToRootFromLocalBranch`  function, the ` _amount`  represents the amount of tokens being transferred from ` _from`  to the contract. It is important to validate that the  `_amount`  is a positive value and does not exceed the balance of tokens that the ` _from`  address

Similarly, in the ` bridgeToLocalBranchFromRoot`  function, the  `_amount`  parameter represents the amount of tokens being transferred from the contract to the ` _to`  address. It is important to validate that the ` _amount`  is a positive value and does not exceed the balance of tokens held by the contract.

```
Function bridgeToRootFromLocalBranch(address _from, address _hToken, uint256 _amount)
 
Function bridgeToLocalBranchFromRoot(address _to, address _hToken, uint256 _amount) 
```
Without proper validation, the  `_amount`  parameter may allow invalid or zero values, resulting in unintended transfers or errors in the token transfer functionality.

##FIX 
Implement proper input validation for the  `_amount` parameter in both the  `bridgeToRootFromLocalBranch`  and ` bridgeToLocalBranchFromRoot`  functions. Ensure that the ` _amount`  is a positive value and does not exceed the available balance of tokens.

```
// Validate _amount parameter
require(_amount > 0, "Invalid amount");
```
5-
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L231-L237

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L209-L210

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L276-L277

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L306-L307


When users want to make deposits they call `callOutAndBridge` `callOutAndBridgeMultiple` `callOutSignedAndBridge` `callOutSignedAndBridgeMultiple` these functions are responsible for the creation of deposits and the use of the fallback toggle which enables the airdrops function. 

They further call `createDeposit` `createDepositMultiple` where there are calls to the port to deposit the assets. Before making the cross chain request 

It is advisable to include sanity checks in `createDeposit``createDepositMultiple` on the `hToken` address and `underlyingAddress` in the `_params`  if any of them is a null(0)  address it can distrupt the system functionality 

Add a require method in the `createDeposit` and `createDepositMultiple` to check the sanity of the addresses 

```
require (_hToken != address(0))
require (_token != addresss(0))
 //this ensures only valid addresses are passed as arguments 
```

6-
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L359-L368

##TITLE
The addVirtualAccount function in RootPort.sol doesn't check for duplicates

Virtual accounts helps users manage their tokens across multiple chains from any Ulysses branch chain making it easy for users to interact with the different chains seemlessly. However the  `addVirtualAccount` function does not include a check to verify if a virtual account already exists for the given user address  `_user` .

##IMPACT 
Without the check, users can  repeatedly request to add a new virtual account  with the same user address, creating multiple virtual accounts for the same user. This could lead to confusion, data inconsistency, or even exploitation of vulnerabilities arising from multiple instances of the same user account. This could include unauthorized access, tampering with balances or transactions, or attempting to bypass certain security measures.

##POC
 ```
 function addVirtualAccount(address _user) internal returns (VirtualAccount newAccount) {
        if (_user == address(0)) revert InvalidUserAddress();

        newAccount = new VirtualAccount{salt: keccak256(abi.encode(_user))}(_user, address(this));
        getUserAccount[_user] = newAccount;

        emit VirtualAccountCreated(_user, address(newAccount));
    }
 ```
 There's no check if `_user` already created an account leading to multiple accounts being created.

##FIX 

```
function addVirtualAccount(address _user) internal returns (VirtualAccount newAccount) {
    if (_user == address(0)) revert InvalidUserAddress();
    
// Check if virtual account already exists

    require(getUserAccount[_user] == address(0), "Virtual account already exists for the user"); 

    newAccount = new VirtualAccount{salt: keccak256(abi.encode(_user))}(_user, address(this));
    getUserAccount[_user] = newAccount;

    emit VirtualAccountCreated(_user, address(newAccount));
}
```

By including this additional check, the function will prevent the creation of duplicate virtual accounts for the same user address and ensure data consistency within the system.


7-
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L86-L102

##Title 
Incorrectly implemented mappings for locating hTokens and their source chains with the underlyingAddress.


`RootPort` contract mapping of `hTokens` are incorrectly implemented, leading to potential issues in locating tokens and their corresponding addresses. The mappings are crucial for identifying and retrieving the global addresses, local addresses, and underlying addresses associated with specific chainIds.
there are some syntax errors in the mapping declarations. The correct syntax for mapping declarations should be  `mapping(keyType => valueType) public mappingName; `. 

##Impact 
Tokens may be incorrectly associated with their global addresses, local addresses, or underlying addresses. This can result in confusion and incorrect identification of tokens when trying to locate them within the system.
Due to the incorrect mappings, it may become challenging to locate tokens and their source chains accurately. This can hinder the functionality of the system and prevent proper operations involving specific tokens.


```
/// @notice Mapping with all global hTokens deployed in the system.
mapping(address token => bool isGlobalToken) public isGlobalAddress;

/// @notice ChainId -> Local Address -> Global Address
mapping(address chainId => mapping(uint256 localAddress => address globalAddress)) public getGlobalTokenFromLocal;

/// @notice ChainId -> Global Address -> Local Address
mapping(address chainId => mapping(uint256 globalAddress => address localAddress)) public getLocalTokenFromGlobal;

/// @notice ChainId -> Underlying Address -> Local Address
mapping(address chainId => mapping(uint256 underlyingAddress => address localAddress)) public getLocalTokenFromUnderlying;

/// @notice Mapping from Local Address to Underlying Address.
mapping(address chainId => mapping(uint256 localAddress => address underlyingAddress)) public getUnderlyingTokenFromLocal;
```
If the mappings are not consistently updated or maintained, it can result in data inconsistencies. For example, if a token's global address is not correctly associated with its local or underlying address, it can lead to incorrect data retrieval and potential mismatches between different mappings.

The correct syntax for mapping declarations should be ` mapping(keyType => valueType) public mappingName;`

For example the `getUnderlyingTokenFromGlobal`  function will not work as intended.

The function receives two parameters: ` _globalAddress ` and ` _srcChainId` .
It attempts to retrieve the  localAddress  associated with the ` _globalAddress`  and  `_srcChainId`  using the ` getLocalTokenFromGlobal`  mapping:

```
 address localAddress = getLocalTokenFromGlobal[_globalAddress][_srcChainId];
```

However, since the`  getLocalTokenFromGlobal  `mapping is incorrectly implemented, this retrieval may not return the intended ` localAddress `. It can lead to incorrect or missing values, causing unexpected behavior in the subsequent steps.
The function then tries to retrieve the underlying address associated with the obtained  localAddress  and ` _srcChainId ` using the  `getUnderlyingTokenFromLocal ` mapping:



`return getUnderlyingTokenFromLocal[localAddress][_srcChainId];`

Similar to the previous step, the incorrect implementation of the  `getUnderlyingTokenFromLocal`  mapping can cause inaccurate or missing values. This can result in returning an incorrect underlying address or potentially throwing an error if the  `localAddress ` is not found in the mapping.


##FIX 
Apply the fix to `Bridge Agent Factory` , Bridge Agents`, `Virtual Account ` mappings also.

it is recommended to correct the mappings as follows:


``` 
/// @notice Mapping with all global hTokens deployed in the system.
mapping(address => bool) public isGlobalAddress;

/// @notice ChainId -> Local Address -> Global Address
mapping(address => mapping(uint256 => address)) public getGlobalTokenFromLocal;

/// @notice ChainId -> Global Address -> Local Address
mapping(address => mapping(address => address)) public getLocalTokenFromGlobal;

/// @notice ChainId -> Underlying Address -> Local Address
mapping(address => mapping(address => address)) public getLocalTokenFromUnderlying;

/// @notice Mapping from Local Address to Underlying Address.
mapping(address => mapping(address => address)) public getUnderlyingTokenFromLocal;
```
By correcting the mappings as above, you ensure that the addresses are associated correctly, allowing for accurate identification and retrieval of tokens and their associated addresses.


9-
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L362-L366

Redundant check in the bridgeIn function of RootBridgeAgent

The `bridgeIn` function makes the necessary checks to ensure `amount` declared in the `_dparams` is greater than the `_dparams` deposit this is the first check

```
if (_dParams.amount < _dParams.deposit) revert InvalidInputParams();
```
The check already ensures that the amount is greater than the deposit which indicates amount can't be zero in any case, however during the deposit process of the `localToken` there is an additional check if the amount is greater than zero.

```if (_dParams.amount > 0) {
            if (!IPort(_localPortAddress).isLocalToken(_dParams.hToken, _srcChainId)) {
                revert InvalidInputParams();
            }
        }
  ```
This check is not needed here as the check was already made.

10-
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L359-L360

addVirtualAccount isn't following the solidity style guide 

In solidity every internal function should be prefixed with an underscore "_" for readability the addVirtualAccount is an internal function that is missing such underscore, users might get confused as to what the function does 
When users want to create a new virtual account the call `fetchVirtualAccount` 

```
   function fetchVirtualAccount(address _user) external override returns (VirtualAccount account) {
        account = getUserAccount[_user];
        if (address(account) == address(0)) account = addVirtualAccount(_user);
    }

    /**
     * @notice Creates a new virtual account for a user.
     * @param _user address of the user to associate a virtual account with.
     */
    function addVirtualAccount(address _user) internal returns (VirtualAccount newAccount) {
        if (_user == address(0)) revert InvalidUserAddress();

        newAccount = new VirtualAccount{salt: keccak256(abi.encode(_user))}(_user, address(this));
        getUserAccount[_user] = newAccount;

        emit VirtualAccountCreated(_user, address(newAccount));
    }
```
The  `fetchVirtualAccount`  function first checks if the virtual account for a given user address is already created. If the account is not yet created `(address(account) == address(0)),` it proceeds to create a new virtual account for that user by calling the  `addVirtualAccount`  function.

The  `addVirtualAccount ` function takes the user's address as a parameter and creates a new instance of the  `VirtualAccount ` contract. It assigns the new account to the user by storing it in the ` getUserAccount  `mapping with the user's address as the key.

It is recommended to add the prefix _addVirtualAccount` to the function for better understanding in the case of users.

11- 
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L855-L870

Users can mistakenly send their `retrySettlement` deposit to a zero address and can lead to loss of funds 

The `_performRetrySettlementCall` performs calls to the layerZero endpoint and sends assets to the `_recipient` address on the destination chain this function lacks the sanity check lf the `_recipient`  is mistakenly set to a dead address the users losses funds 
```
function _performRetrySettlementCall(
        bool _hasFallbackToggled,
        address[] memory _hTokens,
        address[] memory _tokens,
        uint256[] memory _amounts,
        uint256[] memory _deposits,
        bytes memory _params,
        uint32 _settlementNonce,
        address payable _refundee,
        address _recipient,
        uint16 _dstChainId,
        GasParams memory _gParams,
        uint256 _value
    ) internal {
 ```

Also in `MultiCallRootRouter` the following functions should validate the `_reciepient` address on the destination chain this functions lacks the sanity check lf the `_recipient`  is mistakenly set to a dead address the users losses funds 

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/MulticallRootRouter.sol#L508-L517

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/MulticallRootRouter.sol#L544-L553



```
function _approveAndCallOut(
        address refundee,
        address recipient,
        address outputToken,
        uint256 amountOut,
        uint256 depositOut,
        uint16 dstChainId,
        GasParams memory gasParams
    ) internal virtual {
```
```
function _approveAndCallOut(
        address refundee,
        address recipient,
        address outputToken,
        uint256 amountOut,
        uint256 depositOut,
        uint16 dstChainId,
        GasParams memory gasParams
    ) internal virtual {
```   

Including a check that the `_recipient` isn't a zero address is recommended.

12-
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/MulticallRootRouter.sol#L501-L502

`MultiCallRootRouter#_approveAndCallOut` Natspec have a typo adn should be changed to and 

```
*  @param refundee settlement owner adn excess gas receiver.
```

`CoreRootRouter#_syncBranchBridgeAgent` Natspec have a typo BRanch should be changed to Branch

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/CoreRootRouter.sol#L494-L495

```
* @dev Internal function sync a Root Bridge Agent with a newly created BRanch Bridge Agent.
```
13- https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/factories/ERC20hTokenBranchFactory.sol#L101-L105


During token creation in `ERC20hTokenFactory` the `_name`, `_symbol`, `_decimal` for the new token being created aren't checked if they are acceptable by the Ominichain in the case of the ownership of `RootPort` random tokens will be created with different decimals which might harm system functionality

```
newToken = new ERC20hTokenRoot(
            localChainId,
            address(this),
            rootPortAddress,
            _name,
            _symbol,
            _decimals
        );
        hTokens.push(newToken);
    }
 ```
 Add sanity checks on the new token parameters and revert before pushing it

14- https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L1176-L1180

One `BranchBridgeAgent` might represent multiple chains 

 `BranchBridgeAgent` represents a specific chain allowing communication between other Bridges of other chains. If a `BranchBridgeAgent` can represent multiple chain there might be situations where the `BranchBridgeAgent` sends requests to the wrong chain Bridge, 
This will inconvenience users and night lead to loss of funds 

```
getBranchBridgeAgent[_branchChainId] = _newBranchBridgeAgent;
        getBranchBridgeAgentPath[_branchChainId] = abi.encodePacked(_newBranchBridgeAgent, address(this));
    }
```
Include checks if a `BranchBridgeAgent` have been synced to a chain before syncing.