# 合约1

Solidity 合约类似面向对象语言的类，合约中有用于持久化的状态变量，和可以修改状态变量的函数。调用另一个合约实例的函数时，会执行一个 EVM 调用，这个操作会切换执行时的上下文。这样，前一个合约的状态变量就不能访问了。

## 创建合约

可以通过以太坊交易“从外部”或者从 Solidity 合约内部创建合约。

一些集成开发环境，如 Remix，通过使用一些用户界面元素使创建过程更加流畅。在以太坊上编程创建合约最好使用 web3.js（web3.eth.Contract）

创建合约时，会执行一次构造函数，构造函数是可选的，**但是并不支持重载，只允许有一个构造函数**

在内部，构造函数参数在合约代码之后通过 ABI 编码传送，如果你使用 web3.js 可以不用关心这个问题。

如果一个合约想要创建另一个合约，那么创建者必须知道被创建合约的源代码和二进制代码，这意味着不能循环创建依赖项。

## 可见性和 getter

由于 Solidity 有两种函数调用，内部调用不会产生实际的 EVM 调用（“消息调用”），而外部调用则会产生一个 EVM 调用。函数和状态变量有四种可见类型，默认情况下函数类型为`public`，对于状态变量，不能设置为`external`，默认为`internal`
- `external`: 外部函数作为合约接口的一部分，意味着我们可以从其他合约和交易中调用。当收到大量数据的时候，外部函数有时候更有效率。 
- `public`: 合约接口的一部分，可以在内部或者通过消息调用。对于公共状态变量会自动生成一个`getter`函数
- `internal`:  函数和变量只能从内部访问（从当前合约或派生合约访问）
- `private`: 只能从定义函数和变量的合约中访问，并且不能被派生合约使用

可见性标识符的定义位置，对于状态变量是在类型后面，对于函数是在参数列表和返回关键字中间。
```solidity
pragma solidity ^0.4.16;
contract C {
    function f(uint a) private pure returns (uint b) {return a + 1;}
    function setData(uint a) internal {data = a; }
    uint public data;
}
```

## Getter 函数

编译器自动为所有`public`状态变量创建 getter 函数，对于下面给出的合约，编译器会生成一个名为`data`的函数：
```solidity
pragma solidity ^0.4.0;
contract C {
    uint public data = 42;
}
contract Caller {
    C c = new C();
    function call() public {
        uint local = c.data;
    }
}
```
`getter`函数具有外部可见性，如果在内部访问`getter`，他被认为一个状态变量，如果它是外部访问的，即用`this.`，被认为是一个函数。

## 函数修饰器

使用修饰器可以更加轻松地改变函数的行为，例如它们可以在函数执行之前检查某些条件。修饰器是合约的可继承属性，并可能被派生合约覆盖。
```solidity
pragma solidity ^0.4.11;
contract owned {
    function owned() public {owner = msg.sender;}
    address owner;

    // 该合约定义了一个修饰器但是并未使用，将会在派生合约中使用
    // 修饰器所修饰的函数体将会被插入到特殊符号 _; 的位置
    // 这意味着如果 owner 调用这个函数，则函数被执行，否则抛出异常
    modifier onlyOwner {
        require(msg.sender == owner);
        _;
    }
}

contract mortal is owned {
    // 该合约从 owned 中继承了 onlyOwner 修饰符，并将其用于 close 函数
    function close() public onlyOwner {
        selfdestruct(owner);
    }
}

contract priced {
    // 修改器可以接收参数
    modifier costs(uint price) {
        if (msg.value >= price) {
            _;
        }
    }
}
contract Register is priced, owned {
    mapping (address => bool) registeredAddresses;
    uint price;
    function Register(uint initPrice) public {price = initPrice;}

    function register() public payable costs(price) {
        registeredAddresses[msg.sender] = true;
    }
    function changePrice(uint _price) public onlyOwner {
        price = _price;
    }
}
```

> 如果函数有多个修饰器，它们之间以**空格**隔开，修饰器依次检查执行。
