### According to dev `_rootChainId == _localChainId` but here it uses `||` operator so only one case should be true to pass the statement
Recommendation: Use && instead of || in constructor
1. BranchBridgeAgent.sol
https://github.com/code-423n4/2023-09-maia/blob/main/src/BranchBridgeAgent.sol#L127
```
Logs:
  _rootChainId: 1
  _localChainId: 10

Traces:
  [4066934] PrimeTest::setUp()
    ├─ [0] VM::addr(<pk>) [staticcall]
    │   └─ ← _rootBridgeAgentAddress: [0x51519e34B07B635a38E93d37c1D8Bd5348B8C5fd]
    ├─ [0] VM::label(_rootBridgeAgentAddress: [0x51519e34B07B635a38E93d37c1D8Bd5348B8C5fd], _rootBridgeAgentAddress)
    │   └─ ← ()
    ├─ [0] VM::addr(<pk>) [staticcall]
    │   └─ ← _rootBridgeAgentAddress: [0x51519e34B07B635a38E93d37c1D8Bd5348B8C5fd]
    ├─ [0] VM::label(_rootBridgeAgentAddress: [0x51519e34B07B635a38E93d37c1D8Bd5348B8C5fd], _rootBridgeAgentAddress)
    │   └─ ← ()
    ├─ [0] VM::addr(<pk>) [staticcall]
    │   └─ ← _rootBridgeAgentAddress: [0x51519e34B07B635a38E93d37c1D8Bd5348B8C5fd]
    ├─ [0] VM::label(_rootBridgeAgentAddress: [0x51519e34B07B635a38E93d37c1D8Bd5348B8C5fd], _rootBridgeAgentAddress)
    │   └─ ← ()
    ├─ [0] VM::addr(<pk>) [staticcall]
    │   └─ ← _rootBridgeAgentAddress: [0x51519e34B07B635a38E93d37c1D8Bd5348B8C5fd]
    ├─ [0] VM::label(_rootBridgeAgentAddress: [0x51519e34B07B635a38E93d37c1D8Bd5348B8C5fd], _rootBridgeAgentAddress)
    │   └─ ← ()
    ├─ [3995527] → new BranchBridgeAgent@0xFEfC6BAF87cF3684058D62Da40Ff3A795946Ab06
    │   ├─ [772864] DeployBranchBridgeAgentExecutor::deploy() [delegatecall]
    │   │   ├─ [739832] → new BranchBridgeAgentExecutor@0x6d2eed85750d316088343D6d5e91ca59eb052768
    │   │   │   ├─ emit OwnershipTransferred(oldOwner: 0x0000000000000000000000000000000000000000, newOwner: BranchBridgeAgent: [0xFEfC6BAF87cF3684058D62Da40Ff3A795946Ab06])
    │   │   │   └─ ← 3577 bytes of code
    │   │   └─ ← 0x0000000000000000000000006d2eed85750d316088343d6d5e91ca59eb052768
    │   └─ ← 15518 bytes of code
    ├─ [0] console::log(_rootChainId:, 1) [staticcall]
    │   └─ ← ()
    ├─ [0] console::log(_localChainId:, 10) [staticcall]
    │   └─ ← ()
    └─ ← ()
```

2. BranchBridgeAgentFactory.sol
https://github.com/code-423n4/2023-09-maia/blob/main/src/factories/BranchBridgeAgentFactory.sol#L63
```
Logs:
  _rootChainId: 10
  _localChainId: 1

Traces:
  [634461] PrimeTest::setUp()
    ├─ [0] VM::addr(<pk>) [staticcall]
    │   └─ ← _rootBridgeAgentFactoryAddress: [0x3A7888ba2A9eBFe25F7811fbabfce9199c1f5C3F]
    ├─ [0] VM::label(_rootBridgeAgentFactoryAddress: [0x3A7888ba2A9eBFe25F7811fbabfce9199c1f5C3F], _rootBridgeAgentFactoryAddress)
    │   └─ ← ()
    ├─ [0] VM::addr(<pk>) [staticcall]
    │   └─ ← _lzEndpointAddress: [0xB3D9169d14109028BAf801ad61D0060e92b2Bc30]
    ├─ [0] VM::label(_lzEndpointAddress: [0xB3D9169d14109028BAf801ad61D0060e92b2Bc30], _lzEndpointAddress)
    │   └─ ← ()
    ├─ [0] VM::addr(<pk>) [staticcall]
    │   └─ ← _localCoreBranchRouterAddress: [0x51b8f4de3EC6fd451b5A4FC2865c9165896C2373]
    ├─ [0] VM::label(_localCoreBranchRouterAddress: [0x51b8f4de3EC6fd451b5A4FC2865c9165896C2373], _localCoreBranchRouterAddress)
    │   └─ ← ()
    ├─ [0] VM::addr(<pk>) [staticcall]
    │   └─ ← _localPortAddress: [0xb37E923ddfD7c128656A1830b7F09E63de498C64]
    ├─ [0] VM::label(_localPortAddress: [0xb37E923ddfD7c128656A1830b7F09E63de498C64], _localPortAddress)
    │   └─ ← ()
    ├─ [0] VM::addr(<pk>) [staticcall]
    │   └─ ← _owner: [0xC1Bc7C191295820a8D8040f9c51C52C88097c819]
    ├─ [0] VM::label(_owner: [0xC1Bc7C191295820a8D8040f9c51C52C88097c819], _owner)
    │   └─ ← ()
    ├─ [564672] → new BranchBridgeAgentFactory@0xd04404bcf6d969FC0Ec22021b4736510CAcec492
    │   ├─ emit OwnershipTransferred(oldOwner: 0x0000000000000000000000000000000000000000, newOwner: _owner: [0xC1Bc7C191295820a8D8040f9c51C52C88097c819])
    │   └─ ← 2695 bytes of code
    ├─ [0] console::log(_rootChainId:, 10) [staticcall]
    │   └─ ← ()
    ├─ [0] console::log(_localChainId:, 1) [staticcall]
    │   └─ ← ()
    └─ ← ()
```

