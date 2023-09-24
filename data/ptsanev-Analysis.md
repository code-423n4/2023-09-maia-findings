## Summary and approach
The Ulysses ecosystem of the Maia DAO is an omni-chain virtual liquidity management system, with it's roots based on Arbitrium.
The a general approach I took was to go through the docs, since I wasn't a participant of the last contest, which this is a partial continuation of. After getting general familiarity with goals and concepts, I started from the higher SLOC files, since these are usually the files containing core functionality, in my case the ``RootBridgeAgent.sol``. Initial misunderstanding was cleared thanks to the active communication with the dev team and other auditors. Gathering notes, following some basic function flows and looking at the ecosystem from a higher level to try and spot inconsistencies.

## Codebase and protocol analysis
The codebase is well written and explained with the usage of NATSPEC comments. The fragmentation of core functionality makes it easier to navigate through the contracts as the bridge, port and executors are separate files.
The idea of having a system like a real life company with the Root being the HQ, the Arbitrium contracts being something like an office at the same city as the HQ and the different Branches being different offices in other cities. It helps with readability and understandability.
In my opinion the Arbitrium network is a great choice for the root due to it's low gas costs and almost no re-org occurrences.
The protocol makes use of LayerZero for cross-chain messaging and they have made sure to avoid the common and uncommon issues that come with bridging assets and sending omni-chain messages such as assuming the same ``msg.sender`` address, refunding gas correctly and avoiding channel blockage in case of failed transactions.
The protocol is taking an interesting approach in utilizing reading from a packed decoding and encoding multiples of payloads in a single transactions.
The test coverage of the protocol is exceptionally low for a codebase of this size, amounting only to 69%, which in on itself can be a flaw and even though this is an audit, good test coverage makes it easier for the auditors to focus.
There is a lot of control flow being passed around the contracts and even though there is a lot of fragmentation, there are some flow that go circularly and can be fixed so they can be more easily trackable during a code review,
an example being router calling agent calling executor which calls the agent again.

## Systematical risks and system recommendations
I have some general systematic risks such as the owner being a single point of centralization.
The non-existence of other based roles can cause problems in cases of unforeseen events involving private keys and compromisation.
Making use of such quantities of packed encoding all around the codebase is a risky approach for the system as packed encoding is generally prone to collisions and they are harder to decode with potential to lose information.
Forks of different chains and as stated above - unforeseeable events can cause some forms of havoc or simply confusion in the protocol, due to which pausable functionality and upgradeability, even though complexity would increase, could help the protocol avoid certain scenarios, have more control over their security and have potential migrations and upgrades in the future cost less.
So as a sum up:
1. Implement the owner as a multisig or introduce more roles to keep the ecosystem safe on itself and more trustworthy in the eyes of users.
2. Implement upgradability or atleast pausable functionality to avoid consequences from unforeseen events
3. Rethink usage of packed encoding in key areas. There are some instances in the codebase where regular encode/decode is used and in my opinion that is a far safer option at the expense of some gas.

### Time spent:
20 hours