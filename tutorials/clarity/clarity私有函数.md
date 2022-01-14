
原英文文档链接：https://book.clarity-lang.org/ch05-02-private-functions.html

**只读函数**

只读函数可以被合约程序本身调用，也可以从外部调用。它们可以返回任何类型，就像私有函数一样。

顾名思义，只读函数只能执行读操作。您可以读取数据变量和映射，但不能写入它们。只读函数也可以是纯函数式的；也就是说，根据输入计算一些结果并返回它。代码可以这样写：

```
(define-read-only (add (a uint) (b uint))
    (+ a b)
)

(print (add u5 u10))

```

代码还可以这样写：

```
(define-data-var counter uint u0)

(define-read-only (get-counter-value)
    (var-get counter)
)

(print (get-counter-value))

```


然而，下面是一个错误的代码示例：

```
(define-data-var counter uint u0)

(define-read-only (increment-counter)
    (var-set (+ (var-get counter) u1))
)

(print (increment-counter))

```

如您所见，分析告诉我们它在只读函数中检测到写入操作：

> 分析错误：只读语句，检测到写操作（define-data-var counter uint u0）

不过不用担心，没有严重的后果。如果分析失败，则合约程序无效，这意味着它无法部署在网络上。


使只读函数非常有趣的一件事是，它们可以在不实际发送转账交易的情况下被调用！通过使用只读函数，您可以读取应用程序的合约状态，而无需您的用户支付交易费用。 [Stacks.js](https://github.com/blockstack/stacks.js) 和 [官方钱包浏览器扩展](https://www.hiro.so/wallet/install-web) 支持调用内置的只读函数。您现在可以使用 [Stacks Sandbox](https://explorer.stacks.co/sandbox/contract-call) 亲自尝试一下。找到一个带有只读函数的合约程序，直接调用。完全免费！


练习题：创建一个只读函数，返回给指定主体用户的计数器值，如果主体用户不存在于映射中，则返回 u0。

```
(define-map counters principal uint)

(map-set counters 'ST1J4G6RR643BCG8G8SR6M2D9Z9KXT2NJDRK3FBTK u5)
(map-set counters 'ST20ATRN26N9P05V2F1RHFRV24X8C8M3W54E427B2 u10)

(define-read-only (get-counter-of (who principal))
    ;; Implement.
)

;; These exist:
(print (get-counter-of 'ST1J4G6RR643BCG8G8SR6M2D9Z9KXT2NJDRK3FBTK))
(print (get-counter-of 'ST20ATRN26N9P05V2F1RHFRV24X8C8M3W54E427B2))

;; This one does not:
(print (get-counter-of 'ST21HMSJATHZ888PD0S0SSTWP4J61TCRJYEVQ0STB))

```



**私有函数**

私有函数的定义方式与公共函数相同。不同的是它们只能被当前的合约程序调用。它们不能被其他智能合约程序调用，也不能通过发送转账交易直接调用。私有函数可用于创建实用程序或辅助函数以减少代码重复。如果您发现自己在多个位置有重复类似的表达式，那么值得考虑将这些表达式转换为单独的私有函数。

下面的合约程序只允许合约所有者通过两个公共函数更新recipients 地图。不必重复地检查 tx-sender ，而是将其抽象为自己的私有函数，称为 is-valid-caller。

```
(define-constant contract-owner tx-sender)

;; Try removing the contract-owner constant above and using a different
;; one to see the example calls error out:
;; (define-constant contract-owner 'ST20ATRN26N9P05V2F1RHFRV24X8C8M3W54E427B2)

(define-constant err-invalid-caller (err u1))

(define-map recipients principal uint)

(define-private (is-valid-caller)
    (is-eq contract-owner tx-sender)
)

(define-public (add-recipient (recipient principal) (amount uint))
    (if (is-valid-caller)
        (ok (map-set recipients recipient amount))
        err-invalid-caller
    )
)

(define-public (delete-recipient (recipient principal))
    (if (is-valid-caller)
        (ok (map-delete recipients recipient))
        err-invalid-caller
    )
)

;; Two example calls to the public functions:
(print (add-recipient 'ST1J4G6RR643BCG8G8SR6M2D9Z9KXT2NJDRK3FBTK u500))
(print (delete-recipient 'ST1J4G6RR643BCG8G8SR6M2D9Z9KXT2NJDRK3FBTK))

```
另一个很好的好处是：定义私有函数可以降低整体函数的复杂性。大型公共函数可能更难维护，更容易出现开发人员错误。将这些功能拆分为公共函数和一些较小的私有函数可以缓解这些问题。

私有函数可以返回任何类型，包括响应，尽管会返回 ok 或 err， 这并不会影响链的具体化状态。


练习题：写一个名为“is-valid-caller”的私有函数，该函数根据 tx-sender 是否是授权主体用户之一，返回 true 或 false。



```
(define-constant err-invalid-caller (err u1))

(define-map authorised-callers principal bool)
(define-map recipients principal bool)

(map-set recipients tx-sender true)
(map-set authorised-callers 'ST20ATRN26N9P05V2F1RHFRV24X8C8M3W54E427B2 true)

(define-private (is-valid-caller (caller principal))
    ;; Implement.
)

(define-public (delete-recipient (recipient principal))
    (if (is-valid-caller tx-sender)
        (ok (map-delete recipients recipient))
        err-invalid-caller
    )
)

(print (delete-recipient tx-sender))

```




---

