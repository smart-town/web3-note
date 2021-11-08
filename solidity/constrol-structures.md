# 控制结构
JavaScript 中大部分控制结构在 Solidity 中都是可用的。除了`switch`和`goto`。

Solidity 还支持`try/catch`形式的异常处理，但是仅用于**外部函数调用**和**合约创建调用**。

**注意**，与 C 和 JavaScript **不同**，Solidity 中非布尔类型不能转换为布尔类型，因此`if(1){...}`在 Solidity 中无效。

## 函数的输入参数和输出参数
与 JavaScript 一样，函数可能需要参数作为输入；而与 JavaScript 和 C 不同的是，它们可能返回任意数量的参数作为输出。
### 输入参数
输入参数声明方式与变量相同，但是有一个例外，未使用的参数可以省略参数名：
```solidity
pragma solidity ^0.4.16;
contract Simple {
    function taker(uint _a, uint _b) public pure {}
}
```
### 输出参数
输出参数的声明方式在关键词`returns`之后，与输入参数的声明方式相同，如：
```solidity
function cal(uint _a, uint _b) public pure returns (uint o_sum, uint o_product) {
    o_sum = _a + _b;
    o_product = _a * _b;
}
```

## 函数调用

### 内部函数调用
当前合约中的函数可以直接（从“内部”调用），也可以调用：
```solidity
//SPDX-License-Identifier: MIT
pragma solidity >=0.4.22 <0.9.0;
contract C {
    function g(uint a) public pure returns (uint net) {return f();}
    function f() internal pure returns (uint net) {return g(7) + f();}
}
```
这些函数调用在 EVM 中被解释为简单的跳转，这样做的效果就是当前内存不会清除，例如函数之间通过传递内存引用进行内部调用是非常高效的，只能在同一个合约实例的函数可以进行内部调用。

只有同一个合约的函数可以内部调用，仍然应该避免过多的递归调用，因为每个内部函数调用至少使用一个堆栈槽，并且最多有 1024 个可用。

### 外部函数调用
表达式`this.g(8)`和`c.g(2)`，其中`c`是合约实例，也是有效的函数调用。但是这种情况下，函数将会通过一个消息调用来进行“外部调用”，而不是直接的跳转。请注意：不可以在构造函数中通过`this`调用函数，因为此时真正的合约实例还未被创建。

如果想要调用其他合约的函数，需要外部调用。对于一个外部调用，所有的函数参数都需要被复制到内存。当调用其他合约的函数时，随函数调用发送的`wei`和`gas`的数量可以分别由特定选项`.value()`和`.gas()`指定。
```solidity
progma solidity ^0.4.0;
contract InfoFeed {
    function info() public payable returns (uint ret) {return 42;}
}
contract Consumer {
    InfoFeed feed;
    function setFeed(address addr) public {feed = InfoFeed(addr);}
    function callFeed() public {feed.info.value(10).gas(800)();}
}
```
`payable`修饰符用于修饰`info`，否则`value()`将不可用。注意`InfoEdd(addr)`进行了一个显式类型转换。说明“我们知道给定的地址的合约类型是`InfoFeed`”，并且这不会执行构造函数。显式类型转换需要谨慎处理。绝对不要在一个你不清楚类型的合约上执行函数调用。

如果被调用的函数所在合约不存在（也就是账户中不包含代码）或者被调用的合约本身抛出异常或者 gas 用完等，函数调用会抛出异常。

### 具名调用和匿名函数参数
如果被包含在`{}`中，函数调用参数也可以按照任意顺序由名称给出，如：
```solidity
contract C {
    function f(uint key, uint value) public {}
    function g() public {f({value: 2, key: 3});}
}
```
## 通过 new 创建合约
使用关键字`new`可以创建合约，待创建合约的完整代码必须事先知道，因此递归的创建是不可能的。
```solidity
pragma solidity ^0.4.0;
contract D {
    uint x;
    function D(int a) public payable {
        x = a;
    }
}
contract C {
    D d = new D(4);
    function createD(uint arg) public {
        D newD = new D(arg);
    }
    function createAndEndD(uint arg, uint amount) public payable {
        // 随合约创建发送 ether
        D newD = (new D).value(amount)(arg);
    }
}
```
