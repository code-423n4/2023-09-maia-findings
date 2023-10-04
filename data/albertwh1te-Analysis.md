## Overview of Components

1. **RootPort:**
   Responsible for holding all token information and receiving messages from various entities like `BridgeAgent`, `BridgeAgentFactory`, `LocalBranchPort`, and `CoreRootRouter`. It plays a central role in the graph.

   Functions:
   - `callOut`: Function to perform a call to the Root Omnichain Router without token deposit.
   - `callOutSigned`: Perform a call to the Root Omnichain Router without token deposit with msg.sender information.

2. **BranchBridgeAgent:**
   Acts as a `LayerZeroReceiver`, sending messages to `Lz` and utilizing `BranchBridgeAgentExecutor` for settlements.

3. **BranchBridgeAgentExecutor:**
   This contract handles token deposit clearance requests and executes transactions in response to requests from the root environment.

   Functions:
   - `executeWithSettlement`: Executes a single settlement.
   - `executeWithSettlementMultiple`: Executes multiple settlements.

   Settlement related logic is in `CoreBranchRouter`.

4. **CoreBranchRouter:**
   Contains user-related functions such as `addGlobalToken` and `addLocalToken`. It is called by `BranchBridgeAgentExecutor`. The `executeNoSettlement` function implements real logic, with related logic residing in `BranchPort`.

5. **BranchPort:**
   Branch Ports manage deposited balances into the system, while the Root Ports maintain accounting of every deposit in every Branch Port. It represents the last station of the protocol, focusing on token transfers and permission management.

6. **BranchBridgeAgentFactory:**
   In charge of creating `BridgeAgent` and is called by `CoreBranchRouter`.

7. **MulticallRootRouter:**
   Deals with money-related operations. If you intend to move user funds, this is where you should focus.

8. **VirtualAccount:**
   Checks `RootPort` for permissions and executes the required actions. Supports `nopayable` or `payable` actions.

## User Flow

- if it is a signed call, then the user needs to start the bridgeAgent directly
- If it is non-signed, then the user can choose between both bridgeAgent and BaseBranchRouter
- If it is a system/admin call, like adding a new token, the user needs to use the CoreBranchRouter

Here is mermaid sequence graph:

### System Call
```mermaid
sequenceDiagram
	box rgb(0, 100, 0)
	participant user as User
	participant BranchAgent as BranchBridgeAgent
	participant RootAgent as RootBridgeAgent
	participant RootAgentExecutor as RootBridgeAgentExecutor
	participant RootRouter as CoreRootRouter
	participant RootPort as RootPort
	end
	user->>BranchAgent: callOutSystem function
	BranchAgent->>RootAgent: ILayerZeroEndpoint.send -> lzReceiveNonBlocking
	RootAgent->>RootAgentExecutor: execute -> call executeSystemRequest 
	RootAgentExecutor->>RootRouter: executeResponse
	RootRouter->>RootPort: setAddresses/setLocalAddress/_syncBranchBridgeAgent
	RootPort->>RootPort: _addLocalToken/_setLocalToken
	RootPort->>RootAgent: syncBranchBridgeAgent
```

### Swap Interaction Flow

```mermaid
sequenceDiagram
	box rgb(0, 100, 0)
	participant user as User
	participant BranchAgent as BranchBridgeAgent
	participant RootAgent as RootBridgeAgent
	participant RootAgentExecutor as RootBridgeAgentExecutor
	participant RootPort as RootPort
	participant MulticallRootRouter as MulticallRootRouter
	end
	user->>BranchAgent: calloutsignedandbridge function payload=0x85
 	BranchAgent->>RootAgent: ILayerZeroEndpoint.send -> lzReceiveNonBlocking
	RootAgent->>RootPort: call toggleVirtualAccountApproved(Open)
	RootAgent->>RootAgentExecutor: execute -> call executeSignedWithDeposit
	RootAgentExecutor->> MulticallRootRouter : executeSignedDepositSingle
	RootAgentExecutor->>RootAgent: bridgeIn
	RootAgent->>RootPort: bridgeToRoot
	RootAgent->>RootPort: call toggleVirtualAccountApproved(Close)
```


### addGlobalToken

```mermaid
sequenceDiagram
	box rgb(0, 100, 0)
	participant user as User
	participant CoreBranchRouter as CoreBranchRouter
	participant BranchAgent as BranchBridgeAgent
	participant RootAgent as RootBridgeAgent
	participant lz as lzEndpoint
	participant RootAgentExecutor as RootBridgeAgentExecutor
	participant RootPort as RootPort
	participant CoreRootRouter
	participant TokenFactory
	end
	user->>CoreBranchRouter: addGlobalToken 
	CoreBranchRouter->>BranchAgent: callOut with payload=0x01
	BranchAgent->>lz: send with with payload=0x01
	lz->>RootAgent: lzReceiveNonBlocking with _payload=0x01
	RootAgent->>RootAgent:  executionState[_srcChainId][_depositNonce] = STATUS_DONE;
	RootAgent->>RootAgentExecutor :executeNoDeposit
	RootAgentExecutor->>CoreRootRouter: execute
	CoreRootRouter->>RootAgent: callOut
	RootAgent->> lz: send to dst chain branch payload=0x00
	lz->>BranchAgent: lzReceiveNonBlocking payload=0x00
	BranchAgent->>RootAgentExecutor: executeNoSettlement
	RootAgentExecutor->>CoreBranchRouter: executeNoSettlement with _params[0] == 0x01 (_receiveAddGlobalToken)
	CoreBranchRouter->> TokenFactory: createToken
	CoreBranchRouter->>BranchAgent: callOutSystem with payload=0x03
	BranchAgent->> lz: send  with payload=0x00
	lz->> RootAgent: lzReceiveNonBlocking payload=0x00 
	RootAgent->>RootAgentExecutor: executeSystemRequest with _payload=0x00
	RootAgentExecutor->> CoreBranchRouter:executeResponse  funcId == 0x03
	CoreBranchRouter->> RootPort: setLocalAddress
 
```
### addLocalToken

```mermaid
sequenceDiagram
	box rgb(0, 100, 0)
	participant user as User
	participant CoreBranchRouter as CoreBranchRouter
	participant BranchAgent as BranchBridgeAgent
	participant RootAgent as RootBridgeAgent
	participant lz as lzEndpoint
	participant RootAgentExecutor as RootBridgeAgentExecutor
	participant CoreRootRouter
	participant RootPort as RootPort
	end
	user->>CoreBranchRouter: addLocalToken
	CoreBranchRouter->>BranchAgent: callOutSystem payload=0x02
	BranchAgent->>lz: send payload=0x00
	lz->>RootAgent: lzReceiveNonBlocking 
	RootAgent->>RootAgentExecutor: executeSystemRequest
	RootAgentExecutor->>CoreRootRouter: executeResponse FUNC ID 0x02
	CoreRootRouter->>RootPort: setAddresses
```

### Time spent:
20 hours