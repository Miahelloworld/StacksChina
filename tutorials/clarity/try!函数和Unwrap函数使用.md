---
title: try!函数和Unwrap函数使用  
date: 2021-10-06T06:30:24.891Z  
lastmod: 2021-10-06T11:56:02.995Z  
author: sulayman  
category:  
notebook:   
favorite: false  
color: white  
shared with:  

---
英文原文档：https://book.clarity-lang.org/ch06-02-try.html

 try!函数采用optional 或 response 类型，并将尝试解开它。解开是提取内部值并返回它的动作。以下面的例子为例：
 
 
```
(try! (some "wrapped string"))

```

它将解开some 并返回内部的“wrapped string”。



try! 只能成功解开 some 和 ok 的值。如果它收到 none 或 err，它将返回输入值并退出当前控制流。换句话说：

* 如果它收到一个 none，它返回一个 none 并退出。

* 如果它收到一个err，它返回那个err,并退出。它不会打开里面的值！

下面的测试函数允许我们试验这种行为。它将响应类型作为输入传递给 try!。然后我们将使用 ok 和 err 调用该函数并打印结果。

```
(define-public (try-example (input (response uint uint)))
    (begin
        (try! input)
        (ok "end of the function")
    )
)

(print (try-example (ok u1)))
(print (try-example (err u2)))

```

第一个打印结果为（ok "end of the function"），如begin 表达式的末尾所示。但是通过 err 的第二个调用将返回原始值 (err u2)。尝试！因此，函数允许您传播发生在子调用中的错误，我们将在有关[中间人响应](https://book.clarity-lang.org/ch06-04-response-checking.html)的章节中看到更多的用法。


**Unwrap 函数**

其他unwrap函数都是以稍微不同的方式退出当前控制流的变体。

unwrap! 将 optional 或response作为第一个输入，将抛出值作为第二个输入。它遵循与 try! 相同的解包行为！ 但不是传播 none 或 err 而是返回 throw 值。

```
(unwrap! (some "wrapped string") (err "unwrap failed"))

```


unwrap-panic 接受一个输入，它可以是一个optional或response。如果它无法解包输入，它会抛出一个运行时间错误并退出当前流。

```
(unwrap-panic (ok true))

```

unwrap-err-panic 是 unwrap-panic 的相对应者，如果输入是err，则解包，否则抛出运行时间错误。

```
(unwrap-err-panic (err false))

```


理想情况下，除非绝对必须，否则不应使用 -panic 变体，因为它们在失败时不会提供任何有意义的信息。一笔转账交易将返回一个模糊的“运行时间错误”，用户和开发人员就需要弄清楚到底出了什么问题。


**Unpacking assignments**

在使用 let 分配局部变量时，unwrap函数特别有用。如果存在，您可以解包并分配一个值，如果不存在则退出。它使用maps和lists函数来执行。


```
;; Some error constants
(define-constant err-unknown-listing (err u100))
(define-constant err-not-the-maker (err u101))

;; Define an example map called listings, identified by a uint.
(define-map listings
    {id: uint}
    {name: (string-ascii 50), maker: principal}
)

;; Insert some sample data
(map-set listings {id: u1} {name: "First Listing", maker: tx-sender})
(map-set listings {id: u2} {name: "Second Listing", maker: tx-sender})

;; Simple function to get a listing
(define-read-only (get-listing (id uint))
    (map-get? listings {id: id})
)

;; Update name function that only the maker for a specific listing
;; can call.
(define-public (update-name (id uint) (new-name (string-ascii 50)))
    (let
        (
            ;; The magic happens here.
            (listing (unwrap! (get-listing id) err-unknown-listing))
        )
        (asserts! (is-eq tx-sender (get maker listing)) err-not-the-maker)
        (map-set listings {id: id} (merge listing {name: new-name}))
        (ok true)
    )
)

;; Two test calls
(print (update-name u1 "New name!"))
(print (update-name u9999 "Nonexistent listing..."))

```

找到注释的地方 ;; The magic happens here。在 update-name 函数中并仔细研究下一行。这是发生的事情：

* 它定义了一个名为listing的变量。

* 该值将等于 get-listing 函数的解包结果。

* get-listing 返回 map-get? 的结果，它是some列表或者none。

* 如果解包失败，unwap!以 err-unknown-listing 退出。

因此，第一个测试调用将成功并返回（ok true），而第二个调用将出错，并返回（err u100）（err-unknown-listing）。
---

