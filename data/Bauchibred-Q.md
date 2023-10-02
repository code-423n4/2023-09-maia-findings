# QA Report

## **Table of Contents**

|       | Issue                                                                                                                         |
| ----- | ----------------------------------------------------------------------------------------------------------------------------- |
| QA-01 | Multiple issues with how chain Ids are being used in protocol                                                                 |
| QA-02 | Replenishing reserves should be made to vet strategies easier                                                                 |
| QA-03 | Using a nonce of `uint 32` limits the transactions possible in the protocol to `4294967295`                                   |
| QA-04 | The internal function used to check if a Port Strategy has reached its daily management limit could be gamed                  |
| QA-05 | Toggled bridge agent/agent factory could get added for a second time in bridgeAgent/AgentFactories array                      |
| QA-06 | Anyone can create a RootBridgeAgent which is risky since they can directly interact with RoorPort                             |
| QA-07 | Use safeTransferFrom instead of transferFrom for ERC721 transfers                                                             |
| QA-08 | `block.number` will not be a reliable source of timing on Optimism                                                            |
| QA-09 | More checks against `0` inputs should be implemented                                                                          |
| QA-10 | Inconsistencies between sister function implementations                                                                       |
| QA-11 | Multiple contracts either use Ownable but don't need to use it or import it without using it                                  |
| QA-12 | Unlike the `MulticallRootRouterLibZip` the `MultiCallRootRouter` does not correctly decode data                               |
| QA-13 | `ERC20hTokenBranch & ERC20hTokenRoot` could have compatibility issues with tokens that purely comply with EIP-20              |
| QA-14 | Owner currently can't execute any extra calldata that's needed in the case where `_payload.length > PARAMS_SETTLEMENT_OFFSET` |

(Note: You've listed QA-09 twice, so I've assumed it was a mistake and included it only once.)

## QA-01 Multiple issues with how chain Ids are being used in protocol

issue with using uint16 for local chain id, it's not consistent and could lead to issues

Where as layerZero currently uses uint16

### Impact

Low to _medium_, but heavy on the low side since chances are too slim for this to be the case.

### Proof of Concept

### Issue 1

Multiple instances currently have the localchainId being stored as a `uint16` variable, for e.g in the `ArbitrumBranchPort.sol` contract and some have it stored as `uint256`, probably done in order to save gas

Take a look at the `BranchBrdgeAgentFactory.sol's` constructor for example

```solidity
    constructor(
        uint16 _localChainId,
        uint16 _rootChainId,
        address _rootBridgeAgentFactoryAddress,
        address _lzEndpointAddress,
        address _localCoreBranchRouterAddress,
        address _localPortAddress,
        address _owner
    ) {
        require(_rootBridgeAgentFactoryAddress != address(0), "Root Bridge Agent Factory Address cannot be 0");
        require(
            _lzEndpointAddress != address(0) || _rootChainId == _localChainId,
            "Layerzero Endpoint Address cannot be the zero address."
        );
        require(_localCoreBranchRouterAddress != address(0), "Core Branch Router Address cannot be 0");
        require(_localPortAddress != address(0), "Port Address cannot be 0");
        require(_owner != address(0), "Owner cannot be 0");

        localChainId = _localChainId;
        rootChainId = _rootChainId;
        rootBridgeAgentFactoryAddress = _rootBridgeAgentFactoryAddress;
        lzEndpointAddress = _lzEndpointAddress;
        localCoreBranchRouterAddress = _localCoreBranchRouterAddress;
        localPortAddress = _localPortAddress;
        _initializeOwner(_owner);
    }
```

From the above one can thereby see that both `_localChainId` and `_rootChainId` have been changed to `uint16` unlike the former contracts that has them in `uint256`, this mean that any chain with id abovc `65556` can't be used.

### Issue 2

Layer Zero chain Ids could be changed in the future

> Impact of this is also low, info since this would only mean that they would be a DOS (`revertions of any cross chain messaging to/fro the deprecated chain Id`) until the chainId gets updated via migratig the `root` or `BranchBridgeAgentFactory`

Nevertheless multiple contracts within protocol are going to hardcoded with their respective chain Ids, with them being stored as immutable variables, for example see here: https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/BranchBridgeAgent.sol#L119

The ethereum mainnet's chain Id used to be 1 and now it's 101

Chain Ids associated with Layer zero have been changed in the past, with an instance being due to a security upgrade, these contracts would be non-functional since any attempt of a cross message to and fro would be to the wrong chain id

See [this](https://discord.com/channels/881985666265780274/881992936609435688/1077494897625534538), from the official LayerZero discord group, do note that `10Xkelly` is a part of the LayerZero team

![hey](https://github-production-user-asset-6210df.s3.amazonaws.com/107410002/271774649-b9c274a6-857c-414c-a96e-8b5fa2f6431f.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20230930%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20230930T131710Z&X-Amz-Expires=300&X-Amz-Signature=e7cbcf887a4321a984eadaeb860fc41e9571a26a8f1cbb07469bccb917563816&X-Amz-SignedHeaders=host&actor_id=107410002&key_id=0&repo_id=605287485)

### Recommended Mitigation Steps

For the first issue, mske a generic storing of this, advisable to just use a `uint256` since if the chain Ids could be change nothing is stoppin LayerZero from deciding to use `uint256` in the future to store their Ids.

For the second case, follow recommendations made by the LZ team and ensure that fast replies are done for whenever a chain Id changes, i.e query the chain Ids for transactions and in a case where they've changed then they should be migrated immediately

## QA-02 Replenishing reserves should be made to vet strategies easier

### Impact

Low, _suggestion on how to easier vet strategies_

### Proof of Concept

Take a look at `replenishReserves()`

```
    function replenishReserves(address _strategy, address _token) external override lock {
        // Cache Strategy Token Global Debt
        uint256 strategyTokenDebt = getStrategyTokenDebt[_token];

        // Get current balance of _token
        uint256 currBalance = ERC20(_token).balanceOf(address(this));

        // Get reserves lacking
        uint256 reservesLacking = _reservesLacking(strategyTokenDebt, _token, currBalance);

        // Cache Port Strategy Token Debt
        uint256 portStrategyTokenDebt = getPortStrategyTokenDebt[_strategy][_token];

        // Calculate amount to withdraw. The lesser of reserves lacking or Strategy Token Global Debt.
        uint256 amountToWithdraw = portStrategyTokenDebt < reservesLacking ? portStrategyTokenDebt : reservesLacking;

        // Update Port Strategy Token Debt
        getPortStrategyTokenDebt[_strategy][_token] = portStrategyTokenDebt - amountToWithdraw;
        // Update Strategy Token Global Debt
        getStrategyTokenDebt[_token] = strategyTokenDebt - amountToWithdraw;

        // Withdraw tokens from startegy
        IPortStrategy(_strategy).withdraw(address(this), _token, amountToWithdraw);
        //@audit-issue
        // Check if _token balance has increased by _amount
        require(
            ERC20(_token).balanceOf(address(this)) - currBalance == amountToWithdraw, "Port Strategy Withdraw Failed"
        );

        // Emit DebtRepaid event
        emit DebtRepaid(_strategy, _token, amountToWithdraw);
    }

```

As seen where as the above process of replenishing reserves is not vulnerable to the classic `1 wei attack` due to the nature of it's current implementation of querying the balance twice and the only way the attack could be reached is by one breaking in during the withdrawal itself.

The check that's been implemented: `        require( ERC20(_token).balanceOf(address(this)) - currBalance == amountToWithdraw, "Port Strategy Withdraw Failed" );` makes it harder to vet strategies since the condition is too strict

### Recommended Mitigation Steps

Change the check and make it `ERC20(_token).balanceOf(address(this)) - currBalance >= amountToWithdraw` instead, this way vetting strategies would be easier.

## QA-03 Using a nonce of `uint 32` limits the transactions possiible in the protocol to `4294967295`

### Impact

Currently when a total of `4294967295` interactions(including non-bridging interactions) have been made on say a BranchBridgeAgent, the contract becomes unusable

This should be a medium given the high chances of this happening(though it may take a long period), the impact on the protocol when it happens(Protocol will be totally unusable), and the difficulty of fixing the issue when it happens(cannot be fixed through redeployment, and contract is not upgradeable), and the ability of a malicious user to catalyze this on ArbitrumBranch without losing anything(apart from Arbitrum gas fees), I believe this qualifies as atleast a medium impact issue, _but I'm submitting as QA hoping the judge upgrades this if he deems it fit_

### Proof of Concept

In `BranchBridgeAgent.sol`, the used nonces have a type uint32.
And almost all external functions, which users would interact with, increments them because they downstream call one of the `createDeposit` prefixed functions.

With this in mind, it's key to note that it's no _over-inflation_ to say that the `uint32` max value of `4294967295` will definitely be reached even though it may take a long period.

> NB: If this is even done on say the `ArbitrumBranchBridgeAgent.sol` contract, it'd cost aproximately _free_, since the malicious user would only need to pay the gas fees, which can be considered very cheap too

### Recommended Mitigation Steps

Change the nonce to be of type `uint256` instead, checking the differences between the max `uint32` value and the suggested `uint256`
\[ \frac{115792089237316195423570985008687907853269984665640564039457584007913129639935}{4,294,967,295} \]

One can see that not only does this make the attack \(2.6959946667150639794667015087019630673637144422540572481103610249215 \times 10^{61}\) times harder, it also ensures that the contracts never get unusable due to the amount of transactions.

## QA-04 The internal function used to check if a Port Strategy has reached its daily management limit could be gamed

### Impact

The internal function used to check if a Port Strategy has reached its daily management limit could be gamed and allow a port to spend even 2x it's daily limit in less than 24 hours, could even be as low as _1, 2 hours..._

### Proof of Concept

The logic used in the modified `_checkTimeLimit()` function in `BranchPort.sol` can be exploited by a strategy to potentially double its withdrawal limit in a short time span. This loophole arises from the logic that resets the `lastManaged[msg.sender][_token]` timestamp to the beginning of the current day every time the limit is reset, due to the this...

```solidity
        if (block.timestamp - lastManaged[msg.sender][_token] >= 1 days) {
            dailyLimit = strategyDailyLimitAmount[msg.sender][_token];
            unchecked {
                lastManaged[msg.sender][_token] = (block.timestamp / 1 days) * 1 days;
            }
        }
```

This line: `(block.timestamp / 1 days) * 1 days` essentially just sets
In practice, this means a strategy can use up its daily limit just before midnight and then immediately use its entire limit again right after midnight.

This flaw poses a risk to the protocol's assets and could lead to unexpected capital utilization and potential losses. If multiple strategies exploit this vulnerability simultaneously, the negative impact on the protocol could be amplified.

### Step-by-Step POC

1. **Initial Setup:** A daily limit has been set which is monitored and controlled by the `_checkTimeLimit` function.
2. **First Utilization:** A call to the `manage()` is made (which internally calls `_checkTimeLimit`) at say _11:30 PM on Day 1_ and consumes its entire daily limit. At this time, the `lastManaged[msg.sender][_token]` is set to the start of Day 1 due to `(block.timestamp / 1 days) * 1 days` logic, i.e _12:00 AM of Day 1_
3. **Limit Reset Exploit:** By 12:30 AM of Day 2, just after midnight, another call is made to the `manage()` function. Since 24 hours have passed since the start of Day 1 (which is the value stored in `lastManaged[msg.sender][_token]`), the daily limit is reset, this can be confirmed since the below check passes.

```solidity
        if (block.timestamp - lastManaged[msg.sender][_token] >= 1 days)
```

4. **Double Consumption:** Now, the `_amount` specified could even be daily limit again, effectively doubling its allowed withdrawal in a very short time span... in this case _just north of 1 hour (taking into account transaction delays and all.)_

Using this method, the strategy can consistently exploit the time-based logic, leading to higher withdrawals than intended by the protocol.

### Recommended Mitigation Steps

Do not round down the `block.timestamp` to the start of the current day if 24 hours has passed, Instead, set the lastManaged timestamp to the current timestamp, this way the dailyLimit is only reset if 24 hours has passed, below is the change to apply to the `_checkTimeLimit()` function.

```diff
 function _checkTimeLimit(address _token, uint256 _amount) internal {
     uint256 dailyLimit = strategyDailyLimitRemaining[msg.sender][_token];
     if (block.timestamp - lastManaged[msg.sender][_token] >= 1 days) {
         dailyLimit = strategyDailyLimitAmount[msg.sender][_token];
         unchecked {
-                lastManaged[msg.sender][_token] = (block.timestamp / 1 days) * 1 days;
+                lastManaged[msg.sender][_token] = block.timestamp;
         }
     }
     strategyDailyLimitRemaining[msg.sender][_token] = dailyLimit - _amount;
 }
```

> This was submitted as QA after extensive discussion with sponsors and their argument on how this is an intended design to have a max `per day` and not `per 24 hours`, but in whatever case if there is no problem against having a strategy user `2x their daily limit in less than 24 hours` this should be clearly documented and if additional intention is to always have ` lastManaged[msg.sender][_token]` date back to `00:00` of every day then a different approach should be considered due to the issue with leap seconds, otherwise I see no reason and would argue that judge upgrades this to a med severity finding as having strategies 2x their daily limit in less than 24 hours could lead to other issues as well.

## QA-05 Toggled bridge agent/agent factory could get added for a second time in bridgeAgent/AgentFactories array

### Impact

Low, this leads to clashing/duplicating records of bridge agents or their factories.

### Proof of Concept

Take a look at

```soldity
    function addBridgeAgent(address _manager, address _bridgeAgent) external override requiresBridgeAgentFactory {
        if (isBridgeAgent[_bridgeAgent]) revert AlreadyAddedBridgeAgent();

        bridgeAgents.push(_bridgeAgent);
        getBridgeAgentManager[_bridgeAgent] = _manager;
        isBridgeAgent[_bridgeAgent] = true;

        emit BridgeAgentAdded(_bridgeAgent, _manager);
    }

        function toggleBridgeAgent(address _bridgeAgent) external override onlyOwner {
        isBridgeAgent[_bridgeAgent] = !isBridgeAgent[_bridgeAgent];

        emit BridgeAgentToggled(_bridgeAgent);
    }

    //... codeblocks ommitted for brevity reasons
        function addBridgeAgentFactory(address _bridgeAgentFactory) external override onlyOwner {
        if (isBridgeAgentFactory[_bridgeAgentFactory]) revert AlreadyAddedBridgeAgentFactory();

        bridgeAgentFactories.push(_bridgeAgentFactory);
        isBridgeAgentFactory[_bridgeAgentFactory] = true;

        emit BridgeAgentFactoryAdded(_bridgeAgentFactory);
    }
        function toggleBridgeAgentFactory(address _bridgeAgentFactory) external override onlyOwner {
        isBridgeAgentFactory[_bridgeAgentFactory] = !isBridgeAgentFactory[_bridgeAgentFactory];

        emit BridgeAgentFactoryToggled(_bridgeAgentFactory);
    }

```

As seen whenever an agent or a factory gets duplicated it's gets set to the opposite value of what it previously was _and not removed from the array_, now whenever an attempt is made to add an agent or an agent factory this state is being checked, this means that in a case where say the agent has been toggled off the same agent could get added another time in the array.

Even though this could lead to duplication and potentially having multiple agents or agent factories in their respective arrays, these arrays are onnly used to keep a record of them and they are not used elsewhere which justifies the low severity.


### Recommended Mitigation Steps

While adding do not only check if agent/agent factory has been toggled off, but alos if they've been added to the array previously

## QA-06 Anyone can create a RootBridgeAgent which is risky since they can directly interact with RoorPort

### Impact

Low, since this seems to be intended design, but a RootBridgeAgent created by a malicious user, with say a malicious router will be allowed to access some of RootPort critical functions.

### Proof of Concept

Take a look at [RootBridgeAgentFactory.sol#L48-L59](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/src/factories/RootBridgeAgentFactory.sol#L48-L59)

```solidity
    function createBridgeAgent(address _newRootRouterAddress) external returns (address newBridgeAgent) {
        newBridgeAgent = address(
            new RootBridgeAgent(
                rootChainId,
                lzEndpointAddress,
                rootPortAddress,
                _newRootRouterAddress
            )
        );


        IRootPort(rootPortAddress).addBridgeAgent(msg.sender, newBridgeAgent);
    }
```

Anyone can call this function, passing in an invalid(or malicious) `_newRootRouterAddress`, and then `RootPort#addBridgeAgent` is called which immediately grants unto the deployed address, the isBridgeAgent role.

> NB:RootPort is the core of Ulysses, where critical functionalities are handled.

So now, a random BridgeAgent, created by a malicious user, with malicious router can now call RootPort functions which use `requiresBridgeAgent` modifier like `bridgeToRoot`,`burn`... etc.

### Recommended Mitigation Steps

Apply an access control to the function, i.e advisably only the admin should be allowed to call this

## QA-07 Use safeTransferFrom instead of transferFrom for ERC721 transfers

### Impact

VirtualAccount's `withdrawERC721()` is used to transfer ERC721 tokens, but when transferring ERC721 tokens to the recipient, the `transferFrom` keyword is used instead of `safeTransferFrom`, which is bad practise, since, if the recipient is a contract and is not capable of handling ERC721 tokens, the sent tokens could be locked.

### Proof Of Concept

Take a look at []()

Now do note that if `transferFrom()` is used for transferring an ERC721 token, there is no way of checking if the recipient address has capability to handle incoming ERC721 token. If there is no checking, the transaction will push through and the ERC721 token will be locked in the recipient contract address.

### Recommended Mitigation Steps

Use the `safeTransferFrom()` method instead of `transferFrom()` for NFT transfers. This call will revert if the `onERC721Received` function is not implemented in the receiving contract, not allowing an incompatible contract to exercise the option.

## QA-08 block.number will not be a reliable source of timing on Optimism

### Impact

Low, info, Optimism's block.timestamp has a bit of nuances compared to the mainnet that can be seen [here](https://community.optimism.io/docs/developers/bedrock/#handling-l1-and-l2-timestamps)

### Proof of Concept

The Ulysses omnichain contracts is to be deployed on different chains, most especially the brach contracts, but some of the concept around this is linked with using block.timestamp which could lead to some issues for chains such as Optimism

### Recommended Mitigation Steps

This should be taken into account and ensurances should be made that no core functionality around this could be broken

## QA-09 More checks against `0` inputs should be implemented

### Impact

Low, info on having better code structure to prevent unnecesary/costly errors.

### Proof Of Concept

This issue can be seen in multiple files, consider copy-pasting the search command below to see the instances

From the above we can see that

### Proof of Concept

Take a look at the `VirtualAccount.sol's withdrawERC20()` as an example

```solidity
function withdrawERC20(address _token, uint256 _amount) external override requiresApprovedCaller {
        _token.safeTransfer(msg.sender, _amount);
    }
```

According to https://github.com/d-xo/weird-erc20#revert-on-zero-value-transfers

The above would revert if the _token_ does not support 0 value transfer
Another case that could be attached to this is while minting,
Using this search command: https://github.com/search?q=repo%3Acode-423n4%2F2023-09-maia+mint%28&type=code, one can see that based on current implementation there is a whooping sum of 14 instances _(with a few being Out-of-Scope being that they are in test contracts)_ wherre tokens could get minted to the zero address.

### Recommended Mitigation Steps

Introduce a requirement that the amount should not be 0

> NB: The above is just two instances on how applying 0 checks could help improve code structure.

## QA-10 Inconsitencies between sister function implementations

### Impact

Low, inconsistency in implementing sister functions and in this case it's a security measure that's ben applied that's missing which could lead to issues

### Proof Of Concept

Take a look at

```solidity
  function bridgeInMultiple(
      address _recipient,
      address[] memory _localAddresses,
      address[] memory _underlyingAddresses,
      uint256[] memory _amounts,
      uint256[] memory _deposits
  ) external override requiresBridgeAgent {
      // Cache Length
      uint256 length = _localAddresses.length;
//@audit Question: why are  the sanity checks applied in `bridgeOutMultiple()` is not applied here, also why isnt't this locked
      // Loop through token inputs
      for (uint256 i = 0; i < length;) {
          // Check if hTokens are being bridged in
          if (_amounts[i] - _deposits[i] > 0) {
              unchecked {
                  _bridgeIn(_recipient, _localAddresses[i], _amounts[i] - _deposits[i]);
              }
          }

          // Check if underlying tokens are being cleared
          if (_deposits[i] > 0) {
              withdraw(_recipient, _underlyingAddresses[i], _deposits[i]);
          }

          unchecked {
              ++i;
          }
      }
  }
```

From the above one can see that while bridging in multiple tokens, multiple checks are applied from cehcking array lenghts and what not, but looking below at the `bridgeOutMultiple()` function one can see that these checks have not been applied which could lead to unforseen issues

```solidity
    /// @inheritdoc IBranchPort
    function bridgeOutMultiple(
        address _depositor,
        address[] memory _localAddresses,
        address[] memory _underlyingAddresses,
        uint256[] memory _amounts,
        uint256[] memory _deposits
    ) external override lock requiresBridgeAgent {
        // Cache Length
        uint256 length = _localAddresses.length;


        // Sanity Check input arrays
        if (length > 255) revert InvalidInputArrays();
        if (length != _underlyingAddresses.length) revert InvalidInputArrays();
        if (_underlyingAddresses.length != _amounts.length) revert InvalidInputArrays();
        if (_amounts.length != _deposits.length) revert InvalidInputArrays();


        // Loop through token inputs and bridge out
        for (uint256 i = 0; i < length;) {
            _bridgeOut(_depositor, _localAddresses[i], _underlyingAddresses[i], _amounts[i], _deposits[i]);


            unchecked {
                i++;
            }
        }
    }
```

### Recommended Mitigation Steps

Ensure to implement the same security checks in sister functions

## QA-11 Multiple contracts either use Ownable but don't need to use it or import it without using it

### Impact

Low, just an info on how to better implement code

### Proof Concept

1. Contracts that don't need to use it: ERC20hTokenBranchFactory.sol;
2. Contracts that import Ownable but don't use it: ArbitrumBranchPort.sol, CoreBranchRouter.sol, RootBridgeAgent.sol, RootBridgeAgent.sol, RootBridgeAgentFactory.sol, ERC20hTokenRoot.sol

### Recommended Mitigation Steps

More efforts should be placed on reducing bulkiness of code as much as possible

## QA-12 Unlike the `MulticallRootRouterLibZip` the `MultiCallRootRouter` does not correctly decode data

### Impact

Low, this is cause this seems to be intended behaviour, but at current the docs does not clearly cover this.

### Proof of Concept

Take a look at

```solidity
    function _decode(bytes calldata data) internal pure virtual returns (bytes memory) {
        return data;
    }
```

Now here is the implementation from `MulticallRootRouterLibZip.sol`

```solidity
    function _decode(bytes calldata data) internal pure override returns (bytes memory) {
        return data.cdDecompress();
    }
```

As seen in the latter case, the compressed version is used.
But all 11 attempts to decode data in the `MultiCallRootRouter.sol` contract would actually just return the same as is been passed to it, which is an unnecessary behaviour and just wastes gas querying the internal `_decode()` function

### Recommended Mitigation Steps

Correctly document this behaviour if intended or use the right implementation to decode data

## QA-13 `ERC20hTokenBranch & ERC20hTokenRoot` could have compatbility issues with tokems that purely comply with EIP-20

### Impact

Low, potential incompatibility with some ERC20 tokens which is not protocol's intended behaviour as specified in the [ReadME](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/README.md#is-any-part-of-your-implementation-intended-to-conform-to-any-eips-if-yes-please-list-the-contracts-in-this-format).

### Proof Of Concept

Some extra integrations are being queried on some ERC20 tokens to be integrated, which are not a part of the [ERC-20 standard](https://eips.ethereum.org/EIPS/eip-20), and was added later as an [optional extension](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/extensions/IERC20Metadata.sol). As such, it is unsafe to blindly cast all tokens to this interface, and then call this function.

Using these 3 search commands:

- https://github.com/search?q=repo%3Acode-423n4%2F2023-09-maia+.decimals&type=code
- https://github.com/search?q=repo%3Acode-423n4%2F2023-09-maia+.name%28&type=code
- https://github.com/search?q=repo%3Acode-423n4%2F2023-09-maia+.symbol&type=code

From the abive we can see that there are 12 instances of attempts to query protperties that re not generic to the EIP 20 specification.

Additionally, note that [this section of the contest's ReadME](https://github.com/code-423n4/2023-09-maia/blob/f5ba4de628836b2a29f9b5fff59499690008c463/README.md#is-any-part-of-your-implementation-intended-to-conform-to-any-eips-if-yes-please-list-the-contracts-in-this-format) iterates the fact that some contracts must be in compliance with EIP 20: `ERC20hTokenBranch & ERC20hTokenRoot` but as explainred below this is not the case.

> - `ERC20hTokenBranch`: Should comply with `ERC20/EIP20`
> - `ERC20hTokenRoot`: Should comply with `ERC20/EIP20`

### Recommended Mitigation Steps

Either correctly mitigate this, or clearly document that tokens that are purely following the standard might have compatibility issues while integrating the sytem in the case that they do not include this external integrations.

## QA-14 Owner currently can't execute any extra calldata that's needed in the case where `_payload.length > PARAMS_SETTLEMENT_OFFSET`

### Impact

Low.. contract curretly not working as specified

### Proof Of Concept

Take a look at `executeWithSettlement()`

```solidity
     * @notice Function to execute a cross-chain request with a single settlement.
     * @param _recipient Address of the recipient of the settlement.
     * @param _router Address of the router contract to execute the request.
     * @param _payload Data received from the messaging layer.
     * @dev Router is responsible for managing the msg.value either using it for more remote calls or sending to user.
     * @dev SETTLEMENT FLAG: 1 (Single Settlement)
     */
    function executeWithSettlement(address _recipient, address _router, bytes calldata _payload)
        external
        payable
        onlyOwner
    {
        // Clear Token / Execute Settlement
        SettlementParams memory sParams = SettlementParams({
            settlementNonce: uint32(bytes4(_payload[PARAMS_START_SIGNED:PARAMS_TKN_START_SIGNED])),
            recipient: _recipient,
            hToken: address(uint160(bytes20(_payload[PARAMS_TKN_START_SIGNED:45]))),
            token: address(uint160(bytes20(_payload[45:65]))),
            amount: uint256(bytes32(_payload[65:97])),
            deposit: uint256(bytes32(_payload[97:PARAMS_SETTLEMENT_OFFSET]))
        });

        // Bridge In Assets
        BranchBridgeAgent(payable(msg.sender)).clearToken(
            sParams.recipient, sParams.hToken, sParams.token, sParams.amount, sParams.deposit
        );

        // Execute Calldata if there is any
        if (_payload.length > PARAMS_SETTLEMENT_OFFSET) {
            // Execute remote request
     //@audit-issue owner can't execute any extra calldata that's needed in the case where `_payload.length > PARAMS_SETTLEMENT_OFFSET` this si cause the `executeSettlement()` router's function currently has no implementation and would only reverrt
            IRouter(_router).executeSettlement{value: msg.value}(_payload[PARAMS_SETTLEMENT_OFFSET:], sParams);
        } else {
            // Send reamininig native / gas token to recipient
            _recipient.safeTransferETH(address(this).balance);
        }
    }
```

As seen at the core end of the function, in a case where `_payload.length > PARAMS_SETTLEMENT_OFFSET` is true, a query is to be made to the `executeSettlement()` function, but this currently is unimplemented and instead reverts any call made to it, this can be seen [here]()

```solidity
    /// @inheritdoc IBranchRouter
    function executeSettlement(bytes calldata, SettlementParams memory)
        external
        payable
        virtual
        override
        requiresAgentExecutor
    {
        revert UnrecognizedFunctionId();
    }
```

### Recommended Mitigation Steps

Implement the `executeSettlement()` fucntion or cleardly document that for this to be accessble it needs to be properly implemented.
