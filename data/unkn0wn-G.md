### Use != 0 instead of > 0 for Unsigned Integer Comparison

#### Impact
Using > 0 uses slightly more gas than using != 0. Use != 0 when comparing uint variables to zero, which cannot hold values below zero

#### Findings:
```
src/ArbitrumBranchPort.sol::127 => if (_deposit > 0) {
src/ArbitrumBranchPort.sol::132 => if (_amount - _deposit > 0) {
src/BaseBranchRouter.sol::165 => if (_amount - _deposit > 0) {
src/BaseBranchRouter.sol::173 => if (_deposit > 0) {
src/BranchBridgeAgent.sol::906 => if (_amount - _deposit > 0) {
src/BranchBridgeAgent.sol::912 => if (_deposit > 0) {
src/BranchPort.sol::259 => if (_amounts[i] - _deposits[i] > 0) {
src/BranchPort.sol::266 => if (_deposits[i] > 0) {
src/BranchPort.sol::525 => if (_hTokenAmount > 0) {
src/BranchPort.sol::531 => if (_deposit > 0) {
src/RootBridgeAgent.sol::363 => if (_dParams.amount > 0) {
src/RootBridgeAgent.sol::370 => if (_dParams.deposit > 0) {
src/RootBridgeAgent.sol::1146 => if (_underlyingAddress == address(0)) if (_deposit > 0) revert UnrecognizedUnderlyingAddress();
src/RootBridgeAgent.sol::1149 => if (_amount - _deposit > 0) {
src/RootBridgeAgent.sol::1156 => if (_deposit > 0) {
src/RootPort.sol::284 => if (_amount - _deposit > 0) {
src/RootPort.sol::290 => if (_deposit > 0) if (!ERC20hTokenRoot(_hToken).mint(_recipient, _deposit, _srcChainId)) revert UnableToMint();
```
### Recommended Mitigation Steps
Replace > 0 with != 0 to save gas.