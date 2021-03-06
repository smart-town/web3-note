# 概述 

简单的智能合约。

## 存储合约

```solidity
// SPDX-License-Identifier: MIT
pragma solidity >= 0.4.16 <0.9.0

contract SimpleStorage {
    uint storedData;
    function set(uint x) public {
        storedData = x;
    }
    function get() public view returns (uint) {
        return storedData;
    }
}
```

第一行说明源码的版权许可，发布源代码时默认需要。    
下一行告诉编译器所适用的 Solidity 版本。这是为了确保合约不会在新的编译器版本中突然行为异常。

Solidity 中合约的含义就是一组代码（函数）和数据（它的状态），它们位于以太坊区块链的一个特定地址上。    
代码行`uint storedData;`声明一个类型`uint`的状态变量`storedData`。你可以认为它是数据库里的一个位置，可以通过调用管理数据库代码的函数进行查询和变更。对于以太坊来说，上述的合约就是拥有合约`owning contract`。在这种情况下，函数`set()`和`get()`可以用来变更或取出变量的值。

要访问一个状态变量，并不需要`this.`这样的前缀，虽然这是其他语言常见的做法。该合约能够完成的事情并不多：它能够允许任何人在合约中存储一个单独的数字，并且这个数字可以被任何人访问，且没有可行的办法阻止你发布这个数字。当然，任何人都可以再次调用`set()`传入不同的数字，覆盖你的数字，但是这个数字仍会被存储在区块链的历史记录中。

**注意**：所有标识符（合约名称、函数、变量名称）都只能使用`ASCII`字符集。UTF-8 编码数据可以用字符串变量存储。

## 货币合约示例

下面的合约实现了一个简单的加密货币，这里，币的确可以无中生有地产生。但是只有创建合约的人才能做到。而且任何人都可以给其他人转币，不需要注册用户名和密码——所需要的只是以太坊密钥对。
```solidity
// SPDX-License-Identifier: MIT
pragma solidity >=0.7.0 <0.9.0
contract Coin {
    # public 让变量可以从外部读取
    address public minter;
    mapping (address => uint) public balances;

    # 轻客户端可以通过事件针对变化作出高效反应
    event Sent(address from, address to, uint amount);

    # 构造函数，只有合约创建时运行
    constructor() {
        minter = msg.sender;
    }

    function mint(address receiver, uint amount) public {
        require(msg.sender == minter);
        require(amount < 1e60);
        balances[receiver] += amount;
    }

    function send(address receiver, uint amount) public {
        require(amount <= balances[msg.sender], "Insufficient balances.")
        balances[msg.sender] -= amount;
        balances[receiver] += amount;
        emit Sent(msg.sender, receiver, amount);
    }
}
```
这个合约引入了一些新概念。

`address public minter;`这一行声明了一个可以被公开访问的`address`类型的状态变量。`address`类型是一个 160 位的值，且不允许进行任何算数操作。这种类型适合存储合约地址或者外部人员的密钥对。关键字`public`自动生成一个函数，允许你在合约之外访问这个状态变量的当前值。如果没有这个关键字，其他合约没有办法访问这个变量。由编译器生成的函数的代码大致如下：`function minter() external view returns (address) {return minter;}`

下一行`mapping (address => uint) public balances;`也创建了一个公共状态变量，但是它是一个更复杂的变量类型。该类型将`address`映射为无符号整数。`Mappings`可以看作哈希表，它会执行虚拟初始化，以使得所有可能存在的键都映射到一个字节表示全零的值。但是这种类比不够恰当，因为它既不能获得映射的所有键的列表，也不能获得所有值的列表。因此，要么记住你添加到`mappings`中的数据（使用列表或更高级的数据类型可能更好），要么在不需要键列表或值列表的上下文中使用它，就如本例。而由`public`关键字创建的`getter()`函数则是更复杂的情况：
```sol
function balances(address _account) external view returns(uint) {
    return balances[_account];
}
```
正如你所看到的，该函数可以轻松查到账户余额

`event Sent(address from, address to, uint amount)`这行声明了一个所谓的“事件”。它会在`send()`函数的最后一行发出。用户界面（包括服务器应用程序）可以监听区块链上正在发生的事件，而不会花费太多成本，一旦它被发出，监听该事件的`listener`都会收到通知。为了监听这个事件，你可以使用如下 JS 代码（假设 Coin 是通过 web3.js 创建好的合约对象）：
```js
Coin.Sent().watch({}, "", function(error, result) {
    if (!error) {
        console.log(`Coin transfer: ${result.args.amount} were sent from ${result.args.from} to ${result.args.to}.`);
    }
})
```

特殊函数`constructor`是仅在创建合约期间运行的构造函数，不能在事后调用。它永久存储创建合约的人的地址：`msg`以及`tx`和`block`是一个特殊的全局变量，其中包括一些允许访问区块链的属性。`msg.sender`始终是当前（外部）函数调用的来源地址。

