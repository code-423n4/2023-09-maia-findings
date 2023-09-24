Redundant Parameters

The `address _depositor` and `address _recipient` can use the same argument, thus create one variable will do. There isn't a need to create two arguments.


```
    function depositToPort(address underlyingAddress, uint256 amount) external payable lock {
        IArbPort(localPortAddress).depositToPort(msg.sender, msg.sender, underlyingAddress, amount);
    }
```
https://github.com/code-423n4/2023-09-maia/blob/c0dc3550e0754571b82d7bfd8f0282ac8fa5e42f/src/ArbitrumBranchBridgeAgent.sol#L69C1-L71C6

```
    function withdrawFromPort(address localAddress, uint256 amount) external payable lock {
        IArbPort(localPortAddress).withdrawFromPort(msg.sender, msg.sender, localAddress, amount);
    }
```
https://github.com/code-423n4/2023-09-maia/blob/c0dc3550e0754571b82d7bfd8f0282ac8fa5e42f/src/ArbitrumBranchBridgeAgent.sol#L79C1-L81C6

```
    function depositToPort(address _depositor, address _recipient, address _underlyingAddress, uint256 _deposit)
        external
        override
        lock
        requiresBridgeAgent
    {
        // Save root port address to memory
        address _rootPortAddress = rootPortAddress;

        // Get global token address from root port
        address _globalToken = IRootPort(_rootPortAddress).getLocalTokenFromUnderlying(_underlyingAddress, localChainId);

        // Check if the global token exists
        if (_globalToken == address(0)) revert UnknownGlobalToken();

        // Deposit Assets to Port
        _underlyingAddress.safeTransferFrom(_depositor, address(this), _deposit);

        // Request Minting of Global Token
        IRootPort(_rootPortAddress).mintToLocalBranch(_recipient, _globalToken, _deposit);
    }
```

https://github.com/code-423n4/2023-09-maia/blob/c0dc3550e0754571b82d7bfd8f0282ac8fa5e42f/src/ArbitrumBranchPort.sol#L50C1-L70C6

```
    function withdrawFromPort(address _depositor, address _recipient, address _globalAddress, uint256 _amount)
        external
        override
        lock
        requiresBridgeAgent
    {
        // Save root port address to memory
        address _rootPortAddress = rootPortAddress;


        // Check if the global token exists
        if (!IRootPort(_rootPortAddress).isGlobalToken(_globalAddress, localChainId)) revert UnknownGlobalToken();


        // Get the underlying token address from the root port
        address _underlyingAddress =
            IRootPort(_rootPortAddress).getUnderlyingTokenFromLocal(_globalAddress, localChainId);


        // Check if the underlying token exists
        if (_underlyingAddress == address(0)) revert UnknownUnderlyingToken();


        IRootPort(_rootPortAddress).burnFromLocalBranch(_depositor, _globalAddress, _amount);


        _underlyingAddress.safeTransfer(_recipient, _amount);
    }
```

https://github.com/code-423n4/2023-09-maia/blob/c0dc3550e0754571b82d7bfd8f0282ac8fa5e42f/src/ArbitrumBranchPort.sol#L73C1-L95C6