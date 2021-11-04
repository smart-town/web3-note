# 源文件结构

1. SPDX License Identifier
2. Pragmas
3. 版本标识
4. 导入其他源文件
5. 注释

## 1. SPDX License Identifier

`// SPDX-License-Identifier: MIT`

## 2. Pragmas

关键字`pragma`版本标识指令，用来启用某些编译器检查。版本标识通常只对本文件起作用。所以需要将`pragma`添加到项目中的所有源文件中。如果使用了`import`导入其他的文件，标识并不会从被导入的文件加入到导入的文件中。

## 3. 版本标识

版本号的形式通常是`0.x.0`或者`x.0.0`。使用：`pragma solidity ^0.5.2`。

这样，源文件将既不允许低于 0.5.2 版本的编译器编译，也不允许高于（包含）0.6.0 版本的编译器编译。表达式遵循 npm 版本语义。

## 4. 导入其他源文件

Solidity 支持导入语句来模块化代码，语法与 ES6 类似。不过 Solidity 不支持`default export`。

全局层面上，可以：`import "filename";`。这种形式不建议使用，因为会无法预测地污染当前命名空间。更好地方式是导入明确的符号：`import * as sysmbolName from "filename";`。

### 路径

上文中的`fileName`总是会按照路径来处理。以`/`作为目录分隔符，`.`标识当前目录。用`import "./filename"`导入当前源文件同目录下的`filename`。如果用`import "filename" as symbolName`代替，可能会引入不同的文件。

最终导入哪个文件取决于编译器到底是怎样解析路径的。通常，目录层次不需要严格映射本地文件系统，也可以映射到能通过`ipfs`、`http`或`git`发现的资源。

### 在实际编译器中使用

运行编译器时，不仅能够指定如何发现路径的第一元素，还可以路径前缀重映射。例如`github.com/ethereum/dapp-bin/library`会被重映射到`/usr/local/dapp-bin/library`，此时编译器将从映射位置读取文件。如果重映射到多个路径，编译器将会优先尝试重映射路径最长的一个。

**solc**，对于`solc`，这些重映射以`context:prefix=target`形式的参数提供，其中`context:`和`=target`是可选的（此时`target`默认为`prefix`）。所有重映射的值都是被编译过的常规文件（包括它们的依赖）。

## 5. 注释

可以使用单行注释和多行注释
```sol
// 单行注释
/*
    多行注释
*/
```

此外，还有一种注释称为`natspec`注释，它们是使用三个反斜杠`///`或双星号开头的块`/** ... */`书写。它们应该直接写在函数声明或者语句上使用。可以在注释中使用`Doxygen`样式的标签来文档化函数。