### Any comments for the judge to contextualize your findings
Context was added where it was relevant to aid the judges. 

###  Approach taken in evaluating the protocol

Since this audit was focused on the Ulysses protocol, here are the steps taken to complete this audit. 

1. Went through the [Ulysses docs](https://v2-docs.maiadao.io/protocols/Ulysses/introduction/) to understand the protocol. 
	- There are 4 major components: 
		1. Ports: Root and Branch ports.
        2. Routers: Root and Branch routers.
		3. Bridge Agents: Root and Branch Bridge Agents.
		4. Virtualized Accounts and Liquidity. 
2.  Drew two diagrams based on this understanding. [[1]](https://postimg.cc/CR2KTZcb),[[2]](https://postimg.cc/BLjQn2vq)
3.  Started to read the codebase high-level, starting from root and branch ports. 
4.  With the docs in mind, I started to read each contract matches their supposed functionalities, functions calls (depending on how they are chained) do what they are supposed to do. 
5. When in doubt or confused, I consult the docs or ask in the discord channel dedicated for the audit. The sponsor and other wardens were kind enough to answer my questions.   
6.  Next, I evaluate the contracts's logic, following the chained internal functions.
7.  Over the course of the above processes, I take notes about observations and possible attack vectors/paths. 
8. Last but not least, I review the notes and examine observations/attack vectors with more depth. 

### Observations 
- Ulysses codebase has some of the cleanest codes out of the box. 
- More often than not, Integration with Layerzero follows [best practices.](https://layerzero.gitbook.io/docs/evm-guides/best-practice)
- Inconsistencies is some part of the codebase e.g usage of reverts and require simultaneously plus several missing checks for addresses passed to core functions. 


### Architecture recommendations
- The current design uses `_amount - _deposit` in several parts of the codebase for calculation of the htokens that a user wants to bridge in/out or use in a transaction. This is counter-intuitive in some cases. For instance: If a user has htoken in source chain but only wants to use underlying tokens for a specific transaction, they would have no means to do so. A boolean can be passed as well to indicate if the user wants to use htokens or not before even doing the calculation of htokens for the tx. 

- Minor but, error handling is good but it could be better if it includes the contract where the error originated from. e.g `error BranchPort__InvalidInputArrays();` instead of generic `error InvalidInputArrays();`
	

### Systemic and/or Centralization risks
The system is designed to be as decentralized as possible. However, ownership is renounced is some areas of the codebase. 


### Time spent:
80 hours.


### Time spent:
80 hours