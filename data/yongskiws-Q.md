


### Consider Relaying Messages to the Layer Zero Enpoint at the Gas Boundary

The impact is that when those fields are different between chains, one of two things may happen:

Less severe - we waste excess gas, which is refunded to the lzReceive() caller (Layer Zero)
More severe - we underprice the delivery cost, causing lzReceive() to revert and the NFT/ASSET stuck in limbo forever.
The code does not handle a failed lzReceive (differently to a failed executeJob).

Recommended Mitigation Steps
Firstly, make sure to use the target gas costs. i.e. save some gas to emit result event.

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L161-L173
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L161-L174
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L765-L778
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L785-L795
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L140-L153
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L820-L838
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L912-L930

```
src\interfaces\ILayerZeroEndpoint.sol
15:     function send(
16:         uint16 _dstChainId,
17:         bytes calldata _destination,
18:         bytes calldata _payload,
19:         address payable _refundAddress,
20:         address _zroPaymentAddress,
21:         bytes calldata _adapterParams
22:     ) external payable;
```



### RECOMMENDED TO ADD AN INPUT VALIDATION FOR THE CHECK VARIABLE IN THE CONSTRUCTOR

In the `ArbitrumBranchBridgeAgent`,`ArbitrumBranchPort`, `CoreRootRouter`, `RootBridgeAgent `,`VirtualAccount`. Constructor the `address&_rootChainId` is passed in as an input parameter. `address&_rootChainId`. Hence the `address&_rootChainId` variable is a critical variable of the protocol. If `address&_rootChainId` is set to zero by mistake it will prompt a redeployment of the contract which could be wastage of gas and additional work load. Hence an input validity check should be performed for the `address&_rootChainId` variable inside the constructor as follows:

```solidity

src\ArbitrumBranchBridgeAgent.sol
43:     constructor(
44:         uint16 _localChainId,
45:         address _rootBridgeAgentAddress,
46:         address _localRouterAddress,
47:         address _localPortAddress
48:     )

@>require(_localChainId > 0, "localChainId must be greater than zero");
@>require(isValidAddress(_rootBridgeAgentAddress), "Invalid rootBridgeAgentAddress");
@>require(isValidAddress(_localRouterAddress), "Invalid localRouterAddress");
@>require(isValidAddress(_localPortAddress), "Invalid localPortAddress");

57:     {}

src\ArbitrumBranchPort.sol
38:     constructor(uint16 _localChainId, address _rootPortAddress, address _owner) BranchPort(_owner) {
39:         require(_rootPortAddress != address(0), "Root Port Address cannot be 0");
40: 
@>require(_localChainId > 0, "localChainId must be greater than zero");
@>require(isValidAddress(_owner), "Invalid owner");
43:     }


src\CoreRootRouter.sol
71:     constructor(uint256 _rootChainId, address _rootPortAddress) {
@>require(_localChainId > 0, "localChainId must be greater than zero");
@>require(_rootPortAddress != address(0), "Root Port Address cannot be 0");    
72:         rootChainId = _rootChainId;
73:         rootPortAddress = _rootPortAddress;
74: 
77:     }

src\RootBridgeAgent.sol
105:     constructor(
106:         uint16 _localChainId,
107:         address _lzEndpointAddress,
108:         address _localPortAddress,
109:         address _localRouterAddress
110:     ) {
@>require(_localChainId > 0, "localChainId must be greater than zero");
111:         require(_lzEndpointAddress != address(0), "Layerzero Enpoint Address cannot be zero address");
112:         require(_localPortAddress != address(0), "Port Address cannot be zero address");
113:         require(_localRouterAddress != address(0), "Router Address cannot be zero address");

src\VirtualAccount.sol
35:     constructor(address _userAddress, address _localPortAddress) {
@>require(isValidAddress(_userAddress), "Invalid _userAddress");    
@>require(isValidAddress(_localPortAddress), "Invalid _localPortAddress");    
36:         userAddress = _userAddress;
37:         localPortAddress = _localPortAddress;
38:     }



function isValidAddress(address _addr) internal pure returns (bool) {
    return _addr != address(0);
}
```

### Add if necessary Emit event or add STATUS_PENDING to executionState


```solidity
src\BranchBridgeAgent.sol
730:   function _execute(bool _hasFallbackToggled, uint32 _settlementNonce, address _refundee, bytes memory _calldata)
731:         private
732:     {

 E.G @>       require(executionState[_settlementNonce] == STATUS_PENDING, "Transaction has already been executed");

733:         //Update tx state as executed
734:         executionState[_settlementNonce] = STATUS_DONE;
735: 
736:         //Try to execute the remote request
737:         (bool success,) = bridgeAgentExecutorAddress.call{value: address(this).balance}(_calldata);
738: 
739:         //Update tx state if execution failed
740:         if (!success) {
741:             //Read the fallback flag and perform the fallback call if necessary. If not, allow for retrying deposit.
742:             if (_hasFallbackToggled) {
743:                 // Update tx state as retrieve only
744:                 executionState[_settlementNonce] = STATUS_RETRIEVE;
745: 
746:                 // Perform fallback call
747:                 _performFallbackCall(payable(_refundee), _settlementNonce);

//Emit an event to notify that execution failed and fallback is enabled
  @>          emit ExecutionFailedWithFallback(_settlementNonce);

748:             } else {
749:                 // If no fallback is requested revert allowing for settlement retry.
750:                 revert ExecutionFailure();
751:             }
752:         }else
//Emit an event to notify that execution was successful
   @>         emit ExecutionSucceeded(_settlementNonce);
             }
753:     }


src\RootBridgeAgent.sol
768:     function _execute(
769:         bool _hasFallbackToggled,
770:         uint32 _depositNonce,
771:         address _refundee,
772:         bytes memory _calldata,
773:         uint16 _srcChainId
774:     ) private {
E.G // Make sure that the transaction has not been executed previously
@>    require(executionState[_srcChainId][_depositNonce] == STATUS_PENDING, "Transaction has been executed");
775:         //Update tx state as executed
776:         executionState[_srcChainId][_depositNonce] = STATUS_DONE;
777: 
778:         //Try to execute the remote request
779:         (bool success,) = bridgeAgentExecutorAddress.call{value: address(this).balance}(_calldata);
780: 
781:         //Update tx state if execution failed
782:         if (!success) {
783:             //Read the fallback flag.
784:             if (_hasFallbackToggled) {
785:                 // Update tx state as retrieve only
786:                 executionState[_srcChainId][_depositNonce] = STATUS_RETRIEVE;
787:                 // Perform the fallback call
788:                 _performFallbackCall(payable(_refundee), _depositNonce, _srcChainId);

                    //Emit an event to notify that execution failed and fallback is enabled
@>                    emit ExecutionFailedWithFallback(_depositNonce, _srcChainId);
789:             } else {
790:                 // No fallback is requested revert allowing for retry.
791:                 revert ExecutionFailure();
792:             }
793:         }else
        //Emit an event to notify that execution was successful
@>        emit ExecutionSucceeded(_depositNonce, _srcChainId);
            }
794:     }
795: 
```


### Typo in revert error `Branc` SHOULD BE `Branch`

```solidity
src\interfaces\IRootPort.sol
408:     error InvalidCoreBrancBridgeAgent();

src\RootPort.sol
545:         if (_coreBranchBridgeAgent == address(0)) revert InvalidCoreBrancBridgeAgent();
529:         if (_coreBranchBridgeAgent == address(0)) revert InvalidCoreBrancBridgeAgent();
```


### Typo in function _performCall `callee` SHOULD BE `calleer`

```solidity

src\RootBridgeAgent.sol
808:     function _performCall(
809:         uint16 _dstChainId,
810:         address payable _refundee,
811:         bytes memory _payload,
812:         GasParams calldata _gParams
813:     ) internal {
814:         // Get destination Branch Bridge Agent
815: @>         address callee = getBranchBridgeAgent[_dstChainId];
816: 
817:         // Check if valid destination
818: @>        if (callee == address(0)) revert UnrecognizedBridgeAgent();
819: 
820:         // Check if call to remote chain
821:         if (_dstChainId != localChainId) {
822:             //Sends message to Layerzero Enpoint
823:             ILayerZeroEndpoint(lzEndpointAddress).send{value: msg.value}(
824:                 _dstChainId,
825:                 getBranchBridgeAgentPath[_dstChainId],
826:                 _payload,
827:                 _refundee,
828:                 address(0),
829: @>                abi.encodePacked(uint16(2), _gParams.gasLimit, _gParams.remoteBranchExecutionGas, callee)
830:             );
831: 
832:             // Check if call to local chain
833:         } else {
834:             //Send Gas to Local Branch Bridge Agent
835: @>             callee.call{value: msg.value}("");
836:             //Execute locally
837: @>           IBranchBridgeAgent(callee).lzReceive(0, "", 0, _payload);
838:         }
839:     }
```