# Solidity 简介

简单的两个合约示例：

```solidity
// SPDX-License-Identifier: MIT
pragma solidity >= 0.7.0;
contract Store {
    uint storedData;
    function get() public view returns (uint) {
        return storedData;
    }
    function set(uint x) public {
        storedData = x;
    }
}
```
