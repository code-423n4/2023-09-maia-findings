# [NC-1] zroPaymentAddress hardcoded as address(0) while calling LayerZero send function

## Summary 

The layerzero send function is used to send messages to another chain. One of the parameters passed is the zroPaymentAddress. It is advised in the layerzero integration checklist document that this must not be hardcoded to 0 and must be passed as a parameter instead. The impact due to this is that since these contracts are immutable, users would not be able to use the zroPaymentAddress even if they wanted to. 

## Affected SLOC

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L823-L830
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L915-L922
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L940-L947
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L770-L777
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L787-L794

## Reference 

https://layerzero.gitbook.io/docs/evm-guides/layerzero-integration-checklist

## Recommended Mitigation Steps

Pass zroPaymentAddress as a parameter

# [NC-2] useZro hardcoded to false while estimating gas 

## Summary

The layerzer estimateFees function is used to estimate how much fee it might take to send a particular payload through a particular path subject to certain parameters. It is advised in the layerzero integration checklist document that the useZro boolean must not be hardcoded to false while calling this function. However, it is hardcoded in the bridge agents. The impact due to this is that if fee options for the ZRO token in layerzero changes in the future, users might not be able to pay using ZRO for crosschain calls, even if they wanted to unless the contracts are upgraded. 

## Affected SLOC

https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L166-L172
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/RootBridgeAgent.sol#L146-L152

## Reference

https://layerzero.gitbook.io/docs/evm-guides/layerzero-integration-checklist

## Recommended Mitigation Steps

Do not hardcode useZro to false. Instead, pass it as a parameter. 

# [NC-3] Chain IDs are hardcoded in all contracts and cannot be changed

## Summary 

In all bridge agents, the chain IDs are hardcoded. However, in the layerzero integration checklist, it is recommended that there be a onlyAdmin function to set the chain IDs. 

## Affected SLOC

Both BranchBridgeAgent and RootBridgeAgent contracts have this issue. 

## Reference 

https://layerzero.gitbook.io/docs/evm-guides/layerzero-integration-checklist

## Recommended Mitigation Steps 

Have onlyAdmin functions to change chain IDs later on. 

# [NC-4] Bridge Agents do not check if airdrop caps are crossed in adapter parameters

## Summary 

Layerzero has airdrop caps for each chain. This must be checked in the Relayer.sol contract onchain so that the passed value does not cross the cap. However, this is not being done. The impact is that some messages may fail. 


## Affected SLOC


https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L776
https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L776

## Reference

https://layerzero.gitbook.io/docs/evm-guides/advanced/relayer-adapter-parameters#airdrop

## Recommended Mitigation Steps 

Check if the value sent is higher than the airdrop cap on the destination chain onchain via the Relayer.sol contract. 

# [NC-5] Wrong types assigned to addresses and chain IDs

## Summary 

In RootPort, chain IDs are given the type of address and some addresses are assigned as uint256. The affected SLOCs are these: 

```solidity

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

As we can see, chain IDs and localAddress, globalAddress, and underlyingAddress variables are given the wrong types. 

## Affected SLOC 

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L88C1-L102C1

## Recommended Mitigation Steps 

Change the types to correct ones. 


# [NC-6] Wrong comments at some SLOC

## Summary 

There are wrong comments at some SLOC: 

The variable is coreRootBridgeAgentAddress, but the comment is written as the core router: 

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootPort.sol#L42

Here, the approval is given to `_bridgeAgentAddress`, but the comment written is that approval is given to root port: 

https://github.com/code-423n4/2023-09-maia/blob/main/src/MulticallRootRouter.sol#L520

## Recommended Mitigation Steps

Change comments

