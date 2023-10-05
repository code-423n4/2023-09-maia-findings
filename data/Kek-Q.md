# [L-1] [BranchBridgeAgent#L643](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L643) nouce not at correct offsets

Change `nonce = uint32(bytes4(_payload[22:26]));` to match other nonce values in this function: `nonce = uint32(bytes4(_payload[PARAMS_START_SIGNED:PARAMS_TKN_START_SIGNED]));`

PARAMS_START_SIGNED = 21
PARAMS_TKN_START_SIGNED = 25

Marking as low as I have not thought through or validated impact, perhaps a nonce collision is possible here?
Based on the surrounding code, I believe these values should be updated.



# [L-2] BranchPort.togglePortStrategy() does not validate a Port Strategy exists. Could be called to create a token without setting `_minimumReservesRatio` or adding to `strategyTokens` array

> BranchRouters as currently implemented do not do this, marking as low.

[BranchPort.togglePortStrategy()](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L396) has no validation to check if a Port Strategy has been created before. 


This would result in:
- (StrategyToken) `getMinimumTokenReserveRatio[_token]` returning 0 which affects `_minimumReserves()` @ L473 and violates the `MIN_RESERVE_RATIO = 3e3` state variable. 
- (PortStrategy) `strategyDailyLimitAmount` state variable is not set (defaults to 0)

Allows callling `manage()` to withdraw all tokens, leaving 0 in reserves. Will pass modifier check `requiresPortStrategy` @ L559.

> Also affects `replenishReserves` @ L188, `_reservesLacking()` returns current balance?  

This would also affect the public `strategyTokens` variable which would no longer output accurate results.

> It's worth noting a call to `addStrategyToken()` can resolve this issue, however, the protocol should have prevented this state from occuring.

## POC

[BranchPort.addPortStrategy()](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L382) has never been called for a given `_portStrategy` and `_token`

Calling [togglePortStrategy()](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L397) will set `isPortStrategy` true which will pass checks in other logic, however, `portStrategies` and `strategyDailyLimitAmount` state variables are never set like in `addPortStrategy()` @ [L388](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L388) and [L389](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L389) respectively.


## Remeidation

Validate inputs for [BranchPort.togglePortStrategy()](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchPort.sol#L396) to ensure a Port Strategy has previously been created through the `addPortStrategy()` function.

> Note, other toggle functions in Branch Port could also implement validation logic as well.



# [L-3] RootBridgeAgentFactory will maintain ability to add Agent to RootPort (until explicitly disbled by RootPort admin), anyone can call `createBridgeAgent`

> I did not have time to fully assess the impact of this, or if it is a problem. Below are some notes as I understand them which may be useful for further assessment.

A malicious user can create their own BranchAgent and add it to the `isBridgeAgent` mapping of an arbitrary RootPort by finding the associated Root Port Factory's address. 

[RootBridgeAgentFactory.createBridgeAgent()](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/factories/RootBridgeAgentFactory.sol#L48) (can be called by anyone) > [RootPort.addBridgeAgent()](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L382). A RootPort owner must call [RootPort.toggleBridgeAgentFactory()](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L431) to disable the factory, however, this call can be frontrun if done using a public mempool in a separate transaction as the creation of the RootPort itself. 

Functions with the `requiresBridgeAgent` modifier in RootPort that may be callable:

- [RootPort.burn() @ #L318](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L318) which does not validate `_srcChainId != localChainId`. I believe this is ok because there is a check in what I believe is the callstack before `burn()` is called @ [RootBridgeAgent._performCall() @ L821(https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L821). 

- [RootPort.toggleVirtualAccountApproved()](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L372) which would allow a RootPort to interact with an arbitrary VirtualAccount as the Virtual Account's access control @ [L161](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/VirtualAccount.sol#L161) in the `requiresApprovedCaller` queries the RootPort to check if the RootPort can access the VirtualAccount's functions.

Since the malicious users has deployed their own RootBridgeAgent which includes it's own BridgeAgentExecutor as well as deployed their own MultiCallRootRouter, they should be able to trigger function calls, specifically through the [RootBridgeAgent.lzreceive()](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L423)/[RootBridgeAgent.lzreceiveNonBlocking()](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L434) functions. However, calling functions is abstracted through LayerZero and I have no had a chance to dig deeper, I am not sure the range of functions the malicious user could actually do. 

It may be possible, for instance, to cause a DOS by frontrunning calls to (eventually) execute [RootPort.toggleVirtualAccountApproved()](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootPort.sol#L372) and cause other calls to fail such as deposits and withdraws.


## POC

For this test, we are just adding a new BridgeAgent that we control to an arbitrary RootPort.

Add test to: `/2023-09-maia/test/ulysses-omnichain/ArbitrumBranchTest.t.sol`

```javascript
function testAddBridgeAgentArbitrum_asUser(address _user) public {
    //Get some gas
    hevm.deal(address(this), 2 ether);
    hevm.deal(_user, 4 ether);
    hevm.startPrank(_user);

    // Create MulticallRootRouter
    MulticallRootRouter testMulticallRouter;
    testMulticallRouter = new MulticallRootRouter(
        rootChainId,
        address(rootPort),
        multicallAddress
    );

    // Create Bridge Agent 
    // @audit bridgeAgentFactory's `createBridgeAgent` is a public function with no access control
    // @audit Note that this adds the new BridgeAgent onto the rootPort that uses the Factory. (i.e.: we now have a BranchAgent of our control added to the isBridgeAgent mapping of an arbitrary RootPort)
    RootBridgeAgent testRootBridgeAgent = RootBridgeAgent(
        payable(RootBridgeAgentFactory(bridgeAgentFactory).createBridgeAgent(address(testMulticallRouter)))
    );

    //Initialize Router w/ new Bridge Agent
    testMulticallRouter.initialize(address(testRootBridgeAgent));

    //Create Branch Router in FTM
    BaseBranchRouter arbTestRouter = new BaseBranchRouter();

    //Allow new branch from root
    testRootBridgeAgent.approveBranchBridgeAgent(rootChainId);

    //Create Branch Bridge Agent
    rootCoreRouter.addBranchToBridgeAgent{value: 2 ether}(
        address(testRootBridgeAgent),
        address(localBranchBridgeAgentFactory),
        address(testMulticallRouter),
        address(this),
        rootChainId,
        [GasParams(6_000_000, 15 ether), GasParams(1_000_000, 0)]
    );


    console2.log("new branch bridge agent", localPortAddress.bridgeAgents(2));
    

    BranchBridgeAgent arbTestBranchBridgeAgent = BranchBridgeAgent(payable(localPortAddress.bridgeAgents(2)));

    arbTestRouter.initialize(address(arbTestBranchBridgeAgent));

    require(testRootBridgeAgent.getBranchBridgeAgent(rootChainId) == address(arbTestBranchBridgeAgent));

    hevm.stopPrank();
}
```

Run test with
```bash
forge test --match-test testAddBridgeAgentArbitrum_asUser
```

## Recommended Mitigation Steps

Consider adding access control to the [RootBridgeAgentFactory.createBridgeAgent()](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/factories/RootBridgeAgentFactory.sol#L48) function.


