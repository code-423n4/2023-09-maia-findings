## 1. REDUNDANT CONDITIONAL CHECK IN THE `BranchBridgeAgent.redeemDeposit` CAN BE OMITTED TO SAVE GAS

The `BranchBridgeAgent.redeemDeposit` function is used to redeem the deposit by a given `msg.sender`. The `redeemDeposit` function performs the following input validation checks using the `require` statements.

        if (deposit.owner == address(0)) revert DepositRedeemUnavailable(); 
        if (deposit.owner != msg.sender) revert NotDepositOwner(); 

As it is seen above the `deposit.owner == address(0)` conditional checks is redundant since it is anyway verified that the `deposit.owner` is not a zero address in the conditional check which follows it.

If the `deposit.owner == address(0)` then `deposit.owner != msg.sender` is also true since the `msg.sender can not be address zero`. Hence it is recommended to remove the redundant `deposit.owner == address(0)` conditional check to save gas. The modified code is shown below:

        if (deposit.status == STATUS_SUCCESS) revert DepositRedeemUnavailable();
        if (deposit.owner != msg.sender) revert NotDepositOwner(); 

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L439-L441

## 2. `delete` CAN BE USED TO OBTAIN A GAS REFUND INSTEAD OF RESETTING TO ADDRESS(0)

The `BranchBridgeAgent.redeemDeposit` function is used to redeem the deposit by a given `msg.sender`. Inside the `redeemDeposit` function the `deposit.owner` is assigned the `address(0)` in order to zero out the owner as shown below:

        deposit.owner = address(0); 

But the `delete` can be used to obtain a gas refund instead of assigning to `address(0)`. Hence the modified line of code is shown below:

        delete deposit.owner;

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L444

## 3. REDUNDANT `ADD` ARITHMETIC OPERATION CAN BE REPLACED TO SAVE GAS

The `BranchBridgeAgentExecutor.executeWithSettlementMultiple` function is used to execute a cross-chain request with multiple settlements. In the function execution if the `_payload.length > settlementEndOffset` condition is satisfied (which means there is `calldata` to be executed) `executeSettlementMultiple` function of the `Router` contract is called as shown below:

            IRouter(_router).executeSettlementMultiple{value: msg.value}(
                _payload[PARAMS_END_SIGNED_OFFSET + assetsOffset:], sParams
            );

The `bytes` range of the `_payload` is determined as `PARAMS_END_SIGNED_OFFSET + assetsOffset:`. But the arithmetic addition used in the above range calculation can be replaced by using the `settlementEndOffset` value since the `settlementEndOffset == PARAMS_END_SIGNED_OFFSET + assetsOffset`. Hence this will save gas on one `+` arithmetic operation. Hence the above code snippet can be modified as shown below:

            IRouter(_router).executeSettlementMultiple{value: msg.value}(
                _payload[settlementEndOffset:], sParams
            );

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgentExecutor.sol#L121-L123

## 4. ADDRESS VALIDATION CHECK CAN BE PERFORMED AT THE BEGINNING OF THE FUNCTION EXECUTION TO SAVE GAS INCASE OF TRANSACTION REVERT 

The `RootBridgeAgent._performRetrySettlementCall` function performs call to Layer Zero Endpoint Contract for cross-chain messaging. During the function execution there is conditional check to ensure that the `remoteBridgeAgent` is a valid address and is not `address(0)` as shown below:

        address callee = getBranchBridgeAgent[_dstChainId];

        // Check if valid destination
        if (callee == address(0)) revert UnrecognizedBridgeAgent(); 

But the problem here is that this conditional check is performed after the `payload` computation for the remote call for either single asset settlement or multiple asset settlement. Hence if the `callee == address(0)` condition results in `true` the transaction will revert thus wasting the gas consumed on the `payload` computation.

Hence it is recommended to perform the `callee == address(0)` conditional check at the beggining of the `RootBridgeAgent._performRetrySettlementCall` execution thus saving gas if the transaction reverts due to the `callee == address(0)` condition resulting in `false`.

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L906-L910

## 5. REDUNDANT ARITHMETIC OPERATIONS IN THE `RootBridgeAgentExecutor.executeWithDepositMultiple` CAN BE OMITTED TO SAVE GAS

The `RootBridgeAgentExecutor.executeWithDepositMultiple` function is used to execute a remote call with multiple deposits. The encoded data payload is passed to the `executeWithDepositMultiple` as `_payload` and the byte array is sliced to obtain the required portion of the `_payload` to execute the remote call. 

The following calculation is performed to calculate the ending index of the `_payload` which represents the deposit details of the assets.

    PARAMS_END_OFFSET + uint256(uint8(bytes1(_payload[PARAMS_START]))) * PARAMS_TKN_SET_SIZE_MULTIPLE

But the issue is this calculation is performed three times with in the `executeWithDepositMultiple` function redundantly. 

Other two locations are as follows:

        if (length > PARAMS_END_OFFSET + (numOfAssets * PARAMS_TKN_SET_SIZE_MULTIPLE)) {

        IRouter(_router).executeDepositMultiple{value: msg.value}(
            _payload[PARAMS_END_OFFSET + uint256(numOfAssets) * PARAMS_TKN_SET_SIZE_MULTIPLE:], dParams, _srcChainId
        );

Hence it is recommended to perform the above mentioned calculation and assign the result into a memory variable and read the value from the memory to other two locations where the above calculation is used. This will save gas which is spent on redundant arithmetic operations to calculation the above mentioned bytes array index.

https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L125
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L134
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L136-L138
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L210-L214
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L220-L222
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgentExecutor.sol#L226-L229

## 6. THERE IS NO EQUALITY CHECK ON THE ARRAY LENGTHS IN THE `BaseBranchRouter.callOutAndBridgeMultiple` FUNCTION WHICH COULD LEAD TO GAS WASTAGE IN THE EVENT OF A REVERT

The `BaseBranchRouter.callOutAndBridgeMultiple` function is used to bridge in multiple tokens to the root. The function calls the `_transferAndApproveMultipleTokens` internal function which is used to transfer multiple tokens to the `BaseBranchRouter` contract and approve the tokens to the `_localPortAddress` contract. This is performed by iterating through a `for loop` till `_hTokens.length` is reached.

The issue here is if there is a length mismatch in any of the `_hTokens`, `_tokens`, `_amounts` or `_deposits` the transaction will revert inside the `for loop` due to `index out of bound` execption thus wasting the user's gas spent on the computations performed till that point.

Hence it is recommended to perform an array length equality check for the `_hTokens`, `_tokens`, `_amounts` and `_deposits` arrays inside the `BaseBranchRouter.callOutAndBridgeMultiple` function, before `_transferAndApproveMultipleTokens` function is called. Hence if there is an array length mismath then the transaction will revert at the very beginning of its execution without wasting gas on further computation.

https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L104-L110
https://github.com/code-423n4/2023-09-maia/blob/main/src/BaseBranchRouter.sol#L192-L198

## 7. `deposit.status` IS NOT CHECKED AT THE BEGINNING OF THE `BranchBridgeAgent.retryDeposit` FUNCTION WHICH ALLOWS THE `retryDeposit` TO BE CALLED FOR THE SECOND TIME THUS WASTING GAS

The `BranchBridgeAgent.retryDeposit` function is used to retry the deposit transaction when the transaction is failed before. In the execution of the `BranchBridgeAgent.retryDeposit` the `deposit.status` will be updated to `STATUS_SUCCESS` as shown below:

        deposit.status = STATUS_SUCCESS;

But the issue here is the `deposit.status` is not checked for `STATUS_SUCCESS` at the beginning of the `retryDeposit` function which would allow a `deposit.owner` to mistakenly call the `retryDeposit` on the `_depositNonce` for the second time even after succesfully executing it the first time. As a result the second call will only revert in the `RootBridgeAgent.lzReceiveNonBlocking` function when `_payload[0] == 0x03` occurs and the following condition occurs:

            if (executionState[_srcChainId][nonce] != STATUS_READY) {
                revert AlreadyExecutedTransaction();
            }

But the gas spent on the second `retryDeposit` transaction till it reverts after reaching the `RootBridgeAgent.lzReceiveNonBlocking` revert conditon will be lost. But this gas wastage can be avoided by performing the status check on the `deposit.status` value at the beginning of the `BranchBridgeAgent.retryDeposit` function as shown below:

    require(deposit.status != STATUS_SUCCESS, "Deposit already executed");

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L415
https://github.com/code-423n4/2023-09-maia/blob/main/src/RootBridgeAgent.sol#L518-L520

## 8. ARITHMETIC OPERATIONS CAN BE `unchecked` DUE TO PREVIOUS CONDITIONAL CHECKS, TO SAVE GAS

The `BranchPort.replenishReserves` function is used to replenish the minimum reserves required for a particular `strategy token`. The `strategy tokens` will be withdrawn from the specific `_strategy` to replenish the minimum reserve amount.

`getPortStrategyTokenDebt` and `getStrategyTokenDebt` mappings are updated accordingly by the replenished strategy token amount as shown below:

        // Update Port Strategy Token Debt
        getPortStrategyTokenDebt[_strategy][_token] = portStrategyTokenDebt - amountToWithdraw;
        // Update Strategy Token Global Debt
        getStrategyTokenDebt[_token] = strategyTokenDebt - amountToWithdraw;

Both of the above arithmetic operations can not underflow due to previous conditonal check which is given below:

        uint256 amountToWithdraw = portStrategyTokenDebt < reservesLacking ? portStrategyTokenDebt : reservesLacking;

Hence it is recommended to `unchecked` the `getPortStrategyTokenDebt` and `getStrategyTokenDebt` updates to save gas.

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L201-L207

## 9. REDUNDANT `require` STATEMENT CAN BE REMOVED TO SAVE GAS IN THE `BranchPort.setCoreRouter` FUNCTION

The `BranchPort.setCoreRouter` is used to set a new `coreBranchRouterAddress` address as shown below:

    function setCoreRouter(address _newCoreRouter) external override requiresCoreRouter {
        require(coreBranchRouterAddress != address(0), "CoreRouter address is zero");
        require(_newCoreRouter != address(0), "New CoreRouter address is zero");
        coreBranchRouterAddress = _newCoreRouter;
    }

There is a redundant check of `coreBranchRouterAddress != address(0)` check which is redundant and hence can be removed to save gas. Since the `setCoreRouter` function is controlled by the `requiresCoreRouter` modifier, only the `coreBranchRouterAddress` can call the `setCoreRouter` function. Hence the `coreBranchRouterAddress` can never be `address(0)`. 

Hence the `coreBranchRouterAddress != address(0)` require statement check is redundant and should be removed to save gas.

https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchPort.sol#L331-L335