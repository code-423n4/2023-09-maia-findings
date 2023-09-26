[L-1] the strict equity check on replenishReserves is unnessary as some strategies may create additional withdrawal due to excess return.

PortStrategy operates on port balance of a token to incur yield, if applicable. the port tokens can be called back through a call of `withdraw` of the strategy contract. however the balance post-withdrawal is required to be exactly equal to the sum of before + the required withdrawal, this creates a problem for strategy that would have internal pricing and would withdraw additional fund.

```solidity
require(ERC20(_token).balanceOf(address(this)) - currBalance == _amount, "Port Strategy Withdraw Failed");
```

** Recommendation
enforce greater than equal to.

```solidity
require(ERC20(_token).balanceOf(address(this)) - currBalance >= _amount, "Port Strategy Withdraw Failed");
```