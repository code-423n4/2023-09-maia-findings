## [L-01] Unsafe ERC20 Operation(s)
## Description
ERC20 operations within smart contracts can be prone to vulnerabilities due to variations in implementations and standard compliance. It is imperative to consider security best practices when handling ERC20 tokens within your smart contracts. Failing to do so can expose your contract to various risks, including race-condition vulnerabilities. To mitigate these risks, it is highly recommended to follow the best practices such as using OpenZeppelin's SafeERC20 library or ensuring that every operation involving ERC20 tokens is wrapped within a `require` statement. Additionally, when dealing with the `approve` function to allow token transfers, consider using OpenZeppelin's SafeERC20 library's `safeIncreaseAllowance` and `safeDecreaseAllowance` functions to prevent potential race-condition vulnerabilities.

## Impact Analysis
### Potential Risks
1. **Race-Condition Vulnerabilities**: Race conditions can occur when multiple users or contracts simultaneously interact with a contract's state. In the case of ERC20 operations, race conditions can lead to unauthorized or unintended token transfers, potentially causing financial losses.

2. **Loss of Funds**: If ERC20 operations are not properly secured, an attacker might exploit vulnerabilities to drain the contract of its token holdings or manipulate token balances.

3. **Reputation Damage**: Security vulnerabilities in smart contracts can harm the reputation of the project or organization behind the contract, leading to a loss of trust from users and investors.

4. **Legal and Ethical Implications**: Failing to protect users' assets can lead to legal and ethical consequences, as it might be considered negligence or misconduct.

### Ethical Example Exploit: Proof of Concept (PoC) for a Solidity Bug
Here's a simplified example of a Solidity bug related to unsafe ERC20 operations:

```solidity
// This is a simplified vulnerable contract
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/IERC20.sol";

contract VulnerableContract {
    IERC20 public token;

    constructor(address _tokenAddress) {
        token = IERC20(_tokenAddress);
    }

    function unsafeTransfer(address _recipient, uint256 _amount) external {
        // Vulnerable code: No require statement, race condition possible
        token.transfer(_recipient, _amount);
    }
}
```

In the above contract, the `unsafeTransfer` function allows anyone to transfer ERC20 tokens without proper checks. An attacker can potentially exploit this vulnerability by reentrancy attacks or race conditions to steal tokens or disrupt the contract's intended behavior.

### Best Practice Example
Here's an example of how to secure ERC20 operations using OpenZeppelin's SafeERC20 library:

```solidity
// This is a secure version of the contract using SafeERC20
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts/token/ERC20/SafeERC20.sol";

contract SecureContract {
    using SafeERC20 for IERC20;

    IERC20 public token;

    constructor(address _tokenAddress) {
        token = IERC20(_tokenAddress);
    }

    function safeTransfer(address _recipient, uint256 _amount) external {
        // Secure code using SafeERC20
        token.safeTransfer(_recipient, _amount);
    }
}
```

By using `SafeERC20`, the contract ensures that ERC20 transfers are conducted securely, reducing the risk of vulnerabilities.

ERC20 operations can be unsafe due to different implementations and vulnerabilities in the standard.
It is therefore recommended to always either use OpenZeppelin's SafeERC20 library or at least to wrap each operation in a require statement.
To circumvent ERC20's approve functions race-condition vulnerability use OpenZeppelin's SafeERC20 library's safe{Increase|Decrease}Allowance functions.
**Bad**
```sol
ERC20(_hToken).approve(_localPortAddress, _amount - _deposit);
ERC721(_token).transferFrom(address(this), msg.sender, _tokenId);
```
**Good**
```sol
ERC20(_hToken).safeIncreaseAllowance(_localPortAddress, _amount - _deposit);
ERC721(_token).safeTransferFrom(address(this), msg.sender, _tokenId);
```
## **References**
```sol
202309-maia/src/BaseBranchRouter.sol::168 => ERC20(_hToken).approve(_localPortAddress, _amount - _deposit);
202309-maia/src/BaseBranchRouter.sol::175 => ERC20(_token).approve(_localPortAddress, _deposit);
202309-maia/src/VirtualAccount.sol::62 => ERC721(_token).transferFrom(address(this), msg.sender, _tokenId);
```
## [L-02] Unspecific Compiler Version Pragma
Contracts should be deployed with the same compiler version and flags that they have been tested with thoroughly. Locking the pragma helps to ensure that contracts do not accidentally get deployed using, for example, an outdated compiler version that might introduce bugs that affect the contract system negatively.
**Bad**
```sol
pragma solidity ^0.8.0;
```
**Good**
```sol
pragma solidity 0.8.16;
```
**Mitigation**
Remediation
Lock the pragma version and also consider known bugs (https://github.com/ethereum/solidity/releases) for the compiler version that is chosen.
Pragma statements can be allowed to float when a contract is intended for consumption by other developers, as in the case with contracts in a library or EthPM package. 
Otherwise, the developer would need to manually update the pragma in order to compile locally.
## References
```sol
202309-maia/src/ArbitrumBranchBridgeAgent.sol::2 => pragma solidity ^0.8.0;
202309-maia/src/ArbitrumBranchPort.sol::3 => pragma solidity ^0.8.0;
202309-maia/src/ArbitrumCoreBranchRouter.sol::2 => pragma solidity ^0.8.0;
202309-maia/src/BaseBranchRouter.sol::2 => pragma solidity ^0.8.0;
202309-maia/src/BranchBridgeAgent.sol::2 => pragma solidity ^0.8.0;
202309-maia/src/BranchBridgeAgentExecutor.sol::2 => pragma solidity ^0.8.0;
202309-maia/src/BranchPort.sol::3 => pragma solidity ^0.8.0;
202309-maia/src/CoreBranchRouter.sol::2 => pragma solidity ^0.8.0;
202309-maia/src/CoreRootRouter.sol::2 => pragma solidity ^0.8.0;
202309-maia/src/MulticallRootRouter.sol::2 => pragma solidity ^0.8.0;
202309-maia/src/MulticallRootRouterLibZip.sol::2 => pragma solidity ^0.8.0;
202309-maia/src/RootBridgeAgent.sol::2 => pragma solidity ^0.8.0;
202309-maia/src/RootBridgeAgentExecutor.sol::2 => pragma solidity ^0.8.0;
202309-maia/src/RootPort.sol::3 => pragma solidity ^0.8.0;
202309-maia/src/VirtualAccount.sol::3 => pragma solidity ^0.8.0;
202309-maia/src/factories/ArbitrumBranchBridgeAgentFactory.sol::2 => pragma solidity ^0.8.0;
202309-maia/src/factories/BranchBridgeAgentFactory.sol::2 => pragma solidity ^0.8.0;
202309-maia/src/factories/ERC20hTokenBranchFactory.sol::2 => pragma solidity ^0.8.0;
202309-maia/src/factories/ERC20hTokenRootFactory.sol::2 => pragma solidity ^0.8.0;
202309-maia/src/factories/RootBridgeAgentFactory.sol::2 => pragma solidity ^0.8.0;
202309-maia/src/interfaces/BridgeAgentConstants.sol::2 => pragma solidity ^0.8.0;
202309-maia/src/interfaces/BridgeAgentStructs.sol::2 => pragma solidity ^0.8.0;
202309-maia/src/interfaces/IArbitrumBranchPort.sol::3 => pragma solidity ^0.8.0;
202309-maia/src/interfaces/IBranchBridgeAgent.sol::2 => pragma solidity ^0.8.0;
202309-maia/src/interfaces/IBranchBridgeAgentFactory.sol::2 => pragma solidity ^0.8.0;
202309-maia/src/interfaces/IBranchPort.sol::3 => pragma solidity ^0.8.0;
202309-maia/src/interfaces/IBranchRouter.sol::2 => pragma solidity ^0.8.0;
202309-maia/src/interfaces/ICoreBranchRouter.sol::2 => pragma solidity ^0.8.0;
202309-maia/src/interfaces/IERC20hTokenBranch.sol::2 => pragma solidity ^0.8.0;
202309-maia/src/interfaces/IERC20hTokenBranchFactory.sol::2 => pragma solidity ^0.8.0;
202309-maia/src/interfaces/IERC20hTokenRoot.sol::2 => pragma solidity ^0.8.0;
202309-maia/src/interfaces/IERC20hTokenRootFactory.sol::2 => pragma solidity ^0.8.0;
202309-maia/src/interfaces/ILayerZeroEndpoint.sol::3 => pragma solidity >=0.5.0;
202309-maia/src/interfaces/ILayerZeroReceiver.sol::3 => pragma solidity >=0.5.0;
202309-maia/src/interfaces/ILayerZeroUserApplicationConfig.sol::3 => pragma solidity >=0.5.0;
202309-maia/src/interfaces/IMulticall2.sol::2 => pragma solidity ^0.8.0;
202309-maia/src/interfaces/IPortStrategy.sol::3 => pragma solidity ^0.8.0;
202309-maia/src/interfaces/IRootBridgeAgent.sol::2 => pragma solidity ^0.8.0;
202309-maia/src/interfaces/IRootBridgeAgentFactory.sol::2 => pragma solidity ^0.8.0;
202309-maia/src/interfaces/IRootPort.sol::3 => pragma solidity ^0.8.0;
202309-maia/src/interfaces/IRootRouter.sol::2 => pragma solidity ^0.8.0;
202309-maia/src/interfaces/IVirtualAccount.sol::2 => pragma solidity ^0.8.0;
202309-maia/src/token/ERC20hTokenBranch.sol::2 => pragma solidity ^0.8.0;
202309-maia/src/token/ERC20hTokenRoot.sol::2 => pragma solidity ^0.8.0;
```
## [L-03] Do not use Deprecated Library Functions
The usage of deprecated library functions should be discouraged.
This issue is mostly related to OpenZeppelin libraries.
**Bad**
```sol
outputToken.safeApprove(_bridgeAgentAddress, amountOut);
```
**Good**
```sol
outputToken.safeDecreaseAllowance(_bridgeAgentAddress, amountOut);
```
## Security Impact

### 1. Deprecated Library Functions

Using deprecated library functions can introduce significant security risks to smart contracts. Deprecated functions are those that have been marked as outdated and discouraged from use because they may contain vulnerabilities or other issues. When developers continue to use these functions, they expose their contracts to potential exploits and vulnerabilities. 

### 2. Impact on Contracts

The impact of using deprecated library functions on smart contracts can be severe:

#### a. Vulnerabilities

Deprecated functions are often deprecated for a reason – they may contain security vulnerabilities that can be exploited by malicious actors. In the provided example, using `safeApprove` is discouraged, and its continued use can result in potential vulnerabilities.

#### b. Loss of Funds

One of the most significant risks associated with deprecated functions is the potential loss of funds. If a vulnerability exists in a deprecated function, attackers can manipulate the contract's behavior to drain funds, leading to financial losses for contract users.

#### c. Reputation Damage

Contracts that use deprecated functions may damage the reputation of the project and its developers. Investors and users may lose trust in the project if they perceive that security best practices are not followed.

## Exploit Proof of Concept (PoC)

To illustrate the potential impact of using deprecated library functions, let's provide a simple PoC of a Solidity bug related to the usage of `safeApprove`:

```solidity
// This is a simplified example to demonstrate the issue.

pragma solidity ^0.8.0;

import "./IERC20.sol";

contract MaliciousContract {
    address public targetToken;
    address public attacker;

    constructor(address _targetToken, address _attacker) {
        targetToken = _targetToken;
        attacker = _attacker;
    }

    // Exploitable function using deprecated 'safeApprove'
    function exploit() public {
        IERC20(targetToken).approve(attacker, 1000000); // Deprecated function
    }
}
```

In this PoC, a malicious contract uses the deprecated `approve` function, which exposes the target contract to a potential attack. An attacker could deploy this malicious contract and exploit it to gain control over the target token's allowance, potentially leading to unauthorized fund transfers.

## References
```sol
202309-maia/src/MulticallRootRouter.sol::521 => outputToken.safeApprove(_bridgeAgentAddress, amountOut);
202309-maia/src/MulticallRootRouter.sol::559 => outputTokens[i].safeApprove(_bridgeAgentAddress, amountsOut[i]);
```