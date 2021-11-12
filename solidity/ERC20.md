# ERC20 Token

ERC-20 是以太坊最重要的**智能合约标准**之一，它已经成为以太坊区块链上用于可替换通证实现的所有智能合约的标准。

其定义了所有可替换的以太坊通证都应该遵循的通用规则列表。因此该通证标准使所有类型的开发者能够准确预测新通证在更大的以太坊系统中将会如何工作。这简化了开发者的任务。

ERC-20 通证必须实现的一些函数，以实现这些功能：     
*该通证是什么通证，提供的通证数量，单位计量又是什么？对应的账户余额是多少，向某个账户进行转账，批准代理自己花费自己的代币。*

## 取值器
- `function totalSupply() external view returns (uint)`: 通提供的通证数量
- `function balanceOf(address) external view returns (uint)`: 返回地址拥有的通证数量
- `function allowance(address owner, address spender) external view returns (uint256)`: ERC20标准使一个地址能够允许另一个地址从中检索通证。此取值器返回允许`sepnder`代表`owner`花费的剩余通证数量，默认返回 0

## 函数
- `function transfer(address recipient, uint amount) external returns (bool)`：将`amount`数量的通证从`msg.sender`移动到接收者地址，稍后发出`Transfer`事件，如果可以转账，则返回`true`
- `function approve(address spender, uint amount) external returns (bool)`: 设置允许`spender`从函数调用方`msg.sender`余额转账的`allowance`的数额，此函数发出`Approval`事件，返回是否成功设置了余量
- `function transferFrom(address sender, address recipient, uint amount) external returns (bool)`: 使用余量机制将`amount`个通证从`sender`移动到`recipient`，然后从调用者的余量中扣除该数额，此函数发出`Transfer`事件

## 事件
- `event Transfer(address indexed from, address indexed to, uint val)`
- `event Approval(address indexed owner, address indexed spender, uint val)`