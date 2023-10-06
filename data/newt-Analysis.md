## Analysis report

My findings are not a deep dive into a function but rather low hanging fruits that can cause serious damage to the protocol. There are two instances where Root Bridge Agent is being created. One inherits from the other, both with no access control. Though I could not find where rootBridgagentFactory.sol was being imported and used.
After submitting my gas report. I later found more mitigations steps that could be taken to save gas costs. Sadly I could not add another report.

The codebase was reviewed manually as I was working with low hanging fruits with deep impacts. The comments in the code were top tier, easy to read and remember.

There are unnecessary contracts that could have been added into other contracts (Not 2 contracts in 1 .sol file. Thus limiting the number of contracts in the protocol for easier understanding for both auditors and future smart contract engineers of Maia DAO. With that being said the codebase was not difficult to understand from an auditors stand point, simply straight forward. 

I'd recommend that the main contracts (contracts that make transactions and calculations) be stored separately, and so forth. It is easier to work in an organized work space.

Overall I enjoyed auditing the Maia Dao protocol.

### Time spent:
18 hours