# 类型

Solidity 是一种静态类型语言，意味着每个变量都要在编译时指定变量类型。Solidity 提供了几种基本类型，并且可以用基本类型来组合出复杂类型。

除此之外，类型之间可以在包含运算符的表达式中进行交互。`undefined`和`null`在 Solidity 中不存在，但是新声明的变量总是有一个**默认值**，具体的默认值跟**类型相关**。要处理任何意外的值，应该使用**错误恢复**来恢复整个交易，或者返回一个带有第二个`bool`的元组表示成功。

## 值类型

以下类型也称为值类型，因为这些类型的变量将始终按照值传递，也就是说，当这些变量被用作函数参数或者赋值语句中时，总是进行**值拷贝**。

- 布尔类型 `bool`
- 整型 `int`/`uint`。分别表示有符号和无符号的不同位数的整型变量。支持关键字`uint8`到`uint256`（包括`int8-256`），以`8`位为步长递增。`uint`和`int`分别是`uint256`和`int256`。对于整型`X`，可以用`type(x).min,type(x).max`来获取该类型的最小值与最大值。
- 定长浮点型：`fixed/ufixed`。表示各种大小的有符号和无符号的定长浮点型。在关键字`ufixedMxN`和`fixedMxN`中，`M`表示该类型占用的位数，`N`表示可用的小数位数，`M`必须能够整除 8，即 8 到 256 位。N 可以是从 0 到 80 的任意数。`ufixed`和`fixed`分别是`ufixed128x19`和`fixed128x19`。
- 地址类型，地址类型有两种形式，它们大致相同：
    - `address`: 保存一个 20 字节的值（以太坊地址大小）
    - `address payable`: 可支付地址，与`address`相同，不过有成员函数`transfer`和`send`
    - 两者的区别主要在于后者可以接受以太币的地址，而一个普通的`address`不能。

### 地址类型成员变量
`balance`和`transfer`，可以使用`balance`属性查询一个地址的余额，也可以用`transfer`函数向一个可支付地址`payable address`发送以太币。
```solidity
address x = 0x123;
address myAddress = this;
if (x.balance < 10 && myAddress.balance >= 10) x.transfer(10);
```

`send`是`transfer`的低级版本，如果执行失败，当前的合约不会因为异常而终止，但是`send`会返回`false`。

`call`，`delegatecall`，`staticcall`为了与不符合**应用二进制接口**的合约交互，或者要更直接地控制编码，提供了这三个函数，它们都带有一个`bytes memory`参数和返回执行成功状态`bool`和数据`bytes memory`。

函数`abi.encode, abi.encodePacked, abi.encodeWithSelector, abi.encodeWithSignature`可以用来编码结构化数据。
```solidity
bytes memory payload = abi.encodeWithSignature("register(string)", "MyName");
(bool success, bytes memory returnData) = address(nameReg).call(payload);
require(success);
```
此外，为了与不符合应用二进制接口的合约交互，于是就有了可以接受任意类型任意数量参数的`call`函数，这些参数会被打包到以 32 字节为单位的连续区域中存放。其中一个例外是当第一个参数被编码成正好 4 个字节的情况，这种情况下， 这个参数后面不会填充后续参数编码，以允许使用函数签名。
```solidity
address nameReg = 0x12345;
nameReg.call('register', 'MyName')
nameReg.call(bytes4(kecaak256("fun(uint256)")), a)
```
**注意**，所有这三个函数都是低级函数，应该谨慎使用。具体来说，任何未知的合约可能都是恶意的，我们在调用一个合约的同时就将控制权交给了它，而合约又可以调用合约，所以要准备好在调用返回时改变相对应的状态变量。

## 合约类型
每个`contract`定义都有它自己的类型。

你可以隐式地将合约转换为从他们继承地合约，合约可以显式转换为`address`类型。只有当合约具有接受`receive`函数或`payable`回退函数时，才能显式和`address payable`转换，转换仍然使用`address(x)`执行。

## 定长字节数组
关键字有：`bytes1`,`bytes2`...`bytes32`.

## 变长字节数组
- `bytes` 变长字节数组，并不是值类型。
- `string`变长 UTF-8 编码字符串类型，并不是值类型。

。。。