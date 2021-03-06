

程序调用自身的编程技巧称为递归(recursion)。


构成递归需具备边界条件、`递归前进段`和`递归返回段`，当边界条件`不满足`时，`递归前进`，当边界`条件满足`时，`递归返回`。

总结一下递归的特点：

1. 子问题须与原始问题为同样的事，且更为简单；
2. 不能无限制地调用本身，须有个出口，化简为非递归状况处理。

了解这些特点可以帮助我们更好的编写递归函数。

# 一、使用
## 1、阶乘

以阶乘为例：

```js
function factorial(n) {
    if (n == 1) return n;
    return n * factorial(n - 1)
}

console.log(factorial(5)) // 5 * 4 * 3 * 2 * 1 = 120
```

![阶乘](https://github.com/mqyqingfeng/Blog/raw/master/Images/recursion/factorial.gif)

## 2、斐波那契数列

在《JavaScript专题 函数记忆》中讲到过的斐波那契数列也使用了递归，

斐波那契数列递归公式：

- F(0) = 0
- F(1) = 1
- F(n) = F(n - 1) + F(n - 2)（n ≥ 2，n ∈ N*）


```js
function fibonacci(n){
    return n < 2 ? n : fibonacci(n - 1) + fibonacci(n - 2);
}

console.log(fibonacci(5)) // 1 1 2 3 5
```

<br>

# 二、尾调用

## 1、尾调用
在《JavaScript深入 执行上下文栈》中，我们知道：

当执行一个函数的时候，就会创建一个执行上下文，并且压入执行上下文栈，当函数执行完毕的时候，就会将函数的执行上下文从栈中弹出。

试着对阶乘函数分析执行的过程，我们会发现，JavaScript 会不停的创建执行上下文压入执行上下文栈，对于内存而言，维护这么多的执行上下文也是一笔不小的开销呐！那么，我们该如何优化呢？

答案就是尾调用。


`尾调用，是指函数内部的最后一个动作是函数调用。该调用的返回值，直接返回给函数。`

举个例子：

```js
// 尾调用
function f(x){
    return g(x);
}
```

然而

```js
// 非尾调用
function f(x){
    return g(x) + 1;
}
```

并不是尾调用，因为 g(x) 的返回值还需要跟 1 进行计算后，f(x)才会返回值。

两者又有什么区别呢？答案就是执行上下文栈的变化不一样。

为了模拟执行上下文栈的行为，让我们定义执行上下文栈是一个数组：

```js
    ECStack = [];
```

我们模拟下第一个尾调用函数执行时的执行上下文栈变化：

```js
// 伪代码
ECStack.push(<f> functionContext);

ECStack.pop();

ECStack.push(<g> functionContext);

ECStack.pop();
```

我们再来模拟一下第二个非尾调用函数执行时的执行上下文栈变化：

```js
ECStack.push(<f> functionContext);

ECStack.push(<g> functionContext);

ECStack.pop();

ECStack.pop();
```

也就说尾调用函数执行时，虽然也调用了一个函数，但是因为原来的的函数执行完毕，执行上下文会被弹出，执行上下文栈中相当于只多压入了一个执行上下文。然而非尾调用函数，就会创建多个执行上下文压入执行上下文栈。

函数调用自身，称为递归。如果尾调用自身，就称为尾递归。

所以我们只用把阶乘函数改造成一个尾递归形式，就可以避免创建那么多的执行上下文。但是我们该怎么做呢？

<br>

## 2、阶乘函数优化

我们需要做的就是把所有用到的内部变量改写成函数的参数，以阶乘函数为例：

```js
function factorial(n, res) {
    if (n == 1) return res;
    return factorial(n - 1, n * res)
}

console.log(factorial(4, 1)) // 24
```

然而这个很奇怪呐……我们计算 4 的阶乘，结果函数要传入 4 和 1，我就不能只传入一个 4 吗？

这个时候就要用到我们在《JavaScript专题 柯里化》中编写的 curry 函数了：

```js
var newFactorial = curry(factorial, _, 1)

newFactorial(5) // 24
```



