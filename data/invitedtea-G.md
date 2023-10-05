The gas optimization in the code of [Executor.sol at line 473](https://github.com/code-423n4/2023-10-zksync/blob/main/code/contracts/ethereum/contracts/zksync/facets/Executor.sol#L473) is that the `_checkBit()` and `_setBit()` functions are not gas efficient. These functions iterate over all the bits in the bitMap, which is a waste of gas. A more efficient way to do this would be to use bitwise operators, such as `&` and `|`. 

For example, the following code would be more efficient:
```solidity
function _checkBit(uint256 _bitMap, uint8 _index) internal pure returns (bool) {
    return (_bitMap & (1 << _index)) != 0;
     return (_bitMap >> _index) & 1 == 1;
}

function _setBit(uint256 _bitMap, uint8 _index) internal pure returns (uint256) {
    return _bitMap | (1 << _index);
}
```