# [L-01] Contract can receive ETH but has no withdraw function for it

The `ArbitrumCoreBranchRouter.sol` contract has two functions which are set as payable `addLocalToken` and `executeNoSettlement`, because of that they are able to receive Ether. Unfortunately contract is not providing option if some Ether amount is sent to the contract (mistakenly or not) to be withdrawn later.

```solidty
 function executeNoSettlement(bytes calldata _data) external payable override requiresAgentExecutor 
```

```solidty
    function addLocalToken(address _underlyingAddress, GasParams calldata) external payable override
 ```

 
Recommendation:
Add withdraw function or reconfigure functions as not payable in order to prevent such a scenario and loss of funds