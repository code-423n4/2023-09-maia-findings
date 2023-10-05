# Introduction

Since I took part in the previous Maia DAO contest at Code4rena, I'm somehow familiar with the changes which had been made. The developers decided to switch from `anyCall` to `LayerZero`.
As using `LayerZero` is pretty new concept which was not observed in the previous Maia DAO code-base – this Analysis Report is focused only on that code related to `LayerZero` integration.

# Approach Taken

The most of the Code4rena contests are being performed in a security review approach. During the given time-frame, the code is being analyzed to identify as many vulnerabilities and security issues as possible.
The web3 security review is somehow related to web2 penetration test and/or vulnerability assessment – its goal is to identify as many issues as possible and report them back to the developers. During the contest – I took the same approach, which allowed me to identify multiple of Medium and Low risk issues.
However, for this Analysis Report, I’ve decide to take a different approach – an audit.
An audit, is a sort of checklist, which allows to verify if previously defined security assumptions are fulfilled. I decided to use two resources which allowed me to prepare such a checklist – and verified the `LayerZero` integration. Those are:

* [LayerZero Integration Checklist](https://layerzero.gitbook.io/docs/evm-guides/layerzero-integration-checklist) 
* [Smart Contract Security Verification Standard - I4 - Cross-Chain](https://github.com/ComposableSecurity/SCSVS/blob/master/2.0/0x300-Integrations/0x304-I4-Cross-Chain.md)


# LayerZero Integration Checklist

* `Use the latest version of solidity-examples package. Do not copy contracts from LayerZero repositories directly to your project.`

Protocol implements own non-blocking mechanism. It does not copy-paste code directly from LayerZero repository.

* `If your project requires token bridging inherit your token from OFT or ONFT. For new tokens use OFT or ONFT, for bridging existing tokens use ProxyOFT or ProxyONFT.`

OFT is Omnichain Fungible Token. Tokens implementation do not inherit from OFT. Developers chose their own implementation - with less extra features that don't seem to be desired. 
Those tokens (`ERC20hTokenRoot`, `ERC20hTokenBranch`) implement their own `burn()` and `mint()` functions. The rest is inherited from ERC20.

* `For bridging only between EVM chains use OFT and for bridging between EVM and non EVM chains (e.g., Aptos) use OFTV2.`

Protocol does not use LayerZero's Omnichain Fungible Token implementation.

* `Do not hardcode address zero (address(0)) as zroPaymentAddress when estimating fees and sending messages. Pass it as a parameter instead.`

In every `ILayerZeroEndpoint(lzEndpointAddress).send()` call, `zroPaymentAddress` is set to `_refundee`, which is passed by the caller.

* `Do not hardcode useZro to false when estimating fees and sending messages. Pass it as a parameter instead.`

`useZro` is hardcoded to `false` in `getFeeEstimate()` function. This issue affects both `BranchBridgeAgent.sol` and `RootBridgeAgent.sol`. 
When user calls `getFeeEstimate` to get the estimate fee of the message - the call to `ILayerZeroEndpoint(lzEndpointAddress).estimateFees` is being executed. The parameter `useZro` in that call is hardcoded to false.

* `Do not hardcode zero bytes (bytes(0)) as adapterParamers. Pass them as a parameter instead.`

`adapterParamers` is the last parameter in the `ILayerZeroEndpoint(lzEndpointAddress).send`. In both `BranchBridgeAgent.sol` and `RootBridgeAgent.sol` - in the last parameter of `ILayerZeroEndpoint(lzEndpointAddress).send`, we pass `abi.encodePacked` data. 
Only the `_performFallbackCall` function passes empty string: `""`, however, since it's a fallback function - it is expected behavior.

* `Make sure to test the amount of gas required for the execution on the destination. Use custom adapter parameters and specify minimum destination gas for each cross-chain path when the default amount of gas (200,000) is not enough. 
This requires whoever calls the send function to provide the adapter params with a destination gas >= amount set in the minDstGasLookup for that chain. So that your users don't run into failed messages on the destination. It makes it a smoother end-to-end experience for all.`

Function uses `excessivelySafeCall` to pass the amount of gas to be used:

```
(bool success,) = address(this).excessivelySafeCall(
            gasleft(),
            150,
            abi.encodeWithSelector(this.lzReceiveNonBlocking.selector, msg.sender, _srcChainId, _srcAddress, _payload)
        );
```

It sets that parameter to `gasleft()`, which implies, that it will be enough gas to execute. However, using `gasleft()` may have other security consequences, which were reported as separate issue.

* `Do not add requires statements that repeat existing checks in the parent contracts. For example, lzReceive function in LzApp contract checks that the message sender is LayerZero endpoint and the scrAddress is a trusted remote, do not perform the same checks in nonblockingLzReceive.`

Protocol does not derive from `LzApp`/`NonblockingLzApp`, thus it does not repeat any `require` checks.

* `If your contract derives from LzApp, do not call  lzEndpoint.send directly, use _lzSend.`

Protocol does not derive from `LzApp`/`NonblockingLzApp`, thus it uses `ILayerZeroEndpoint(lzEndpointAddress).send{value: _value}` directly

* `For ONFTs that allow minting a range of tokens on each chain, make the variables that specify the range  (e.g. startMintId and endMintId) immutable.`

Protocol does not use LayerZero's Omnichain Non Fungible Token implementation.

# Smart Contract Security Verification Standard - I4 - Cross-Chain

| # | # |
| --- | --- |
| **I4.1** |  LayerZero Integration Checklist has been verified paragraph above |
| **I4.2** | According to [LayerZero docs](https://layerzero.gitbook.io/docs/technical-reference/testnet/default-config#default-block-confirmation): `To simplify writing a User Application contract, LayerZero does not require any configuration. In this case, a UA sending messages will be using the system defaults.` |
| **I4.3** | Protocol uses own implementation of tokens and non-blocking mechanism | 
| **I4.4** | See (I4.4) for more detailed explaination | 
| **I4.5** | Protocol implements `requiresEndpoint` modifier which behaves like LayerZero Trusted Remote | 
| **I4.6** | Token does not use OpenZeppelin's ERC20. It uses Solmate implementation instead | 


| # | # |
| --- | --- |
| **I4.LZ.1** | Protocol does not use LayerZero's Omnichain  Fungible Token implementation |
| **I4.LZ.2** | Protocol does not use LayerZero's Omnichain Non Fungible Token implementation |
| **I4.LZ.3** | Protocol does not use LayerZero's Omnichain Fungible Token implementation. There's no `_debitFrom` implemented. |
| **I4.LZ.4** | According to [LayerZero docs](https://layerzero.gitbook.io/docs/technical-reference/testnet/default-config#default-block-confirmation): `To simplify writing a User Application contract, LayerZero does not require any configuration. In this case, a UA sending messages will be using the system defaults.` |
| **I4.LZ.5** | Protocol implements its own non-blocking mechanism, which allows to retry failed messages |
| **I4.LZ.6** | Having `forceResume` would require having administrative rights. However, Bridge Agents do not have function with that rights, e.g. like owner EOA. To avoid being blocked, a non-blocking implementation is being used.  |
| **I4.LZ.7** | Check  LayerZero Integration Checklist results (paragraph above) for reference |
| **I4.LZ.8** | UA does not inherit from `LzApp`/`NonblockingLzApp` |
| **I4.LZ.9** | According to [LayerZero docs](https://layerzero.gitbook.io/docs/technical-reference/testnet/default-config#default-block-confirmation): `To simplify writing a User Application contract, LayerZero does not require any configuration. In this case, a UA sending messages will be using the system defaults.`  |
| **I4.LZ.10** | Protocol implements its own non-blocking mechanism and tokens |
| **I4.LZ.11** | Protocol implements its own non-blocking mechanism and tokens |
| **I4.LZ.12** | Protocol does not use LayerZero's Omnichain  Fungible Token implementation |
| **I4.LZ.13** | Protocol does not derives from `LzApp`/`NonblockingLzApp`, thus it uses `ILayerZeroEndpoint(lzEndpointAddress).send{value: _value}` directly. |
| **I4.LZ.14** | Protocol does not derive from `LzApp`/`NonblockingLzApp`, thus it does not repeat any `require` checks. |


* (I4.4)
Dynamic parameters are passed to `abi.encodePacked` in a way, which does not provoke any collisions. There's either no dynamic type or only one dynamic type passed to `abi.encodePacked`.
The only part of the code where `abi.encodePacked` encodes dynamic datatypes which are close to each other (thus can lead to collison) are in `BranchBridgeAgent.sol`, where `deposit.hTokens`, `deposit.tokens`, `deposit.amounts`, `deposit.deposits` are being `abi.encodePacked` together.

However, protocol protects from collision in that case, because it verifies the length of those data:

```
 if (_hTokens.length > MAX_TOKENS_LENGTH) revert InvalidInput();
        if (_hTokens.length != _tokens.length) revert InvalidInput();
        if (_tokens.length != _amounts.length) revert InvalidInput();
        if (_amounts.length != _deposits.length) revert InvalidInput();

```

Since `deposit.hTokens`, `deposit.tokens`, `deposit.amounts`, `deposit.deposits` need to have the same length - they cannot create a collision in `abi.encodePacked`.


# Conclusion

LayerZero integration passed most of the requirements from the above checklist. The elements which were not passed do not pose any security risk. This implies that LayerZero has been integrated correctly.

### Time spent:
20 hours