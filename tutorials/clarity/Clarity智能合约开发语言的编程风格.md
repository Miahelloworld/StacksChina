
该教程的原英文文档：https://book.clarity-lang.org/ch14-01-coding-style.html

**编程风格**

这里有一些关于Clarity编程风格的建议。如果编程风格的SIPs标准被批准，那么SIPs标准将被包含在这里。所以，这一章就没有被命名为由社区主导的编程Clarity指南。在那之前，我们只能根据长期开发者的经验，做出最大的努力。

**Begin函数的编程风格**

大多数函数的输入量都是不变的。因此，有些开发者有过度使用begin函数的倾向。如果你的begin函数只包含一个表达式，那么你可以把它去掉。

```
(define-public (get-listing (id uint))
    (begin
        (ok (map-get? listings {id: id}))
    )
)

```

应该去掉begin函数的代码：

```
(define-public (get-listing (id uint))
    (ok (map-get? listings {id: id}))
)

```

实际上，begin函数是有[运行时成本](https://book.clarity-lang.org/ch13-00-runtime-cost-analysis.html)的，所以不使用它，会使你的智能合约调用更便宜。其他不变函数表达式中的begin函数也是如此。


```
>> ::get_costs (+ 1 2)
+----------------------+----------+------------+
|                      | Consumed | Limit      |
+----------------------+----------+------------+
| Runtime              | 4000     | 5000000000 |
+----------------------+----------+------------+
3

>> ::get_costs (begin (+ 1 2))
+----------------------+----------+------------+
|                      | Consumed | Limit      |
+----------------------+----------+------------+
| Runtime              | 6000     | 5000000000 |
+----------------------+----------+------------+
3

```


**嵌套的let函数**

let函数允许我们定义局部变量。如果你不得不多次读取数据或者重新做一次计算，那么它就很有用。变量表达式实际上是按顺序计算的，这意味着后面的变量表达式可以引用前面的表达式。因此，如果你想做的只是根据一些先前的变量来计算一个值，就没有必要嵌套多个let表达式。

```
(let
    (
    (value-a u10)
    (value-b u20)
    (result (* value-a value-b))
    )
    (ok result)
)

```

避免使用*-panic函数

有多种方法来解包值，但一般应避免使用 unwrap-panic 和 unwrap-err-panic函数。如果它们不能解开提供的值，就会以运行时间错误而中止调用。运行时间错误不会给调用智能合约的应用程序提供任何有意义的信息，并使错误处理更加困难。只要有可能，就使用unwrap！和unwrap-err！这2个函数，来显示有意义的错误代码信息。

比较下面例子中的函数update-name和update-name-panic。


```
(define-public (update-name (id uint) (new-name (string-ascii 50)))
    (let
        (
            ;; Emits an error value when the unwrap fails.
            (listing (unwrap! (get-listing id) err-unknown-listing))
        )
        (asserts! (is-eq tx-sender (get maker listing)) err-not-the-maker)
        (map-set listings {id: id} (merge listing {name: new-name}))
        (ok true)
    )
)

(define-public (update-name-panic (id uint) (new-name (string-ascii 50)))
    (let
        (
            ;; No meaningful error code is emitted if the unwrap fails.
            (listing (unwrap-panic (get-listing id)))
        )
        (asserts! (is-eq tx-sender (get maker listing)) err-not-the-maker)
        (map-set listings {id: id} (merge listing {name: new-name}))
        (ok true)
    )
)

```

最好将 unwrap-panic 和 unwrap-err-panic函数的使用限制在你已经事先知道解包不会失败的情况下(例如，有一个事先的防护措施）。


**避免使用if函数**

说实话，我们实际上不能避免使用if函数。但无论何时你使用它，都要问问自己是否真的需要它。很多时候，你可以重构代码，用asserts！或try！函数来代替if函数。新的开发者往往喜欢会创建嵌套的if结构，因为他们需要按顺序检查多个条件。这些结构变得非常难懂，而且容易出错。

举例说明：


```
(define-public (update-name (new-name (string-ascii 50)))
    (if (is-eq tx-sender contract-owner)
        (ok (var-set contract-name new-name))
        err-not-contract-owner
    )
)

```

上面的代码可以被修改成：

```
(define-public (update-name (new-name (string-ascii 50)))
    (begin
        (asserts! (is-eq tx-sender contract-owner) err-not-contract-owner)
        (ok (var-set contract-name new-name))
    )
)

```

多个嵌套的if表达式通常写成这样样式。

```
(define-public (some-function)
    (if bool-expr-A
        (if bool-expr-B
            (if bool-expr-C
                (ok (process-something))
                if-C-false
            )
            if-B-false
        )
        if-A-false
    )
)

```

也可以跟下面的代码做比较：

```
(define-public (some-function)
    (begin
        (asserts! bool-expr-A if-A-false)
        (asserts! bool-expr-B if-B-false)
        (asserts! bool-expr-C if-C-false)
        (ok (process-something))
    )
)

```

**什么时候使用 match函数和什么时候不应该使用 match函数**：

match是一个非常强大的函数，但在许多情况下，一个try！函数就足够用了。通常观察到的代码样式是这样的。

```
(match (some-expression)
    success (ok success)
    error (err error)
)

```

对其来说，函数上的简化无非是函数调用本身。

```
(some-expression)

```

match函数解开response函数的结果，然后用解开的ok或err值进入成功或失败分支。因此，立即返回这些值是没有意义的。

下面是一个在主网智能合约中发现的transfer函数使用的真实案例：
```
(define-public (transfer (token-id uint) (sender principal) (recipient principal))
    (if (and (is-eq tx-sender sender))
        (match (nft-transfer? my-nft token-id sender recipient)
            success (ok success)
            error (err error))
        (err u500)
    )
)

```

重构if函数和match函数，我们可以把代码写成这样：


```
(define-public (transfer (token-id uint) (sender principal) (recipient principal))
    (begin
        (asserts! (is-eq tx-sender sender) (err u500))
        (nft-transfer? my-nft token-id sender recipient)
    )
)

```

---

