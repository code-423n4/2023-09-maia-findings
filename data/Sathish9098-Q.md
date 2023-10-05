# QA FINDINGS

##

## [L-1] ``_minimumReserves`` always returns 0 if the ``getMinimumTokenReserveRatio[_token]`` mapping not set 

### Impact
_minimumReserves() always returns return value as zero and the over all calculations are unwanted and waste of gas  

### POC

```solidity
FILE: 2023-09-maia/src/BranchPort.sol

468: function _minimumReserves(uint256 _strategyTokenDebt, uint256 _currBalance, address _token)
469:        internal
470:        view
471:        returns (uint256)
472:    {
473:        return ((_currBalance + _strategyTokenDebt) * getMinimumTokenReserveRatio[_token]) / DIVISIONER;
474:    }

```
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L468-L474

### Recommended Mitigation

Add if condition to check getMinimumTokenReserveRatio[_token] . If the value is 0 then directly return 0

```diff

+ if(getMinimumTokenReserveRatio[_token] ==0){
+ return 0;
+ }

```

																																								
																								


																		
																		


																																															





Multiplication on the result of a division	

Dividing an integer by another integer will often result in loss of precision. When the result is multiplied by another number, the loss of precision is magnified, often to material magnitudes. (X / Z) * Y should be re-written as (X * Y) / Z	 

(userCollateralShare[user] *   (EXCHANGE_RATE_PRECISION / FEE_PRECISION) *


Division by zero not prevented	

The divisions below take an input parameter which does not have any zero-value checks, which may lead to the functions reverting when zero is passed.	(EXCHANGE_RATE_PRECISION / FEE_PRECISION) *	


Allowed fees/rates should be capped by smart contracts	

Fees/rates should be required to be below 100%, preferably at a much lower limit, to ensure users don't have to monitor the blockchain for changes prior to using the protocol	

256      function setBigBangEthMarketDebtRate(uint256 _rate) external onlyOwner { , 257          bigBangEthDebtRate = _rate;																							
																										
Solidity version 0.8.20 may not work on other chains due to PUSH0	

The compiler for Solidity 0.8.20 switches the default target EVM version to [Shanghai](https://soliditylang.org/blog/2023/05/10/solidity-0.8.20-release-announcement/#important-note), which includes the new PUSH0 op code. This op code may not yet be implemented on all L2s, so deployment on these chains will fail. To work around this issue, use an earlier [EVM version](https://book.getfoundry.sh/reference/config/solidity-compiler#evm_version). While the project itself may or may not compile with 0.8.20, other projects with which it integrates, or which extend this project may, and those projects will have problems deploying these contracts/libraries.	

2:   pragma solidity ^0.8.18;, 0.8.20	

Loss of precision	

Division by large numbers may result in the result being zero, due to solidity not supporting fractions. Consider requiring a minimum amount for the numerator to ensure that it is always larger than the denominator	

uint256 _maxDebtPoint = (_ethMarketTotalDebt *debtRateAgainstEthMarket) / 1e18;	

Draft imports may break in new minor versions	

While OpenZeppelin draft contracts are safe to use and have been audited, their 'draft' status means that the EIPs they're based on are not finalized, and thus there may be breaking changes in even [minor releases](https://docs.openzeppelin.com/contracts/3.x/api/drafts). If a bug is found in this version of OpenZeppelin, and the version that you're forced to upgrade to has breaking changes in the new version, you'll encounter unnecessary delays in porting and testing replacement contracts. Ensure that you have extensive test coverage of this area so that differences can be automatically detected, and have a plan in place for how you would fully test a new version of these contracts if they do indeed change unexpectedly. Consider creating a forked version of the file rather than importing it from the package, and manually patch your fork as changes are made.	

5:    import {IERC20Permit} from "@openzeppelin/contracts/token/ERC20/extensions/draft-ERC20Permit.sol";


Signature use at deadlines should be allowed	

According to [EIP-2612](https://github.com/ethereum/EIPs/blob/71dc97318013bf2ac572ab63fab530ac9ef419ca/EIPS/eip-2612.md?plain=1#L58), signatures used on exactly the deadline timestamp are supposed to be allowed. While the signature may or may not be used for the exact EIP-2612 use case (transfer approvals), for consistency's sake, all deadlines should follow this semantic. If the timestamp is an expiration rather than a deadline, consider whether it makes more sense to include the expiration timestamp as a valid timestamp, as is done for deadlines.	

159:         if (participant.expiry < block.timestamp) { ,  require(aoTapOption.expiry > block.timestamp, "adb: Option expired");																							
																										
NFT doesn't handle hard forks	

When there are hard forks, users often have to go through [many hoops](https://twitter.com/elerium115/status/1558471934924431363) to ensure that they control ownership on every fork. Consider adding require(1 == chain.chainId), or the chain ID of whichever chain you prefer, to the functions below, or at least include the chain ID in the URI, so that there is no confusion about which chain is the owner of the NFT.	

function tokenURI( 69           uint256 _tokenId  70       ) public view override returns (string memory) {  71           return tokenURIs[_tokenId];	

tokenURI() does not follow EIP-721	

The [EIP](https://eips.ethereum.org/EIPS/eip-721) states that tokenURI() "Throws if _tokenId is not a valid NFT", which the code below does not do. If the NFT has not yet been minted, tokenURI() should revert	

68       function tokenURI(																							
																										
Open TODOs	

Code architecture, incentives, and error handling/reporting questions/issues should be resolved before deployment	

//    (TODO: Word better?) ,// TODO: Make whole function unchecked																							
																										
Calls to _get() will revert when totalSupply() returns zero	

totalSupply() being zero will result in a division by zero, causing the transaction to fail. The function should instead special-case this scenario, and avoid reverting.	

uint256 lpPrice = (SG_POOL.totalLiquidity() * 51:              uint256(UNDERLYING.latestAnswer())) / SG_POOL.totalSupply();																							
																										
latestAnswer() is deprecated	

Use latestRoundData() instead so that you can tell whether the answer is stale or not. The latestAnswer() function returns zero if it is unable to fetch data, which may be the case if ChainLink stops supporting this API. The API and its deprecation message no longer even appear on the ChainLink website, so it is dangerous to continue using it.	

uint256 _btcPrice = uint256(BTC_FEED.latestAnswer()) * 1e10;	


Use of a single-step ownership transfer	

The existing transferOwnership function immediately transfers ownership to the new address. Consider implementing a two-step variant, where the 'acceptor' of the ownership must call a separate function in order for the transfer to take effect. This can help to prevent mistakes where the wrong address is used, and ownership is irrecoverably lost.	

function transferOwnership(uint256 tokenId, address newOwner, bool direct, bool renounce) public onlyOwner(tokenId) {																							
																										
Missing checks for ecrecover() signature malleability	

ecrecover() accepts as valid, two versions of signatures, meaning an attacker can use the same signature twice, or an attacker may be able to front-run the original signer with the altered version of the signature, causing the signer's transaction to revert due to nonce reuse. Consider adding checks for signature malleability, or using OpenZeppelin's ECDSA library to perform the extra checks necessary in order to prevent malleability.	

206:             if (signer == ecrecover(digest, v, r, s)) {

																																																																																																																																						

																																																																				