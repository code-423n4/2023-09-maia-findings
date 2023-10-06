## Introduction

Greetings Judges and Team,
I am Aamir, a budding auditor in the field of smart contract security. This report outlines my findings after conducting an audit of the project.


## Findings:

| Priority       | Count |
| -------------- | ----- |
| High           | 3     |
| Medium         | 1     |
| Low            | 0     |
| Gas            | 2     |
| Non Critical   | 2     |

## Methodology

### Audit Process

I found all of these issues by going through the code manually. I want to mention that experienced auditors usually use a mix of automated tools and code reviews to help them out. As I gain more experience, I'm looking to add these tools and techniques to my arsenal to make the auditing process even better

### Tools and Techniques
Foundry and Remix IDE

## Codebase quality analysis
Although I was not able to go through all of the contracts but I can say that the code was pretty tough to crack. Most of the important functions were written with a lot of precaution. After spending around 110 hours on this audit I still couldn't test all of the function because either I couldn't get much time or I couldn't understand the functionality. 

## For judges
I have submitted POC for all of my findings. These Test functions uses some mock contracts that you can find in `test/ulysses-omnichain/mocks` folder. Some of them are also available at the end of this `test/ulysses-omnichain/RootForkTest.t.sol` file. Some tests use some custom internal function that are located just below the tests functions itself. There is `test/ulysses-omnichain/weird-tokens` folder as well that contains the contract for weird behavior tokens mocks like `fee-on-transfer` tokens. Please have a look at them when you find some weird function or contract that is not available in the original codebase.

## Centralization risks
I personally think that a lot of control is given to third parties for implementing their own Routers and Agents. These two are essential part of the codebase and all of the users will interact with these contract only. And chances are the party who will implement these contracts might try to do some malicious activity. I don't know I might be completely wrong. Just giving my opinion on this.

## Architecture
I think using `LayerZero` is an excellent choice for this as it is reputed organization and has undergone a lot of audits for it's contracts. One thing that I don't like is that some functions are given in `Agent` contracts and according to the docs, the interacting point for the users should be routers not agents if I am not wrong. This also gives a convenience to a person who is trying to use these contract in his own codebase.

## Conclusion

In conclusion, I had a lot of fun doing this audit. I got to learn a lot and got to know about some interesting use cases of different protocols as well. It was a great opportunity for me to improve myself and learn new things. Thanks everybody for giving this opportunity :)







### Time spent:
110 hours