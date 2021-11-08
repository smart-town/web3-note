# 合约结构

Solidity 中，合约类似于其他语言中的“类”。

每个合约中可以包含状态变量、变量、函数、事件 Event、结构体、枚举类型的声明，且合约可以从其他合约继承。

还有一些特殊的合约，如 库和接口。

## 状态变量

状态变量是永久地存储在合约存储中的值。

## 函数

函数是代码的可见单元，函数通常在合约内部定义，但是也可以在合约外部定义。
```solidity
// SPDX-License-Identifier: MIT
pragma solidity >0.7.0 <0.9.0;
contract TinyAction {
    function Mybid() public payable {
        // ...
    }
}
function helper(uint x) pure returns(uint) {
    return x * 2;
}
```
函数调用发生在合约内部或者外部，且函数对其他合约有不同的可见性。函数可以接受参数和返回值。

## 函数修改器

函数修改器可以用来以声明的方式修改函数语义。
```solidity
pragma solidity >= 0.4.22 <0.9.0;
contract MyPurchase {
    address public seller;
    modifier onlySeller() {
        require(msg.sender == seller, "Only seller can call this");
        _;
    }
    function abort() public onlySeller { // 修改器用法
        // ...
    }
}
```

## 事件 Event

事件是能方便调用以太坊虚拟机日志功能的接口。
```solidity
contract TinyAuction {
    event HighestBidIncreased(address binder, uint amount); // 事件
    function bid() public payable {
        emit HighestBidIncreased(msg.sender, msg.value); // 触发事件
    }
}
```

## 结构体

结构体是可以将几个变量分组的自定义类型。
```solidity
contract TinyBallot {
    struct Vote {
        uint weight;
        bool voted;
        address delegate;
        uint vote;
    }
}
```

## 枚举类型

枚举可以用来创建一定数量的“常量值”构成的自定义类型。
```solidity
contract Upchain {
    enum State {Created, Locked, InValid} 枚举
}
```