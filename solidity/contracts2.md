# 合约2

## 函数

### View 函数
将函数声明为`View`类型，这种情况下要保证不修改状态。

以下被认为是修改状态：
1. 修改状态变量
2. 产生事件
3. 创建其他合约
4. 使用`selfdestruct`
5. 通过调用发送以太币
6. 调用任何没有标记为`view`或`pure`的函数
7. 使用低级调用
8. 使用包含特定操作码的内联汇编

```solidity
pragma solidity ^0.4.16;
contract C {
    function f(uint a, uint b) public view returns(uint) {
        return a * (b + 12) + now;
    }
}
```
**注意**：编译器没有强制`view`方法不能修改状态

### Pure 函数
函数可以声明为`pure`，这种情况下，承诺不读取或修改状态

以下被认为从状态中读取：
1. 读取状态变量
2. 访问`this.balance`或`<address>.balance`
3. 访问`block, tx, msg`中任意成员，除`msg.sig`和`msg.data`外
4. 调用任何未标记为`pure`的函数
5. 使用包含某些操作码的内联汇编
```solidity
function f (uint a, uint b) public pure returns (int) {
    return a * (b + 12);
}
```
**注意**：编译器没有强制`pure`方法不能读取状态。

### Fallback 函数
合约可以有一个未命名的函数，这个函数不能有参数也不能有返回值，如果在一个到合约的调用中，没有其他函数与给定的函数标识匹配（或没有提供调用数据），那么这个函数（fallback函数）会被执行。。

除此之外，每当合约收到以太币（没有任何数据），这个函数就会执行。此外为了接收以太币，`fallback`函数必须标记为`payable`，如果不存在这样的函数，则合约不能通过常规交易接收以太币。

在这样的上下文中，通常需要很少的`gas`来完成这个调用，准确来说： 2300gas。所以使得 fallback 函数调用尽量廉价很重要。请注意，调用 fallback 函数的交易（不是内部交易）所需要的 gas 要高很多。因为每次交易都会额外收取 2100 gas 或更多费用，用于签名检查等。

请确保你在部署合约之前彻底测试 fallback 以确保成本低于 2300 gas。

> 一个没有定义 fallback 的合约，直接接收以太币会抛出异常，并返回以太币。所以如果你想合约接收以太币，就必须实现 fallback 函数。

```solidity
pragma solidity ^0.4.0;
contract Test {
    // 发送到这个合约的所有消息都会调用该函数（因为没有其他函数了）
    // 向这个合约发送以太币会导致异常，因为没有 payable 修饰符
    function() public {x = 1;}
    uint x;
}
contract Sink {
    function() public payable {}
}
contract Caller {
    function callTest(Test test) public {
        test.call(0xabdcef); //不存在的哈希
        // 导致 test.x = 1
    }
}
```

### 函数重载
合约可以具有多个不同参数的同名函数，这也适用于继承函数。