**1. Context**

A time-boxed architecture review of the Maia DAO Ecosystem was done by @0xabi96 (twitter). I work as a full-time Software Engineer in Distributed Ledgers.

**2. Approach taken in evaluating the codebase**
- Documentation [https://v2-docs.maiadao.io/]
- Github [https://github.com/code-423n4/2023-05-maia]
- Video Tutorial [https://maiaeco.notion.site/Code4rena-Code-Overview-3f455e66c457427ab3650a73a614641c]
- Layer Zero Documentation [https://layerzero.gitbook.io/docs/evm-guides/interfaces/evm-solidity-interfaces/ilayerzeroendpoint]
- VS Code: Solidity Visual Developer, CVL (Certora Verification Language)
Solidity Visual Developer uses a graph structure to display how the contracts interact with the functions. It shows how many and which internal and external calls to functions are done to get a good overview of the system.
- CVL is used to formally verify contracts compared to a given spec. This can be used to test against the main invariants considered in scope and beyond. I will cover this more in my findings report separately.

**3. Architecture Recommendations**

The following architecture improvements and feedback could be considered:

**3.1 ERC1155Receiver contracts are deprecated by OpenZeppelin**
Since _VirtualAccount.sol_ implements the ERC1155Receiver interface, it is deprecated with a newer version of Openzeppelin (release tag v5.0.0-rc.0). Please implement ERC1155Holder into _VirtualAccount.sol_ when updating Openzeppelin. 
The release note says -> ERC1155Receiver: Removed in favor of ERC1155Holder.* 

*Source: [https://github.com/OpenZeppelin/openzeppelin-contracts/releases].

**4.1 Systemic Risks**

In _BranchPort.sol_, the code will underflow if there's not enough debt to repay. Consider adding a check to ensure that there's enough debt to repay before subtracting _amount.

Use a simple if-statement to check: ```if(getPortStrategyTokenDebt[msg.sender][_token] > _amount)```.

The same check should be used: ```if(getStrategyTokenDebt[_token] > _amount)```

This topic will be covered in depth in my finding. 


### Time spent:
8 hours