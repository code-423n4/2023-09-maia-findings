# [L-1] Nonce is too low

In the protocol the `depositNonce` and `settlementNonce` in bridgeAgent are set to `uint32`. The maximum of those numbers are only 2^32 ~ 4 bil. It is too low for a protocol that is going to have a lot of bridge transactions. The transactions are going to be revert when the nonces are overflow.

https://github.com/code-423n4/2023-09-maia/blob/ffbe532c6f5224a55ce099b4016bd8806bdbc913/src/BranchBridgeAgent.sol#L84
https://github.com/code-423n4/2023-09-maia/blob/ffbe532c6f5224a55ce099b4016bd8806bdbc913/src/RootBridgeAgent.sol#L75

Normally, the nonce should be set to `uint64` which is safe.

# [L-2] Cannot turn on branch bridge agent factory again

When branch bridge agent is disable, it cannot be enable again, the only option is creating a new one.

        // Check if BridgeAgentFactory is active
        if (IPort(_localPortAddress).isBridgeAgentFactory(_newBridgeAgentFactoryAddress)) {
            // If so, disable it.
            IPort(_localPortAddress).toggleBridgeAgentFactory(_newBridgeAgentFactoryAddress);
        } else {
            // If not, add it.
            IPort(_localPortAddress).addBridgeAgentFactory(_newBridgeAgentFactoryAddress);
        }

https://github.com/code-423n4/2023-09-maia/blob/ffbe532c6f5224a55ce099b4016bd8806bdbc913/src/CoreBranchRouter.sol#L249-L255

# [NC-1] Function `setCoreRouter` in BranchPort cannot be called

The function setCoreRouter can only be called by coreRouter. However the contract CoreBranchRouter has no function to call this function.

        function setCoreRouter(address _newCoreRouter) external override requiresCoreRouter {

https://github.com/code-423n4/2023-09-maia/blob/ffbe532c6f5224a55ce099b4016bd8806bdbc913/src/BranchPort.sol#L331-L335

# [NC-2] Wrong natspec

The natspec below is for a different setter function, not the bridgeOut function.

    /**
     * @notice Setter function to decrease local hToken supply.
     *   @param _localAddress token address.
     *   @param _amount amount of tokens.
     *   @param _deposit amount of underlying tokens.
     *
     */
    function bridgeOut(
        address _depositor,
        address _localAddress,
        address _underlyingAddress,
        uint256 _amount,
        uint256 _deposit
    ) external;

https://github.com/code-423n4/2023-09-maia/blob/ffbe532c6f5224a55ce099b4016bd8806bdbc913/src/interfaces/IBranchPort.sol#L111-L124