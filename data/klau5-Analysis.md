# Analysis of the code base

## Branch vs Root

Root chain is the virtual chain that manages the whole bridging service. Branch is EVM-based chain that uses the bridging service.

Root is operated at Arbitrum chain, but ‘Root chain’ doesn’t mean Arbitrum chain. Arbitrum chain also treats as one of its branches and is separate from the management features.

## hToken

hToken is a token for managing bridged assets, which is matched 1:1 with the underlying token deposited in the bridge. 

There are two types of hTokens: branch hTokens and root hTokens.The root hToken is also known as the global token, which is minted only when the underlying token is deposited at branch bridge.

The branch hToken can be received when requesting bridging from the root chain to a branch chain, if user choose to receive it as an hToken rather than an underlying token.

Here's an example. You want to bridge a token called USDC. The existing USDC token is the underlying token.

USDC tokens exist on both the Ethereum and Polygon chains. However, even though they are the same USDC token, the bridge system manages them separately on each chain, and you should think of them as two different, independent tokens.

A global token is created for each chain's USDC (Underlying token), and if you bring Ethereum's USDC to Polygon, it will be minted as a branch hToken token. (See Polygon's blue Ethereum USDC branch hToken minted)

![https://user-images.githubusercontent.com/70058709/273309334-d788b905-eec0-43e8-966b-8d59f2bf7550.png](https://user-images.githubusercontent.com/70058709/273309334-d788b905-eec0-43e8-966b-8d59f2bf7550.png)

Branch hTokens do not need to exist on every chain. If you want to move Polygon's USDC to Ethereum but there is no branch hToken for Polygon USDC, you can first requesting to deploy a Polygon USDC branch hToken, and second move your Polygon USDC into Ethereum as branch hToken.

Arbitrum does not have a branch hToken, but uses the Root hToken(global token) directly. Because it operates on the same chain, it can be minted/burned directly without going through LZ.

## Virtual Account

One Virtual Account contract per user address is assigned to the root chain. For example, if you use an EOA to bridge tokens to the root chain, it transfers or mints global token to the Virtual Account contract owned by that EOA.

The Virtual Account can only be accessed by a preregistered Router contract or EOA. By putting a payload into the virtual account, you can make contract do arbitrary contract call and send/receive NFTs.

## Contract relationship

![https://user-images.githubusercontent.com/70058709/273310276-6cfd7c5b-86ef-48b3-b0c1-e1db9d30fd3f.png](https://user-images.githubusercontent.com/70058709/273310276-6cfd7c5b-86ef-48b3-b0c1-e1db9d30fd3f.png)

The user requests bridging by calling a function on the Router or Bridge Agent.

The Bridge Agent is responsible for receiving the user's request and passing it to the LayerZero endpoint, or receiving and processing messages from the LayerZero endpoint.

The Bridge agent executor is called when a bridging request comes in. It is responsible for giving the bridged token and passing the task to the Router if further action is required.

Port manages deposited underlying token and hToken. There is one Port for each chain, and it stores the underlying token deposited by the user for bridging and manages the hToken.

![https://user-images.githubusercontent.com/70058709/273312771-184ccc2c-bacf-47c4-a190-461647fc3097.png](https://user-images.githubusercontent.com/70058709/273312771-184ccc2c-bacf-47c4-a190-461647fc3097.png)

Each chain has one Core bridge agent/bridge agent executor/router, and many regular bridge agent/bridge agent executor/router sets which using customized routers. 

If a user wants to integrate their own logic into the bridge, they can deploy an agent, agent executor, and router to root and branch. The agent and agent executor are generated by the factory, while the router can be customized and deployed by the user. User can add the desired tasks to the router to integrate with the bridge.

A user-deployed root agent needs a branch agent to communicate with it. They should be deployed in pairs.

### Time spent:
36 hours