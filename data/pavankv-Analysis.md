

Analysis Report :-

**Index of table**
| Sl.no  | Particulars  |
|---|---|
| 1   | Overview  |
| 2  | Architecture view(Diagram)  |
| 3  | Approach taken in evaluating the codebase  |
| 4  | Centralization risks  |
| 5 | Mechanism review  |
| 6  | Hours spend   |
|   |   |

## OverView 
Ulysses Protocol is one of the ecosystem or protocol of MAIA DAO V2 . It is designed to focus on capital efficiency and trying to lower cost of renting liquidity. By enabling users to interoperate of message between cross chainmessaging between cross-chains to conducting various economic activities such as providing liquidity to other chains , deposits , withdrawal and swaps tokens much more.

Ulysses has two main composts
1. Virtulized Liquidity 
This is old mechanism used by international businessmens to aggregate their liquidity from bank accounts in different countries into a single virtual pool. This would allow them to have faster and more efficient access to cash to fund its operations around the world. The same concept used ulysses by underlying assets can be remain deposit/lock in thier desired chains while still the can provide liquidity and amke some yeild farming this increases capital efficiency and reduce the operating cost .

2. Arbitrary Cross-Chain Execution
It means ulyssess provide their user to execute any swaps , deposits , withdrawal and system request between branch chain ,root chain and Arbitrum chain deployment. It's help attract the more users helps to achieve organisation goals. User can execute multi calls with cross-chains and having virtual account was given more prioritize and provided opportunities to delpoy factories and it's strategies.   




## Architecture view(Diagram) :-

![Diagram Of archeticeture](https://user-images.githubusercontent.com/69415766/272919160-52c6e7de-8dd1-400f-bad9-95842753befc.png)

Note :- It's free diagram writing software it has limited access for free verion that's why i'm able add line between branchbridgeagent and branchport. 


Link to view full Diagram
https://lucid.app/lucidchart/29b056d8-c159-48ab-bbcd-b76940327f09/edit?page=0_0&invitationId=inv_c475bc44-edbf-418e-a9de-6ed4085e8fb2#




## Approach taken in evaluating the codebase

We approached manual code review by looking into each space in the scoped contract and decided to explain some core functional contracts.it is important to note that manual code reviews can be very time-consuming.

**BranchChain**

The components of branch chains are
1. BranchBridgeAgent.sol 
2. BranchBridgeAgentExecutor.sol
3. BranchPort.sol
4. BaseBranchRouter.sol



1.BranchBridgeAgent.sol 

The state varibles are :-
`rootBridgeAgentPath` used to store the path  for root bridge agent address where cross-chain request are sent to the root chain router.
`depositNonce` used to store deposit nonce . Each deposit will increment the deposit nonce by 1 .
`getDeposit` it is mapping data type used to get specific deposit input struct for corresponding deposit nonce.
`executionState` it is mapping of execution state stores the settleemnt nonce and it's state (eg. statusDone, statusRetrive and others).
`_unlocked` it is used to re-entrancy check.

core function are :-

`callOut()` 
This function to perform a call to the root omnichain router without token deposits.It calls the internal function `performCall()` to call layer zero endpoint for cross-chain messaging with arguments of _refundee , payload bytes data type and gasparams. In `performCall()` function call `send()` of layer zero messaging layer with arguments of rootchainId, rootAgentPath, payload, refundee and abi encode gparams.

`callOutSystem()`
This functionis used perform call to cross-chain messaging with help of `performCall()` function.

`callOutAndBridge() and callOutAndBridgeMultiple()`
The above two functions performs calls to the Root omnichain while depositing assets without msg.sender information .

`callOutSigned() , callOutSignedAndBridge() and callOutSignedAndBridgeMultiple()` 
The above functions to perform a call to the root Omnichain router while depositing one or more assets with msg.sender information .

`retryDeposit()`
The above function is used to call root omnichain to retry a failed deposit that has not been executed yet.

`retrieveDeposit()`
The above function is used to request token back to branch chain after failing of omnichain interaction with arguments of deposit nonce and gasparam struct.

`redeemDeposit()`
The above function is used to retry a failed deposit entry on this branch chain with arguments depositNonce.

`retrySettlement()` 
The above function is used to retry a request of failed settlement on the root chain with agruments of settlementNonce , params of bytes data types ,gas param struct and boolean data type.

`ClearTokens()` and `clearToken()`
The above function is used to request balance clearance from a port after swaps tokens with cross-chain.

`lzReceiveNonBlocking()`
The above function plays vital role in cross-chain mechanism layer by calling endpoint arguments without blocking .

 
2.BranchBridgeAgentExecutor.sol

core functions are :-

`executeNoSettlement()` 
The above is used to call `executeNoSettlement()` of router without any settlement the function name only specifies that, with arguments of address of router and bytes data type payload.

`executeWithSettlement()` and `executeWithSettlementMutliple()`
The above functions is used to call cross-chain request  with one time or more settlement with arguments address of recipient , address of router and bytes data type payload .

3. Branch Port

State varibles are :-
`coreBranchRouterAddress` :-
Used to store the branch router address

`isBridgeAgent`:-
It is the mapping data type stores the each bridge agent is active or not status.

`isBridgeAgentFactory`:-
It is the mapping data type stores the each bridge agent factory is active or not status.

`bridgeAgentFactories`:-
It is the dynamic array stores the addresses of the bridge agent factories.

`bridgeAgent`:-
It is the dynamic array stores the addresses of the bridge agent.

`isStrategyToken`:-
It is the mapping stores the whether the each strategy is true or false.

`strategyTokens`:-
It is the dynamic array stores the addresses of the strategy tokens deployed.

`getStrategyTokenDebt`:-
It is a mapping data type stores debt of the each each strategy tokens.

`getMinimumTokenReserveRatio`
It is mapping data type stores the minimum reserve of each strategy tokens to be maintained .

`isPortStrategy`
It is mapping data type returns boolean of each port strategy addresses .

`portStrategies`
It is dynamic array consist of address of port strategies .

`getPortStrategyTokenDebt`
It is mapping data type stores debt of each port strategies.

`lastManaged`
It is mapping data type stores last time of the each port strategy.

`strategyDailyLimitAmount`
It is the mapping data type stores the daily limit of each strategy and it's token.

`strategyDailyLimitRemaining`
It is the mapping data type stores the daily limit remaining of each strategy and it's token .

`_unlocked`
Used for re-entrancy.

Core functions :-

`manage()`
The above functions used to withdraw assets by increasing strategy token debt and port strategy token debt by specified amount .

`replenishReserves()`
The above function is used to refill the borrowed reserves with reserve only , by decreasing the strategy token debt and port strategy token debt by specified amount .

`replenishReserves()` 
is used to refill the borrowed reserves and refill the token's reserves also.

`withdraw()`
It is used to withdraw underlying/native tokens in exchange of htoken/ local tokens.

`bridgeIn()` and `bridgeInMultiple()`
The above functions is used to increase the ltokens supply by depositing the ltokens and withdrawing the underlying tokens.

`bridgeOut()` and `bridgeOutMultiple()`
The above functions is used to decreasing the supply of htokens/local tokens by burn the specified htokens and safe transfer of underlying tokens.

4.BaseBranchRouter.sol

state variable are :-

`localPortAddress`
It is used to store the local branch port address .

`localBridgeAgentAddress`
It is used to store  the local bridge agent address .

`bridgeAgentExecutorAddress` 
It is used to store the local bridge agent executor .

Core functions are:-

`callOut()`
The above used to request cross chain messaging without deposit using local bridge agent callout() function .

`calloutAndBridge()` and `callOutAndBridgeMultiple()`
The above functions is used to send requesting cross chain with single or more deposits using local bridge agent calloutAndBridge() function and calloutAndBridgeMultiple() function.

`executeNoSettlement()`
The above function is used call cross-chain request without settlement but in branch router it declared as revert when it called .

`executeWithSettlement()` and `executeWithSettlementMultiple`
The above is used to call cross-chain to settlement once or more but in branch router declared as revert whenever it called.

`_transferAndApproveToken()` and `_transferAndApproveMultipleTokens()`
The above functions is used to transfer tokens to router and approve for local port to safetransferfrom.



**RootChain**

The components of root chain are :-
1. RootBridgeAgent.sol
2. RootBridgeAgentExecutor.sol
3. RootPort.sol
4. CoreRootRouter.sol

1. RootBridgeAgent.sol

state variable are :-
`getBranchBridgeAgent`
This is mapping data type used to store branch bridge agent address of each chain.

`getBranchBridgeAgentPath`
This is mapping data type used to branch bridge agent path for each chain.

`isBranchBridgeAgentAllowed`
It is used to store the whether branch bridge agent is allowed to add/syn new branch briage agent .

`settlementNonce`
It is used to store the nonce of settle ment it will increase by 1 in each settlement

`getSettlement`
It is mapping data type used to store each nonce  corresponding settlement info struct .


Core functions are :-

`callOut()`
It is function used to cross-chain request with the endpoint of Layer zero with calling performCall().

`callOutAndBridge()` and `callOutAndBridgeMultiple()`
This functions allows to move assets from root chain to branch chain and allowing redeeming the assets if settlement fails for both smart contract and virtual account also .

`retrySettlement()`
It is used to retry a user's balance settlement 

`redeemSettlement()`
This function allows redemption of failed settlement's global tokens.

`bridgeIn()` and `bridgeInMultiple()`
This function is used to move assets from branch chain to root chain by using localport.BridgeToRoot() function .

`lzReceiveNonBlocking()`
The above function plays vital role in cross-chain mechanism layer by calling endpoint arguments without blocking .

2. RootBridgeAgentExecutor :-

Core functions are :-

`executeSystemRequest()`
This function is used to call system request from a remote chain.

`executeNoDeposit()`
This function is used to request from remote chain without deposit .

`executeWithDeposit()` and `executeWithDepositMultiple()`
The above functions have some unique slicing method of value in struct these functions is used to execute request from remote chain. 

`_bridgeIn()` and `_bridgeInMultiple()`
The above functions used to move assets from branch to root omni chain environment .


3 .RootPort.sol

State variable are:-

`coreBranchRouterAddress` :-
Used to store the branch router address

`isBridgeAgent`:-
It is the mapping data type stores the each bridge agent is active or not status.

`isBridgeAgentFactory`:-
It is the mapping data type stores the each bridge agent factory is active or not status.

`bridgeAgentFactories`:-
It is the dynamic array stores the addresses of the bridge agent factories.

`bridgeAgent`:-
It is the dynamic array stores the addresses of the bridge agent.

`isStrategyToken`:-
It is the mapping stores the whether the each strategy is true or false.

`strategyTokens`:-
It is the dynamic array stores the addresses of the strategy tokens deployed.

`getStrategyTokenDebt`:-
It is a mapping data type stores debt of the each each strategy tokens.

`getMinimumTokenReserveRatio`
It is mapping data type stores the minimum reserve of each strategy tokens to be maintained .

`isPortStrategy`
It is mapping data type returns boolean of each port strategy addresses .

`portStrategies`
It is dynamic array consist of address of port strategies .

`getPortStrategyTokenDebt`
It is mapping data type stores debt of each port strategies.

`lastManaged`
It is mapping data type stores last time of the each port strategy.

`strategyDailyLimitAmount`
It is the mapping data type stores the daily limit of each strategy and it's token.

`strategyDailyLimitRemaining`
It is the mapping data type stores the daily limit remaining of each strategy and it's token .

`getBridgeAgentManager`
It is used to store the address of each bridge agent factory and it's manager.

`isGlobalAddress`
It is used to store the each htokens deployed globally.

`getGlobalTokenFromLocal`
It is used to store the address of each chain and it's local and global address

`getLocalTokenFromGlobal`
It is used to store the address of each chain and it's global and local address

`getLocalTokenFromUnderlying`
It is used to store each chain address and it's underlying and local tokens addresses.

`getUnderlyingTokenFromLocal`
It is used to store each chain address and it's local and underlying addresses .

Core function are :-

`bridgeToRootFromLocalBranch()`
This function is used to move assets from local branch to root port.

`bridgeToLocalBranchFromRoot()`
This function is used to move assets from root port to local branch port

`mintToLocalBranch()`
This function is used to mint new local tokens and send to recipeint address .

`burnFromLocalBranch()`
This function is used to burn local tokens of depositer address.

`bridgeToRoot()`
This function used to update port state to match new deposit.

 4. CoreRootRouter.sol
 
 State variable are :-
 `bridgeAgentAddress`
 It used to store branch bridge agent address
 
 `bridgeAgentExecutorAddress`
 It is used to store branch bridge agent executor address.
 
 `hTokenFactoryAddress`
 It is used to store root factory address.
 
 Core functions are :-

`executeResponse()` 
This function is used to execute request from brnach bridge agent with no assets deposits.

`execute()`
This function is used to execute the cross-chain request without deposit 

`executeDepositSingle()` and `executeDepositMultiple()` 
The above functions is used to execute cross-chain request from bridge agent with one or more deposits . 

`executeSigned()`
This function is used to execute cross-chain request which contains information of msg.sender and witout deposits.

`executeSignedDepositSingle()` and `executeSignedDepositSingle()`
This function is used to executing a cross chain request with msg.sender information and depositing one or more assets.


## Centralization risks
The function which have onlyOwner() modifier can be land in danger because private key compromise of user. Other than that ther is no centralisation risk here because it is interaction between the cross-chains by swaps , deposits , withdrawal and sending requests. So here users want to follow particular policies of cross-chain do not mis understand as centralisation risk of authority of chains. 


## Mechanism review

In this anaylsis report we tried eloborate the mechanism of the ulysses protocol .The components of the mechanism are

1.`Root chain`
Root chain act as center for branch chains . It has responsibility to maintain it's components like RootPort , RootBridgeAgent and RooTRouters . In root chain interaction between it's components is well designed in Ulysses protocol . 

`Root port` :-
Root port act as vault in root chain where it accepts deposits and withdraws of local tokens and global tokens . If user want to send deposit native tokens they can request Root router and it passes request to Root port it will make some accounting and issues local Htokens. And root port contains all mappings of Local to global address and vise-versa in simple words we can say registry of book . Mint function , burn function , addNew chain functions , setters and getters function so on . 

`Root Routers` :- 
Root routers in root chain act as intermediatery between root bridge agents and Root port it forwards request/response of swaps flow requested by user or cross-chains . Root routers instructs the root bridge agents to transfer output tokens to desired destination chain . It contains core function like  bridge agents management function like adding new bridge aggents . Strategy functions like add/remove strategy token and port strategy . Execute function to request/response to user swaps and instructs the root bridge agents 

`Root BridgeAggent ` :-
Root bridge agents is in root chain and it is connceted with all branch chain and it's bridge agents and also local chain means root routers in-order to make some cross-chain request/response by user maintaing assets through cross-chains. It contains core functions like Layer zero to send cross-chain request/response it's mainly used for swaps with cross-chain tokens and callout's function to deposit tokens in local port and cross-chain ports and settlement function to clear the tokens by deposits local htokens and transfering global/underlying tokens with associated branch chain request/response . Bridge In and Bridge out function to move htokens from branch chain to root chain port deposits and vise-versa .


2. `Branch chain` 
Branch chain  is a type chain which contains components like root chains but it cannot act as root chain means if brach chain want to swap tokens with other branch chain it cannot directly call. Then root chain come into play major in between branch chains . It's components like BranchBridgeAgent , BranchRouter, BranchPort and BranchBridgeAgentExecutor . 

`BranchBridgeAgent` :-
BranchBridgeAgent is a one of the component of branch chain which act as middle man between routers and users who desired to request/resposnse to cross chains. BranchBridgeagent executes the request and response of users with help of layer Zero function which perform calls of destination chain argument provided by users . If a branchbridgeagent received the request then it passes to branchBridgeAgent executor and flow to router and port of local branch. The core functions are callout's which perform calls to layer zero endpoint for cross chain messages and deposits with single mulitple tokens in Omnichain or Local branch port . Deposit function to retry deposit , retrive depsoits and 
reddem deposits . Token management functions like clear tokens balance clearance from port, when user swaps tokens from local port to cross chain port then this function will invoke .

`BranchBridgeaAgentExecutor` :- 
BranchBridgeAgentExecutor act as intermediary between Branch routers and Branch Bridge Agent. It passes function request to  branch routers from branchbridgeagent and vise-versa . The core function are  executor fumction like settlement and clear tokens of cross-chain request

`BranchRouter` :-
BranchRouters plays vital role in between users - Port and BridgeAgent - BranchPort it is user-facing entry and exit points . Branch router passes the request/repsonse of both users and branchbrigde agents to local port about deposits and help to perform call to layer zero through branchbridgeagents . The core function are bridge agent external functions which means recieves request from bridge agent about to settlement of swaped tokens from local port to cross chain port .Callout functions used to deposit tokens in local port . 

`BranchPort` :-
Branch port act as vault system accepts deposits , withdrawal and swaps from user and bridge agents request/response . It mananges factories, strategies and it's tokens . The core functions are port strategy management to withdraw assets , repay borrowed reserves and hToken management to  withdraw underlying/native token in exchange of local tokens , increasing and decreasing local htokens supply and admin functions.


3.`Arbitrum deployment chain`

i. **ArbitrumBranchBridgeAgent.sol**
This contract act as a middle man betwwen users and routers to access cross-chain messagingand port interacting for asset management connecting arbitrum branch chain contracts and the root omnichain environment. The core function are :-

`depositToPort()` :-
This function is used deposit underlying assets in arbitrum port . First it calls to `port.depositToPort()` with arguments of despositor , recipeint who receives local htokens , address of underlying assets and amount of assets to be desposit.

`withdrawFromPort()` :-
This function is used to withdraw underlying/native tokens in exchange of local htokens . It call `port.withdrawToPort` with arguments of depositer , recipeint who recieves the underlying tokens and amount to be withdraw form port .

`_performCall()` 
This function call layer zero endpoint for cross-chain messaging first it will send gas to root bridge agent  and calls `lzReceive()` with arguments of rootChainId and payload of bytes data type .

` _performFallbackCall()`
This function performs call to layerzero endpoint contract for cross chain messaging . It calls `rootBridgeAgent.lzReceive()` with arguments rootChainId and settlement nonce .


ii. **ArbitrumCoreBranchRouter.sol**
This function is used for permissionlessly adding new tokens or bridge agents and governance .

The core functions are :-
`addLocalToken()` 
This function is used for deploy/add token already in the global environment to a branch chain. First it encode with arguments underlying decimals and concatenation of strings stored to bytes data type and again encode.packed and stored bytes data type and calls callsoutsystem() function of local bridge agent to perform call to layer zero endpoint for cross-chain messaging with arguments of address of gasRefundee , params of byte data type and gasparams struct.

`_receiveAddBridgeAgent()`
This is internal function is called by `executeNoSettlement()` function it is used to deploy/add a token to active the global environment in the root chain.First it have validation methods to check `msg.sender` is a valid BridgeAgentFActory then it creates the new bridgeagnet then check whether the new bridge agent address is valid or not and finally calls localBridgeAgent callsoutsystem() function of local bridge agent to perform call to layer zero endpoint for cross-chain messaging with arguments of address of gasRefundee , params of byte data type and gasparams struct.

`executeNosettlement()` 
This function is used to execute cross-chain request without any settlement . The check mechanism check the zero index is equal to 2 and call `_recieveAddBridgeAgent` internal function. Second check if data zero index is equal to 3 then calls `_toggleBranchBridgeAgentFactory` function to turn off/on the bridgeagent address .Third checks if data zero indexed is 4 then call the `_removeBranchBridgeAgent()` with bridge agent address  to remove from that position . Fourth check the data index is equal to 5 then calls `_manageStrategyToken()` function to add/remove a token used by port strategies. Fifth check whether the data zero indedx is 6 then calls `_managePortStrategy()` function used to add/deploy token already in global environment in the root chain.Lastly if argument data zero index doesn't satisfy the above 5 checks then it will revert.

iii. **ArbitrumBranchPort.sol**
This contract is used for port implementation for arbitrum branch chain deployment 

The core function are :-
`depositToPort()`
This function which called by bridgeAgent with arguments depositer, recepient , underlyingaddress and amount of deposit first it cache root port address and get global token address with validation.Then it calls safeTransferFrom() to deposit native/underlying tokens and sends corresponded htokens minted .

`withdrawFromPort()`
This function which is called by bridgeAgent with arguments depositer, recipient , globalAddress and amount to withdraw .It cache the rootPortAddress and validation of address of underlying token in specific chain , calls the burn() function with mentioned amount and transfer the corresponding underlying tokens to recipient address.

`_bridgeIn()`
This function is used to bridge in the assets from root chain , calls the `rootPort.bridgeToLocalBranchFromRoot()` function with arguments recipient address local tokens and amount of tokens to be transfer.

`bridgeOut()`
This function is used to bridge out the assets to root chain , here interesting mechanism come into play first it deposits underlying assets in arbitrumBranchPort then it calls the `rootPort.bridgeToRootFromLocalBranch()` function with  corresponding htoken/local tokens to be transfer.


## Hours spend 

67 hours .

Note :-
It is primary data.





### Time spent:
67 hours