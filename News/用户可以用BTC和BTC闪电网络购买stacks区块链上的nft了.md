
用户可以用BTC和BTC闪电网络购买stacks区块链上的nft了

LNSwap.org推出了一个可以嵌入的代码，开发者可以整合api到自己的nft交易网站。让普通用户可以在独立网站上直接通过btc兑换stx，或者在nft交易网站上直接支付btc，就可以购买并收到nft。

**体验网址：**
https://www.lnswap.org/


**开发者说明文档：**

https://www.lnswap.org/developers

**实际使用案例：**

stxnft交易市场上的roots系列的nft采用了这种交易方式。

![image](https://miro.medium.com/max/2400/1*yfOPLhwZ69rDWsIZgA1XKw.png)


**如何整合？**

开发者可以参考以下代码：

```
<div id=”root”></div>
<script src=”https://lnswap-widget.vercel.app/widget.js"
id=”LNSwap-Widget-Script”
data-config=”{‘name’: ‘lnswap’, ‘config’: {‘targetElementId’: ‘root’}}”>
</script>
// Start the Swap
lnswap(‘swap’,
‘swapType’,
‘user stx address’,
‘amount in STX’,
‘(only for mintnft) NFT Contract Address’,
‘(only for mintnft) NFT Mint Function Name’);
// e.g. Mint NFT with Lightning
lnswap(‘swap’, ‘mintnft’, ‘ST27SD3H5TTZXPBFXHN1ZNMFJ3HNE2070QX7ZN4FF’, 25, ‘ST27SD3H5TTZXPBFXHN1ZNMFJ3HNE2070QX7ZN4FF.stacks-roots-v2’, ‘claim-for’);
// e.g. Trustless Swap Lightning to STX
lnswap(‘swap’, ‘reversesubmarine’, ‘ST27SD3H5TTZXPBFXHN1ZNMFJ3HNE2070QX7ZN4FF’, 5);
```


**整合api后效果图：**


![image](https://miro.medium.com/max/2400/1*11vLrOHVsDYatGYbnMpw5g.png)


---

