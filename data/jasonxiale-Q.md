# 1. `RootBridgeAgent.getFeeEstimate` and `BranchBridgeAgent.getFeeEstimate` doesn't comply with layerZero' Checklist
File:
```text
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L160-L173
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L140-L153
```
Quoting from [LayerZero Integration Checklist](https://layerzero.gitbook.io/docs/evm-guides/layerzero-integration-checklist):
> Do not hardcode useZro to false when estimating fees and sending messages. Pass it as a parameter instead.

# 2. BranchBridgeAgent._createDepositMultiple doesn't check if array.length is greater than zero
File: 
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L860C14-L888

[BranchBridgeAgent._createDepositMultiple](
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L860C14-L888) doesn't check if the parameter's length is greater than zero, if the input array's length is zero, gas will wasted.

# 3. `BranchPort.setCoreRouter` is never called.
File:
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L331-L335
Function [BranchPort.setCoreRouter](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L331-L335) requires the caller to be router, but the function is never called within current codebase

# 4. RootBridgeAgent._performFallbackCall should add if branch to handle local messages like `RootBridgeAgent._performCall` and `RootBridgeAgent._performRetrySettlementCall`
File: 
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L938-L948

Both [RootBridgeAgent._performCall](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L808-L839) and [RootBridgeAgent._performRetrySettlementCall](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L856C14-L931) can handle Root <==> ArbBrach messages without layerZero messaging, and [RootBridgeAgent._performFallbackCall](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L938-L947) missing this kind of code

# 5. VirtualAccount lacking of withdraw function for ERC1155
File:
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/VirtualAccount.sol#L51-L63
According to [VirtualAccount.sol#L17](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/VirtualAccount.sol#L17-L17), `VirtualAccount.sol` is able to receive ERC1155, but the contract doesn't an external function to withdraw ERC1155 like `VirtualAccount.withdrawERC20` and `VirtualAccount.withdrawERC721`

# 6. ETH might stuck in the contract
There are many functions in the code are defined as **payable**, especially for `ArbitrumCoreBranchRouter.sol`, and `ArbitrumBranchBridgeAgent.sol`. 
Normally the functions need to be defined as **payable** because the function need to pay the layerzero gas fee, but for `ArbitrumCoreBranchRouter.sol`, and `ArbitrumBranchBridgeAgent.sol`, calls will never be cross-chain, so those function need to pay the gas fee, thus those function need not to be **payable**.
Take [ArbitrumCoreBranchRouter.addLocalToken](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/ArbitrumCoreBranchRouter.sol#L51-L66) as an example, if users calls the the function with some ETH(because the user need to send ETH as gas fee for layerZero on other branch chains except ARB, and user doesn't know that), the ETH will stuck in the contract
```solidity
    function addLocalToken(address _underlyingAddress, GasParams calldata) external payable override {
        //Encode Data, no need to create local token since we are already in the global environment
        bytes memory params = abi.encode(
            _underlyingAddress,
            address(0),
            string.concat("Arbitrum Ulysses ", ERC20(_underlyingAddress).name()),
            string.concat("arb-u", ERC20(_underlyingAddress).symbol()),
            ERC20(_underlyingAddress).decimals()
        );

        // Pack FuncId
        bytes memory payload = abi.encodePacked(bytes1(0x02), params);

        //Send Cross-Chain request (System Response/Request)
        IBridgeAgent(localBridgeAgentAddress).callOutSystem(payable(msg.sender), payload, GasParams(0, 0));
    }
```

# block.chainid's type should be uint instead of uint16
Within the code, `localChainId` and `rootChainId` those variables' type is uint16, but according to [Block and Transaction Properties](https://docs.soliditylang.org/en/v0.8.0/units-and-global-variables.html#block-and-transaction-properties), block.chainid's type should be **uint**
> block.chainid (uint): current chain id