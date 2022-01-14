

英文原地址： https://github.com/stacksgov/sips/blob/main/sips/sip-009/sip-009-nft-standard.md


**序言**


* * *

SIP号码：009

标题：不可替代代币的标准特征定义

作者：Friedger Müffke (mail@friedger.de), Muneeb Majeed

归类：技术

类型：标准

状态：已批准

创建时间：2020 年 12 月 10 日

许可证：CC0-1.0

签字人：Jude Nelson jude@stacks.org，技术指导委员会主席

**简介**


* * *

不可替代的代币或NFT是在区块链上注册的数字资产，具有区分它们的唯一标识符和属性。它是一种可以被唯一地识别、拥有和转移的不可替代的代币。该SIP009协议旨在提供一种灵活且易于实施的标准，开发人员在创建自己的 NFT 时可以在Stacks区块链上使用该标准。本标准仅规定了一种基本要求，不可替代的代币可以具有比本标准规定的更多的功能。


**许可和版权**


* * *


此SIP标准适用于知识共享CC0 1.0 通用许可条款，https://creativecommons.org/publicdomain/zero/1.0/ 该SIP的版权归Stacks开放互联网基金会所有。

**介绍**

* * *

代币是通过智能合约程序在区块链上注册的数字资产。不可替代的代币（NFT）是一种全球唯一的代币，可以通过其唯一标识符进行识别。

在具有智能合约程序的区块链中，包括Stacks区块链，开发人员和用户可以使用智能合约程序来注册不可替代的代币并与之交互。

Stacks区块链用于开发智能合约的编程语言是Clarity， Clarity具有用于定义和使用不可替代代币的内置语言。尽管存在这些内置语言，但定义一个通用接口（在 Clarity 中称为“特征”）是有价值的，该接口允许不同的智能合约程序以可以重复使用的方式与不可替代的代币合约进行互操作。此SIP标准定义了该特征。

每个NFT始终属于一个智能合约。智能合约的NFT从1开始枚举。当前最后一个ID由智能合约函数提供。资产ID与合约ID一起定义了一个全球唯一的 NFT。


**规范**

* * *

Stacks区块链中的每个符合SIP-009标准的智能合约程序都必须实现在Trait部分定义的 [trait函数](https://github.com/stacksgov/sips/blob/main/sips/sip-009/sip-009-nft-standard.md#trait)，nft-trait，并且必须满足以下功能的要求：


**最后的代币ID**


```
(get-last-token-id () (response uint uint))
```
不接受任何参数并返回使用合约程序注册的最后一个NFT的标识符。在迭代所有NFT时，返回的ID可以用作上限。

此函数绝不能返回错误响应。它可以定义为只读，例如：define-read-only

**代币URI**

```
(get-token-uri (uint) (response (optional (string-ascii 256)) uint))
```

获取一个NFT标识符并返回包含解析为NFT元数据的有效URI的响应。 URI字符串必须包含在optional中。如果相应的NFT不存在或合约不维护元数据，则响应必须是（ok none）。如果NFT存在有效的 URI，则响应必须是 (ok (some "<URI>"))。返回的 URI 的长度限制为256个字符。元数据的规范应包含在单独的SIP协议中。

此函数绝不能返回错误响应。它可以定义为只读，即define-read-only。


**拥有人**

```
(get-owner (uint) (response (optional principal) uint))
```
获取一个NFT标识符并返回一个响应，其中包含拥有给定标识符的 NFT 的主体。主体用户必须包装在optional中。如果相应的NFT不存在，则响应必须是 (ok none)。拥有人可以是合约程序的主体用户。

如果对函数 get-owner 的调用返回了某个主体用户A，那么它必须返回相同的值，直到以主体用户 A 作为发送方调用transfer函数为止。

对于ID大于 get-last-token-id 函数返回的最后一个代币ID的任何对 get-owner 的调用，该调用必须返回响应（ok none）。

此函数绝不能返回错误响应。它可以定义为只读，即define-read-only。



**Transfer函数**

```
(transfer (uint principal principal) (response bool uint))
```


该函数将给定标识符的NFT所有权从发送方主体用户更改为接收方主体用户。

这个函数必须用define-public 定义，因为它改变了状态，并且必须是外部可调用的。

成功调用transfer函数后，函数 get-owner 必须返回transfer调用的接收者作为新的所有者。

对于ID大于 get-last-token-id 函数返回的最后一个代币ID 的任何transfer调用，该调用必须返回错误响应。

建议使用标准化代码列表中的错误代码，并实现将错误代码转换为消息的功能，这些功能在单独的SIP协议中定义。


**Trait**


```
(define-trait nft-trait
  (
    ;; Last token ID, limited to uint range
    (get-last-token-id () (response uint uint))

    ;; URI for metadata associated with the token 
    (get-token-uri (uint) (response (optional (string-ascii 256)) uint))

     ;; Owner of a given token identifier
    (get-owner (uint) (response (optional principal) uint))

    ;; Transfer from the sender to a new principal
    (transfer (uint principal principal) (response bool uint))
  )
)

```

**使用native asset 函数**

尽管不可能在 Clarity trait 中进行授权，但合约程序的实施者必须定义至少一个作为 Clarity 初始化语言提供的内置原生不可替代的[asset类别](https://app.sigle.io/friedger.id/FDwT_3yuMrHDQm-Ai1OVS)。这允许客户使用Post 条件（如下所述），并利用其他好处，例如对这些资产余额的本地支持和通过 stacks-blockchain-api 进行转账。此 SIP协议中包含的参考实现使用本机资产初始化语言，并为其使用提供了良好的样板。

初始化asset函数包括：

* define-non-fungible-token

* nft-burn?

* nft-get-owner?

* nft-mint?

* nft-transfer?


定义了以下使用初始化asset函数的要求：


**Transfer函数**


如果在拒绝模式下没有post条件，或没有任何关于更改所有者的NFT条件，从客户端调用transfer函数，则函数调用必须失败并显示 abort_by_post_condition。


**在应用程序中使用NFTs**

* * *
希望在应用程序中使用不可替代代币合约的开发人员, 应首先提供或跟踪各种不同的不可替代代币实现。在验证不可替代的代币合约时，他们应该获取该合约的接口和/或源代码。如果合约程序实现了trait，那么应用程序可以使用这个标准的合约接口来进行转账和获取这个标准中定义的其他细节。

此 trait 中的所有函数都返回response类型，这是 Clarity 中 trait 定义的要求。但是，其中一些函数应该是“fail-proof”，也就是说它们永远不应该返回错误。这些“fail-proof”函数是推荐为只读的函数。如果实现此特征的合约程序为这些函数返回错误，则可能表明此合约程序不合规，这些合约程序的用户应该小心地使用它们。


**Post条件的使用**

* * *

Stacks区块链包括一个称为“Post-Conditions”或“Constraints”的功能。通过定义post-conditions，用户可以创建交易，其中包含有关该合约中可能发生的事情的预定义保证。

例如，当应用程序调用transfer函数时，它们应该始终使用post条件来指定NFT的新所有者就是transfer函数调用中的接收方主体用户。

**相关资料**

* * *
NFTs是区块链上已建立的资产类别。[在这里](https://www.ledger.com/academy/what-are-nft)阅读例子。



**EIP 721协议**

* * *

以太坊拥有EIP721协议，它在以太坊区块链上定义了不可替代的代币。明显的区别是 EIP721中的 transfer函数使用以代币id结尾的参数的不同顺序。此 SIP 中的 transfer函数使用代币ID 作为第一个参数，这与 Clarity 中的其他固有的函数一致。此外，该 SIP 仅定义了一个函数，用于获取指向NFT元数据的 URI。代币元数据的模式和其他属性的规范应在单独的 SIP 中定义。

**激活状态**

* * *


如果部署了5个使用遵循此规范的相同特征的合约程序，则此 SIP 被激活。这必须发生在比特币 tip #700,000 之前。

遵循此规范的trait目前在主网上已经可以使用。SP2PABAF9FTAJYNFZH93XENAJ8FVY99RRM50D2JG9.nft-trait.nft-trait


**源代码案例**

* * *

**参考实现**

* * *


Friedger 的clarity智能合约程序案例
https://github.com/friedger/clarity-smart-contracts/blob/master/contracts/sips/nft-trait.clar



---

