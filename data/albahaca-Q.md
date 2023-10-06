# Low


# Summary
|      | issue | insteance |
|------|-------|-----------|
|[L-01]|_safeMint() should be used rather than _mint() wherever possible|2|
|[L-02]|Contracts are not using their OZ upgradeable counterparts|4|
|[L-03]|Use of transfer instead of safeTransfer|2|
|[L-04]|no check if the burn amount is zero or not|2|
|[L-05]|Unused/empty receive()/fallback() function|3|
|[L-06]|Gas griefing/theft is possible on unsafe external call|4|
|[L-07]|Arbitrary 'from' Address in safeTransferFrom Method Poses Potential Security Risk in ArbitrumBranchPort.depositToPort Function|8|
|[L-08]|Ignored Return Value from External Call|13|
|[L-09]|Missing Event Emission for Critical Parameter Changes|6|
|[L-10]|Dynamic Type Usage in abi.encodePacked() in BranchBridgeAgent.sol|12|
|[L-11]|Ignoring Return Value of Low-Level Call in RootBridgeAgent.sol|2|
|[L-12]|Dubious Typecasting in Function|6|
|[L-13]|Potential Security Risk due to Lack of Approval Check in NFT Transfers|1|



## [L-01] _safeMint() should be used rather than _mint() wherever possible
_mint() is [discouraged](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/d4d8d2ed9798cc3383912a23b5e8d5cb602f7d4b/contracts/token/ERC721/ERC721.sol#L271) in favor of _safeMint() which ensures that the recipient is either an EOA or implements IERC721Receiver. Both open [OpenZeppelin](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/d4d8d2ed9798cc3383912a23b5e8d5cb602f7d4b/contracts/token/ERC721/ERC721.sol#L238-L250) and [solmate](https://github.com/Rari-Capital/solmate/blob/4eaf6b68202e36f67cab379768ac6be304c8ebde/src/tokens/ERC721.sol#L180) have versions of this function so that NFTs aren’t lost if they’re minted to contracts that cannot transfer them back out.

```solidity
30  _mint(account, amount);
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/token/ERC20hTokenBranch.sol#L30

### same issue in these lins of code:
https://github.com/code-423n4/2023-09-maia/blob/main/src/token/ERC20hTokenRoot.sol#L59


## [L-02] Contracts are not using their OZ upgradeable counterparts
Description
The non-upgradeable standard version of OpenZeppelin's library, such as `Ownable`, `Pausable`, `Address`, `Context`, `SafeERC20`, `ERC1967Upgrade` etc, are inherited / used by both the proxy and the implementation contracts.

As a result, when attempting to use the upgrades plugin mentioned, the following errors are raised:

```
Error: Contract `FiatTokenV1` is not upgrade safe

contracts/v1/FiatTokenV1.sol:58: Variable `totalSupply_` is assigned an initial value
  Move the assignment to the initializer
  https://zpl.in/upgrades/error-004

contracts/v1/Pausable.sol:49: Variable `paused` is assigned an initial value
  Move the assignment to the initializer
  https://zpl.in/upgrades/error-004

contracts/v1/Ownable.sol:28: Contract `Ownable` has a constructor
  Define an initializer instead
  https://zpl.in/upgrades/error-001

contracts/util/Address.sol:186: Use of delegatecall is not allowed
  https://zpl.in/upgrades/error-002
       
```
Having reviewed these errors, none had any adversarial impact:

`totalSupply_` and `paused` are explictly assigned the default values `0` and `false`
the implementation contracts utilises the internal `_transferOwnership()` in the initializer, thus transferring ownership to `newOwner` regardless of who the current owner is
`Address's` `delegatecall` is only used by the `ERC1967Upgrade` contract. Comparing both the `Address` and `ERC1967Upgrade` contracts against their upgradeable counterparts show similar behaviour (differences are some refactoring done to shift the delegatecall into the `ERC1967Upgrade` contract).
Nevertheless, it would be safer to use the upgradeable versions of the library contracts to avoid unexpected behaviour.

```solidity
9   import {ERC1155Receiver} from "@openzeppelin/contracts/token/ERC1155/utils/ERC1155Receiver.sol";
10  import {IERC1155Receiver} from "@openzeppelin/contracts/token/ERC1155/IERC1155Receiver.sol";
11  import {IERC721Receiver} from "@openzeppelin/contracts/token/ERC721/IERC721Receiver.sol";
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L9-L11

```solidity
4   import {IERC721Receiver} from "@openzeppelin/contracts/token/ERC721/IERC721Receiver.sol";
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/interfaces/IVirtualAccount.sol#L4

### Recommended Mitigation Steps
Where applicable, use the contracts from `@openzeppelin/contracts-upgradeable` instead of `@openzeppelin/contracts`.

## [L-03] Use of transfer instead of safeTransfer
It is good to add a require statement that checks the return value of token transfers or to use something like OpenZeppelin's safeTransfer/safeTransferFrom unless one is sure the given token reverts in case of a failure. Failure to do so will cause silent failures of transfers and affect token accounting in contract.

```solidity
62   ERC721(_token).transferFrom(address(this), msg.sender, _tokenId);
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L62


## [L-04] no check if the burn amount is zero or not
if the amount is zero so the unhecked block will divided zero on 3 and we use gas for nothing ! if we set zero we may pass the _burn checks, i know it is passed by only permit but it's better to avoid this happen because it's seting by human and it means it can be set with 0 balance to burn !

```solidity
36   _burn(msg.sender, amount);
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/token/ERC20hTokenBranch.sol#L36

### same issue in these lins of code:
https://github.com/code-423n4/2023-09-maia/blob/main/src/token/ERC20hTokenRoot.sol#L71


## [L‑05] Unused/empty receive()/fallback() function
If the intention is for the Ether to be used, the function should call another function, otherwise it should revert (e.g. require(msg.sender == address(weth)))

```solidity
149   receive() external payable {}
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L149

### same issue in these lins of code:
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L128

https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L44

## [L-06]Gas griefing/theft is possible on unsafe external call
return data (bool success,) has to be stored due to EVM architecture, if in a usage like below, 'out' and 'outsize' values are given (0,0) . Thus, this storage disappears and may come from external contracts a possible Gas griefing/theft problem is avoided

```solidity
717  (bool success,) = bridgeAgentExecutorAddress.call{value: address(this).balance}(_calldata);

737  (bool success,) = bridgeAgentExecutorAddress.call{value: address(this).balance}(_calldata);
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L717

### same issue in these lins of code:
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L754

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L779


## [L-07] Arbitrary 'from' Address in safeTransferFrom Method Poses Potential Security Risk in ArbitrumBranchPort.depositToPort Function
The current implementation of the depositToPort function allows for an arbitrary 'from' address to be used in the safeTransferFrom method. This could potentially lead to unauthorized transfers if the '_depositor' address is not properly validated.

```solidity
66    _underlyingAddress.safeTransferFrom(_depositor, address(this), _deposit);
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol#L66



Recommendation: To mitigate this potential security risk, it is recommended to use msg.sender as the 'from' address in the safeTransferFrom method. This ensures that only the address that called the function can transfer funds, adding an extra layer of security to the contract.


### same issue in these lins of code:

https://github.com/code-423n4/2023-09-maia/blob/main/src/ArbitrumBranchPort.sol#L128

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L526

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L532

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L1151

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L301

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L787-L794

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L940-L947

## [L-08] Ignored Return Value from External Call
The function _transferAndApproveToken ignores the return value of the approve function call from the ERC20 token contract. This can lead to unexpected behavior if the call fails but the contract continues execution.
```solidity
168    ERC20(_hToken).approve(_localPortAddress, _amount - _deposit);
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L168


Recommendation:
Check the return value of the approve call and handle any failures appropriately. For example:
```solidity
bool success = ERC20(_hToken).approve(_localPortAddress, _amount - _deposit);
require(success, "Token approval failed");
```

This will revert the transaction if the approve call fails, preventing potential issues.

### same issue in these lins of code:

https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L175

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L503

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L168

https://github.com/code-423n4/2023-09-maia/blob/main/src/CoreBranchRouter.sol#L175

https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L416

https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L452

https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L248

https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L425

https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L337

https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L328

https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L239

https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L264





## [L-09] Missing Event Emission for Critical Parameter Changes
The function initialize in the BaseBranchRouter contract sets the bridgeAgentExecutorAddress but does not emit an event to log this change. This can make it difficult to track changes to critical access control parameters.

```solidity
    function initialize(address _localBridgeAgentAddress) external onlyOwner {
        require(_localBridgeAgentAddress != address(0), "Bridge Agent address cannot be 0");
        localBridgeAgentAddress = _localBridgeAgentAddress;
        localPortAddress = IBridgeAgent(_localBridgeAgentAddress).localPortAddress();
        bridgeAgentExecutorAddress = IBridgeAgent(_localBridgeAgentAddress).bridgeAgentExecutorAddress();
        
        renounceOwnership();
    }
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L60-L67

Recommendation:
Emit an event whenever bridgeAgentExecutorAddress is set. This will allow off-chain services and users to easily track and respond to changes. For example:

```solidity
event BridgeAgentExecutorAddressSet(address indexed newAddress);

function initialize(address _localBridgeAgentAddress) external onlyOwner {
    // existing code...
    bridgeAgentExecutorAddress = IBridgeAgent(_localBridgeAgentAddress).bridgeAgentExecutorAddress();
    emit BridgeAgentExecutorAddressSet(bridgeAgentExecutorAddress);
    // existing code...
}
```
This will provide a clear audit trail of changes to the bridgeAgentExecutorAddress.

### same issue in these lins of code:
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L129

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L334

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L64

https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenBranchFactory.sol#L74

https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/ERC20hTokenRootFactory.sol#L51


## [L-10] Dynamic Type Usage in abi.encodePacked() in BranchBridgeAgent.sol
Detect collision due to dynamic type usages in `abi.encodePacked`

```solidity
        bytes memory payload = abi.encodePacked(
            _hasFallbackToggled ? bytes1(0x86) : bytes1(0x06),
            msg.sender,
            uint8(_dParams.hTokens.length),
            _depositNonce,
            _dParams.hTokens,
            _dParams.tokens,
            _dParams.amounts,
            _dParams.deposits,
            _params
        );
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L317-L327

Recommendation:
Do not use more than one dynamic type in `abi.encodePacked()` (see the [Solidity documentation](https://solidity.readthedocs.io/en/v0.5.10/abi-spec.html?highlight=abi.encodePacked#non-standard-packed-modeDynamic)). Use `abi.encode()`, preferably.

### same issue in these lins of code:
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L171

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#219-L221

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L241-L250

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L317-L327

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L287-L296

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L362-L371

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L373-L381

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L386-L396

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L386-L396

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L398-L407

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L776

## [L-11] Ignoring Return Value of Low-Level Call in RootBridgeAgent.sol
The function _performRetrySettlementCall in the RootBridgeAgent contract has an issue where the return value of a low-level call is ignored. Specifically, the line callee.call{value: _value}(""); does not check or handle the return value.

```solidity
927   callee.call{value: _value}("");
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L927

Recommendation:
It is recommended to ensure that the return value of a low-level call is checked or logged. Ignoring the return value can lead to potential issues as it prevents the detection of failures or exceptions that might occur during the execution of the called contract. By verifying the return value, the contract can properly handle any errors or exceptions that occur during the external call.

To address this issue, consider modifying the code as follows:
```solidity
(bool success, bytes memory data) = callee.call{value: _value}("");

if (!success) {
    // Handle the failure or log the error
    // Example: revert("Low-level call failed");
}
```

### same issue in these lins of code:

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L835


## [L-12] Dubious Typecasting in Function
The function executeWithDeposit performs several typecasts from bytes to bytes4, bytes20, and bytes32. This is done to extract parameters from the _payload argument and create a DepositParams struct. However, the typecasting process is not clearly documented, which could lead to confusion or misinterpretation.

`Impact:` The lack of clear documentation and the use of magic numbers (like PARAMS_TKN_START, PARAMS_TKN_START_SIGNED, etc.) for slicing the _payload could lead to potential bugs or vulnerabilities. It also makes the code harder to read and understand, which could slow down development and increase the risk of errors.
```solidity
        DepositParams memory dParams = DepositParams({
            depositNonce: uint32(bytes4(_payload[PARAMS_START:PARAMS_TKN_START])),
            hToken: address(uint160(bytes20(_payload[PARAMS_TKN_START:PARAMS_TKN_START_SIGNED]))),
            token: address(uint160(bytes20(_payload[PARAMS_TKN_START_SIGNED:45]))),
            amount: uint256(bytes32(_payload[45:77])),
            deposit: uint256(bytes32(_payload[77:PARAMS_TKN_SET_SIZE]))
        });
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L88-L94

Recommendation: Consider using clear and descriptive constants for slicing the _payload. This will make the code more readable and reduce the risk of errors. Additionally, provide clear documentation explaining the typecasting process, including why it's necessary and how the _payload should be structured. This will make the function easier to use correctly and reduce the risk of misuse.

### same issue in these lins of code:
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L173-L179

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L286-L297

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L299-L310

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L312-L320

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L322-L330


## [L-13] Potential Security Risk due to Lack of Approval Check in NFT Transfers
The contract may be vulnerable to unauthorized access or manipulation due to the absence of an approval check before transferring Non-Fungible Tokens (NFTs).

Description
The withdrawERC721 function in the VirtualAccount contract transfers NFTs without checking if the contract has been approved to manage the NFTs on behalf of the owner. This could potentially allow unauthorized transfers of NFTs.

Potential Impact
Without an approval check, an attacker could potentially exploit this vulnerability to transfer NFTs without the owner's consent, leading to loss of assets.

```solidity
62    ERC721(_token).transferFrom(address(this), msg.sender, _tokenId);
```
https://github.com/code-423n4/2023-09-maia/blob/main/src/VirtualAccount.sol#L62

Recommendation
Implement an approval check before transferring NFTs. This can be done by calling the isApprovedOrOwner function of the ERC721 standard. This function checks if the contract is approved to manage the NFTs on behalf of the owner or if the contract is the owner of the NFT. If the contract is not approved or the owner, the function should revert to prevent unauthorized transfers.


