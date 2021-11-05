# 特殊变量和函数

## 特殊变量

在全局命名空间中已经存在了一些特殊地函数和变量，它们主要用来提供关于区块链的信息或一些通用的工具函数。

> 可以将其理解为 Solidity 原生 API

### 区块和交易属性
- `blockhash(uint blockNumber) returns (bytes32)`: 指定区块的区块哈希，仅可用于最新的 256 个区块且不包括当前区块
- `block.chainid(uint)`: 当前链 id
- `block.coinbase(address)`: 挖出当前区块的矿工地址
- `block.difficulty(uint)`: 当前区块难度
- `block.gaslimit(uint)`: 当前区块 gas 限额
- `block.number（uint)`: 当前区块号
- `block.timestamp(uint)`: 自 Unix epoch 起始当前区块以秒计的时间戳
- `gasleft() returns (uint256)`: 剩余的 gas
- `msg.data(bytes)`: 完整的 calldata
- `msg.sender(address)`: 消息发送者（当前调用）
- `msg.sig(bytes4)`: calldata 的前 4 字节（函数标识符）
- `msg.value(uint)`: 随消息发送 wei 数量
- `tx.gasprice(uint)`: 交易的 gas 价格
- `tx.origin(address payable)`: 交易发起者（完全的调用链） 

## 错误处理

- `assert(bool condition)`: 如果条件不满足，会导致 Panic 错误，则撤销状态更改——用于检查内部错误
- `require(bool condition)`: 如果条件不满足则撤销状态更改
- `require(bool condition, string memory message)`: 如果条件不满足则撤销状态更改，用于检查由输入或外部组件引起的错误，可以同时提供一个错误消息
- `revert()`: 终止运行并撤销更改
- `revert(string memory reason)`: 终止运行并撤销状态更改，同时提供一个解释性字符串

## 数学和密码学函数

- `addmod(uinx x, uint y, uint k) returns (uint)`，计算 `(x + y) % k`
- `keccak256(bytes memory) returns (bytes32)` 计算 Keccak-256 哈希
- `sha256(bytes memory) returns (bytes32)` 计算参数的 SHA-256 哈希
- 其他

