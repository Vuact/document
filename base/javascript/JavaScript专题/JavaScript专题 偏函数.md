

维基百科中对偏函数 (Partial application) 的定义为：

在计算机科学中，偏函数是指固定一个函数的一些参数，然后产生另一个更小元的函数。

什么是元？元是指函数参数的个数，比如一个带有两个参数的函数被称为二元函数。

举个简单的例子：

```js
function add(a, b) {
    return a + b;
}

// 执行 add 函数，一次传入两个参数即可
add(1, 2) // 3

// 假设有一个 partial 函数可以做到局部应用
var addOne = partial(add, 1);

addOne(2) // 3
```

个人觉得翻译成“局部应用”或许更贴切些，以下全部使用“局部应用”。

<br>

# 一、柯里化 与 局部应用(偏函数)

如果看过上一篇文章《JavaScript专题 柯里化》，实际上你会发现这个例子和柯里化太像了，所以两者到底是有什么区别呢？

柯里化和偏函数都是用于将多个参数函数，转化为接受更少参数函数的方法。 

- 柯里化：是将函数转化为多个嵌套的一元函数，也就是每个函数只接受一个参数。
- 偏函数：可以接受不只一个参数，它被固定了部分参数作为预设，并可以接受剩余的参数

<br>

# 二、partial

我们今天的目的是模仿 underscore 写一个 partial 函数，比起 curry 函数，这个显然简单了很多。

也许你在想我们可以直接使用 bind 呐，举个例子：

```js
function add(a, b) {
    return a + b;
}

var addOne = add.bind(null, 1);

addOne(2) // 3
```

然而使用 bind 我们还是改变了 this 指向，我们要写一个不改变 this 指向的方法。

<br>

### 1、第一版

根据之前的表述，我们可以尝试着写出第一版：

```js
// 第一版
// 似曾相识的代码
function partial(fn) {
    var args = [].slice.call(arguments, 1);
    return function() {
        var newArgs = args.concat([].slice.call(arguments));
        return fn.apply(this, newArgs);
    };
};
```

我们来写个 demo 验证下 this 的指向：

```js
function add(a, b) {
    return a + b + this.value;
}

// var addOne = add.bind(null, 1);
var addOne = partial(add, 1);

var value = 1;
var obj = {
    value: 2,
    addOne: addOne
}
obj.addOne(2); // ???
// 使用 bind 时，结果为 4
// 使用 partial 时，结果为 5
```

<br>

### 2、第二版

然而正如 curry 函数可以使用占位符一样，我们希望 partial 函数也可以实现这个功能，我们再来写第二版：

```js
// 第二版
var _ = {};

function partial(fn) {
    var args = [].slice.call(arguments, 1);
    return function() {
        var position = 0, len = args.length;
        for(var i = 0; i < len; i++) {
            args[i] = args[i] === _ ? arguments[position++] : args[i]
        }
        while(position < arguments.length) args.push(argumetns[position++]);
        return fn.apply(this, args);
    };
};
```

我们验证一下：

```js
var subtract = function(a, b) { return b - a; };
var subFrom20 = partial(subtract, _, 20);
subFrom20(5); //15
```

<br>
<br>



值得注意的是：underscore 和 lodash 都提供了 partial 函数，但只有 lodash 提供了 curry 函数。

