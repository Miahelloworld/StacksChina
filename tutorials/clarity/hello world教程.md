
学习 Clarity语言的基础知识，并编写一个简单的 Hello World 智能合约程序。

英语开发文档原地址：https://docs.stacks.co/write-smart-contracts/hello-world-tutorial

**介绍**

在智能合约的世界里，一切都是区块链交易。您使用钱包中的代币在交易中部署智能合约，并且在合约发布后，对该合约的每次调用也是一次交易。由于区块时间会影响函数执行和返回的速度，因此使用模拟区块链对智能合约进行本地开发和测试是有利的，以便函数立即执行。本教程介绍如何使用Clarinet进行本地智能合约开发，clarinet是一种用于构建和测试Clarity 智能合约的开发工具。

Clarity 是 Stacks 区块链上使用的智能合约语言，是一种基于LISP的开发语言，并使用其括号表示法。 Clarity 是一种解释型语言，并且是可判定的。要了解有关该语言的更多基础知识，请参阅 Clarity 简介。

跟多了解clarity简介：https://docs.stacks.co/write-smart-contracts/overview
什么是clarinet：https://docs.stacks.co/write-smart-contracts/clarinet

**在本教程中，您将学习到：**

* 创建一个新的Clarinet项目

* 向项目添加新的 Clarity 智能合同程序

* 用两种类型的函数填入合约程序

* 在本地模拟区块链环境中执行功能

* （可选）在stacks区块链测试网上部署和测试合约

**前期准备：**

对于本教程，您应该在本地安装了Clarinet。有关如何设置本地环境的说明，请参阅安装[安装Clarinet教程](https://docs.stacks.co/write-smart-contracts/clarinet#installing-clarinet)。您还应该有一个文本编辑器或IDE来编辑Clarity智能合约。

请注意，您还可以通过在线REPL完成本教程的编码部分，例如 clear.tools。如果您使用的是在线REPL，您可以跳到教程的第3步并将代码输入沙盒。如果您使用的是 Visual Studio Code，您可能需要安装 Clarity Visual Studio Code 插件。

**其他备选的准备工作：**

虽然本教程主要关注本地智能合约开发，但您可能希望将合约部署到实时区块链。简单起见，合约部署是使用测试网络沙盒执行的。如果您希望完成可选的部署步骤，您应该安装Stacks浏览器钱包，并且您应该从浏览器测试模式上的水龙头请求一些测试网络的STX代币。请注意，从水龙头请求 testnet STX加密货币最多可能需要15分钟，因此，建议您在开始教程之前，提前请求STX加密货币。
浏览器测试模式网址：https://explorer.stacks.co/sandbox/deploy?chain=testnet
stacks浏览器钱包下载地址：https://www.hiro.so/wallet/install-web
STX加密货币测试申请水龙头：https://explorer.stacks.co/sandbox/faucet?chain=testnet

**第一步：新建项目**

在本地安装Clarinet后，打开一个新的终端窗口,并使用以下命令创建一个新的Clarinet项目：

```
clarinet new clarity-hello-world && cd clarity-hello-world
```

此命令为您的智能合约项目创建一个新目录，其中包含了样板配置和测试文件。创建新项目只会创建Clarinet配置文件，在下一步中您可以向项目中添加智能合约程序。

**第二步：创建新的智能合约程序**

在 clear-hello-world 目录中，使用以下命令创建一个新的Clarity智能合约程序：

```
clarinet contract new hello-world
```


该命令在contracts目录中添加了一个新的hello-world.clar 文件，并在test目录中添加了一个hello-world_test.ts文件。本教程会忽略测试文件，但对于生产环境，您可以使用它创建单元测试。


**第三步：在 hello-world 合约中添加代码**

在文本编辑器或者IDE中打开 contract/hello-world.clar 文件。删除样板注释，就本教程而言，它们不是必需的。
对于本教程，您将向合约程序里写入两个Clarity函数。 Clarity函数完全用括号括起来，空格无关紧要。第一个函数是一个名为 say-hi 的公共函数。

```
(define-public (say-hi)
  (ok "hello world"))
```

Clarity中的公共函数可从其他智能合约调用，这使您能够将复杂的任务分解为更小、更简单的智能合约（[通过wiki百科了解什么是分离关注点](https://en.wikipedia.org/wiki/Separation_of_concerns)）



> 要创建私有函数，您可以使用define-private 关键字。私有函数只能在声明它们的智能合约中调用。外部合约只能调用公共函数。


该函数不接受任何参数，仅使用ok响应构造函数返回“hello world”。第二个函数是一个名为echo-number的只读函数。

```
(define-read-only (echo-number (val int))
  (ok val))
```


只读函数也是公共函数，顾名思义，它们不能更改任何变量或数据映射。 echo-number接受一个int类型的输入参数，并使用ok响应返回传递给函数的值。

Clarity语言同时支持多种其他类型, 可以通过下面的链接查看更多参考类型：https://docs.stacks.co/references/language-types

完整的 contract/hello-world.clar 文件应如下所示：

```
(define-public (say-hi)
  (ok "hello world"))

(define-read-only (echo-number (val int))
  (ok val))
```

您可以通过以下步骤在本地控制台中与此合约进行交互。您还可以选择将此合约部署到测试网络并在实时区块链上与其交互。

**第四步：在Clarinet控制台中与智能合约进行交互**

在终端中的 clear-hello-world 目录中，使用以下命令验证合约程序中的语法是否正确：

```
clarinet check
```

如果没有错误，则该命令不返回任何输出。如果有错误，请验证您的合约程序是否与上一节中列出的代码完全相同。 在同一目录中，使用以下命令启动本地控制台：

```
clarinet console
```

此控制台是一个Clarinet读取-评估-打印循环 (REPL)，可在调用函数时立即执行Clarity代码。当调用Clarinet控制台时，它会输出内存中可用合约程序和模拟钱包的摘要信息：


```
clarity-repl v0.11.1
Enter "::help" for usage hints.
Connected to a transient in-memory database.
Contracts
+-------------------------------------------------------+-------------------------+
| Contract identifier                                   | Public functions        |
+-------------------------------------------------------+-------------------------+
| ST1HTBVD3JG9C05J7HBJTHGR0GGW7KXW28M5JS8QE.hello-world | (echo-number (val int)) |
|                                                       | (say-hi)                |
+-------------------------------------------------------+-------------------------+

Initialized balances
+------------------------------------------------------+---------+
| Address                                              | STX     |
+------------------------------------------------------+---------+
| ST1HTBVD3JG9C05J7HBJTHGR0GGW7KXW28M5JS8QE (deployer) | 1000000 |
+------------------------------------------------------+---------+
| ST1J4G6RR643BCG8G8SR6M2D9Z9KXT2NJDRK3FBTK (wallet_1) | 1000000 |
+------------------------------------------------------+---------+
| ST20ATRN26N9P05V2F1RHFRV24X8C8M3W54E427B2 (wallet_2) | 1000000 |
+------------------------------------------------------+---------+
| ST21HMSJATHZ888PD0S0SSTWP4J61TCRJYEVQ0STB (wallet_3) | 1000000 |
+------------------------------------------------------+---------+
| ST2QXSK64YQX3CQPC530K79XWQ98XFAM9W3XKEH3N (wallet_4) | 1000000 |
+------------------------------------------------------+---------+
| ST3DG3R65C9TTEEW5BC5XTSY0M1JM7NBE7GVWKTVJ (wallet_5) | 1000000 |
+------------------------------------------------------+---------+
| ST3R3B1WVY7RK5D3SV5YTH01XSX1S4NN5B3QK2X0W (wallet_6) | 1000000 |
+------------------------------------------------------+---------+
| ST3ZG8F9X4VKVTVQB2APF4NEYEE1HQHC2EDBF09JN (wallet_7) | 1000000 |
+------------------------------------------------------+---------+
| STEB8ZW46YZJ40E3P7A287RBJFWPHYNQ2AB5ECT8 (wallet_8)  | 1000000 |
+------------------------------------------------------+---------+
| STFCVYY1RJDNJHST7RRTPACYHVJQDJ7R1DWTQHQA (wallet_9)  | 1000000 |
+------------------------------------------------------+---------+
```



控制台提供了使用Clarity命令与您的合约程序进行互动的能力。你可以使用以下命令调用 say-hi 函数：

```
(contract-call? .hello-world say-hi)
```


控制台立即返回（ok "hello world"），这个函数的预期返回值。接下来，我们要调用echo-number函数：

```
(contract-call? .hello-world echo-number 42)
```

控制台立即返回 (ok 42)，该函数的预期返回值以及您调用它的参数。你可以尝试使用不正确的类型调用 echo-number 函数，在本例中为无符号整数：

```
(contract-call? .hello-world echo-number u42)
```

控制台应该返回 Analysis error: expecting expression of type 'int', found 'uint', 会检查出由于类型错误，对合约程序参数的调用无效。

您现在已经学习了Clarity语言的基础知识并使用了Clarinet开发工具。您可能希望有选择地将智能合约部署到测试网络，接下来的教程中，我们会继续学习。


**在测试网络上部署和测试合约程序**


在本教程中，您将使用测试网络沙盒来部署您的智能合约。确保您已使用连接钱包按钮，将Stacks网络钱包连接到沙盒，然后将您的智能合约复制并粘贴到写入和部署页面上的Clarity代码编辑器中。编辑合约程序的名称或使用提供给您的随机生成的名称。

![image](https://docs.stacks.co/images/hello-world-testnet-sandbox.png)

单击部署按钮将合约程序部署到区块链。这时将显示包含交易信息的Stacks网络钱包窗口。验证交易看起来正确，并且网络切换设置为Testnet，然后单击确认。


合约程序被添加到矿工内存池中，并包含在区块链的下一个区块中。此过程最多可能需要15分钟才能完成。您可以在浏览器的交易页面或您的网络钱包的活动页面中查看。

再确认您的合约程序执行后，你可以跳转到沙盒的调用合约程序页面，并搜索您的合约。在顶部输入框中输入您的钱包地址，您可以通过单击Stacks网络钱包图标并单击复制地址按钮来复制此地址。在底部输入框中输入智能合约名称，在本例中为 hello-world。单击获取智能合约以查看智能合约更多部署信息。

![image](https://docs.stacks.co/images/hello-world-sandbox-contract.png)


单击函数总览信息中的say-hi函数，然后单击 Call Function 执行沙盒中的函数调用。这将显示包含交易信息的Stacks网络钱包。然后核对信息，再点击确认，执行函数调用。

函数调用被添加到矿工内存池中，并在区块链的下一个区块中执行。此过程最多可能需要15分钟才能完成。您可以在浏览器的交易页面或stacks网络钱包的活动页面中查看。

交易完成后，您可以从网络钱包的活动页面访问交易总览信息页面。交易总览信息页面显示函数的输出如下：


![image](https://docs.stacks.co/images/hello-world-transaction-summary.png)

您现在已经学习了在Stacks测试网络上部署智能合约并与之交互的方法。您还了解了无需等待区块时间，就可以执行本地开发的优势。



















---

