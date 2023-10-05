**src/CoreBranchRouter.sol**
- L31 - In the constructor, a storage variable is set, hTokenFactoryAddress, which is immutable, but it is not validated at any time that the address is != 0x, therefore it is important to add that validation in the constructor.

**src/factories/ArbitrumBranchBridgeAgentFactory.sol**
- L88 - It is an unnecessary recheck _rootBridgeAgentFactoryAddress == rootBridgeAgentFactoryAddress, which the user is asked for as input, generating unnecessary annoyance to the user.

**src/factories/BranchBridgeAgentFactory.sol**
- L124 - It is an unnecessary recheck _rootBridgeAgentFactoryAddress == rootBridgeAgentFactoryAddress, which the user is asked for as input, generating unnecessary annoyance to the user.

**src/factories/RootBridgeAgentFactory.sol**
- L35 - The immutable variable lzEndpointAddress is created, but when it is set in the constructor it is not validated that the address is != 0x.

**src/BranchBridgeAgent.sol**
- L587-701 - The lzReceiveNonBlocking function has more than 100 lines of code and contains multiple ifs, therefore it would be beneficial to use auxiliary functions to simplify the code and facilitate understanding.

**src/VirtualAccount.sol**
- L36/37 - In the constructor, two storage variables are set, userAddress and localPortAddress, which are immutable, but it is not validated at any time that the addresses are != 0x, therefore it is important to add these validations in the constructor.

**src/CoreRootRouter.sol**
- L73 - In the constructor, a storage variable is set, rootPortAddress, which is immutable, but it is not validated at any time that the address is != 0x, therefore it is important to add that validation in the constructor.
