[L-1] the strict equity check on replenishReserves is unnessary as some strategies may create additional withdrawal due to excess return.

PortStrategy operates on port balance of a token to incur yield, if applicable. the port tokens can be called back through a call of `withdraw` of the strategy contract. however the balance post-withdrawal is required to be exactly equal to the sum of before + the required withdrawal, this creates a problem for strategy that would have internal pricing and would withdraw additional fund.

Another issue is the strategy has no way to report gains to the debt, so that there is no way to increase the overall token balance recorded in the BranchPort, even if the strategy makes a profit based on its management. 

```solidity
require(ERC20(_token).balanceOf(address(this)) - currBalance == _amount, "Port Strategy Withdraw Failed");
```

** Recommendation
enforce greater than equal to.

```solidity
require(ERC20(_token).balanceOf(address(this)) - currBalance >= _amount, "Port Strategy Withdraw Failed");
```


[L-2] addStrategyToken in BranchPort should check duplicate of `_token`.

addStrategyToken would push the token to an array of strategyTokens, and setting the token specific minimumReserveRatio. Hence it should add checks of duplcate to prevent overwritting an existing ones.

```solidity
    function addStrategyToken(address _token, uint256 _minimumReservesRatio) external override requiresCoreRouter {
        if (_minimumReservesRatio >= DIVISIONER || _minimumReservesRatio < MIN_RESERVE_RATIO) {
            revert InvalidMinimumReservesRatio();
        }

        strategyTokens.push(_token);
        getMinimumTokenReserveRatio[_token] = _minimumReservesRatio;
        isStrategyToken[_token] = true;

        emit StrategyTokenAdded(_token, _minimumReservesRatio);
    }
```

## Recommendation

```solidity
    function addStrategyToken(address _token, uint256 _minimumReservesRatio) external override requiresCoreRouter {
+++ require(getMinimumTokenReserveRatio[_token] == 0, "token already exists");