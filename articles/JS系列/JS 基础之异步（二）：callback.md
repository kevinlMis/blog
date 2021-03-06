## step1：callback

```js
asyncFunction(function(value) {
    // todo
})
```

这种回调函数，大家是最熟悉的。一般是需要在某个耗时操作之后执行某个回调函数。

例如：

```js
setTimeout(function() {
    console.log('Time out')
}, 1000)
```

其中，我们称`setTimeout`为发起函数，`fn`为回调函数。都是在主线程上调用的，其中发起函数用来发动异步过程，回调函数用来处理结果。在执行`setTimeout`1s后，执行function函数。

下面，我们再看一种情况。

```js
$.ajax({
    url:'XXX1',
    success: function(res) {
        $.ajax({
            url:'XXX2',
            success: function(res) {
                $.ajax({
                    url: 'XXX3',
                    success: function(res) {
                        // todo
                    },
                    fail: function(err) {
                        console.log(err)
                    }
                })
            },
            fail: function(err) {
                console.log(err)
            }
        }) 
    },
    fail: function(err) {
    	console.log(err)
	}
})
```

在上例中，我们看到这段回调函数，不断的在回调，这只是三层回调，在实际应用中，我们遇到的需求会更复杂，回调也许更多，调试起来也就更麻烦，代码也更不美观，这就是我们要引入的第一个问题：回调地狱。

**问题1: 回调地狱**

回调地狱是JS里一个约定俗成的名称，一般情况下，一个业务依赖于上层业务，上层业务又依赖于更上一层的业务，以此类推，如果我们使用回调函数来处理异步的话，就会出现回调地狱。

主要是因为：大脑对业务的逻辑处理是线性的、阻塞的、单线程的，但是回调表达异步的方式是非线形的、非顺序的，这使得正确推导这类代码的难度很大，很容易出bug。

再例如：

```js
// A
$.ajax({
    ...
    success: function (...) {
        // C
    }
});
// B
```

A和B发生于现在，在JavaScript主程序的直接控制之下，而C会延迟到将来发生，并且是在第三方的控制下，在本例中就是函数$.ajax(...)。从根本上来说，这种控制的转移通常不会给程序带来很多问题。

但是，请不要被这个小概率迷惑而认为这种控制切换不是什么大问题。实际上，这是回调驱动设计最严重（也是最微妙）的问题。它以这样一个思路为中心：有时候ajax(...)，也就是你交付回调函数的第三方不是你编写的代码，也不在你的直接控制之下，它是某个第三方提供的工具。

这种情况称为**控制反转**，也就是把自己程序一部分的执行控制交给某个第三方，在你的代码和第三方工具直接有一份并没有明确表达的契约。

既然是无法控制的第三方在执行你的回调函数，那么就有可能存在以下问题，当然通常情况下是不会发生的：

1. 调用回调过早
2. 调用回调过晚
3. 调用回调次数太多或者太少
4. 未能把所需的参数成功传给你的回调函数
5. 吞掉可能出现的错误或异常
6. ......

这种控制反转会导致信任链的完全断裂，如果你没有采取行动来解决这些控制反转导致的信任问题，那么你的代码已经有了隐藏的Bug，尽管我们大多数人都没有这样做。

这里，我们引出了回调函数处理异步的第二个问题：控制反转。

**问题2：控制反转**

综上，回调函数处理异步流程存在2个问题：

**1. 缺乏顺序性： 回调地狱导致的调试困难，和大脑的思维方式不符**

**2. 缺乏可信任性： 控制反转导致的一系列信任问题**

那么如何来解决这两个问题，先驱者们开始了探索之路......