在开发中，我们经常会遇到在数组中查找指定元素的需求，可能大家觉得这个需求过于简单，然而如何优雅的去实现一个 findIndex 和 findLastIndex、indexOf 和 lastIndexOf 方法却是很少人去思考的。本文就带着大家一起参考着 underscore 去实现这些方法。

在实现前，先看看 ES6 的 findIndex 方法，让大家了解 findIndex 的使用方法。

# 1、ES6: findIndex

ES6 对数组新增了 findIndex 方法，它会返回数组中满足提供的函数的第一个元素的索引，否则返回 -1。

举个例子：

```js
function isBigEnough (value, index, arr) {
  console.log(value, index, arr);
  return value >= 15;
}

const resIndex = [12, 5, 18, 44].findIndex(isBigEnough);  
console.log(resIndex);

//输出：
// 12 0 [ 12, 5, 18, 44 ]
// 5  1 [ 12, 5, 18, 44 ]
// 18 2 [ 12, 5, 18, 44 ]
// 2
```

findIndex 会找出第一个大于 15 的元素的下标，所以最后返回 2。

findIndex还接受第二个参数，用来绑定回调函数的this对象。

```js
const person = {name: 'John', age: 20};

[10, 12, 26, 15].findIndex(function (item) {//用第二个参数时，回调不能是箭头函数
     return item > this.age;
}, person);    // 2
```

是不是很简单，其实，我们自己去实现一个 findIndex 也很简单。

<br>

### 实现findIndex

思路自然很明了，遍历一遍，返回符合要求的值的下标即可。

```js

Array.prototype.findIndex = function (callBack, context) {
    var arr = this;
    for(var i = 0; i < arr.length; i++) {	
	if(callBack.call(context, arr[i], i, arr)) return i;
    }
    return -1;
	
    //this值指向array
    //若context值为undefined, 在非严格模式下指向windows
};

console.log([12, 5, 18, 44].findIndex(function (item) {
     return item > 15;
})); //2
```
<br>

# 2、ES6: findLastIndex

findIndex 是正序查找，但正如 indexOf 还有一个对应的 lastIndexOf 方法，我们也想写一个倒序查找的 findLastIndex 函数。实现自然也很简单，只要修改下循环即可。

```js
Array.prototype.findLastIndex = function(callBack, context) {
    var arr = this;
    for(var i = arr.length - 1; i >= 0; i--) {	
	if(callBack.call(context, arr[i], i, arr)) return i;
    }
    return -1;
};

console.log([12, 5, 18, 44].findLastIndex(function (item) {
     return item > 15;
})); //3
```

<br>

# 3、createIndexFinder: findIndex与findLastIndex合体

然而问题在于，findIndex 和 findLastIndex 其实有很多重复的部分，如何精简冗余的内容呢？这便是我们要学习的地方，日后面试问到此类问题，也是加分的选项。

underscore 的思路就是利用传参的不同，返回不同的函数。这个自然是简单，但是如何根据参数的不同，在同一个循环中，实现正序和倒序遍历呢？

让我们直接模仿 underscore 的实现：

```js
Array.prototype.createIndexFinder = function (direction) {
     direction = direction || 1;
	
     return function (callBack, context) {
          var arr = this;
	  var i = direction > 0 ? 0 : arr.length - 1;	
	  var step = direction > 0 ? 1 : -1;
	  
	  for (; i >= 0 && i < arr.length; i += step) {	
	        if(callBack.call(context, arr[i], i, arr)) return i;
	  }
	  return -1;
     }
};

Array.prototype.findIndex = Array.prototype.createIndexFinder(1);
Array.prototype.findLastIndex = Array.prototype.createIndexFinder(-1);
```


<br>

# 4、sortedIndex

findIndex 和 findLastIndex 的需求算是结束了，但是又来了一个新需求：在一个排好序的数组中找到 value 对应的位置，保证插入数组后，依然保持有序的状态。

假设该函数命名为 sortedIndex，效果为：

```js
[10, 20, 30].sortedIndex(25); // 2
```

也就是说如果，注意是如果，25 按照此下标插入数组后，数组变成 [10, 20, 25, 30]，数组依然是有序的状态。

那么这个又该如何实现呢？

既然是有序的数组，那我们就不需要遍历，大可以使用二分查找法，确定值的位置。让我们尝试着去写一版：

```js
// 第一版
Array.prototype.sortedIndex = function (obj) {
    var arr = this;
    var low = 0, 
        high = arr.length;

    while (low < high) {
      var mid = Math.floor((low + high) / 2);
      if(arr[mid] < obj) low = mid + 1;
      else high = mid;
    }

    return high;
};

console.log([10, 20, 30, 40, 50].sortedIndex(35)) // 3
```

现在的方法虽然能用，但通用性不够，比如我们希望能处理这样的情况：

```js
var stooges = [{name: 'stooge1', age: 10}, {name: 'stooge2', age: 30}];

var result = stooges.sortedIndex({name: 'stooge3', age: 20}, function(stooge){
    return stooge.age
});

console.log(result) // 1
```

所以我们还需要再加上一个参数 iteratee 函数对数组的每一个元素进行处理，一般这个时候，还会涉及到 this 指向的问题，所以我们再传一个 context 来让我们可以指定 this，那么这样一个函数又该如何写呢？

```js
// 第二版
function cb(func, context) {
       if (context === void 0) return func;
       return function() {
           return func.apply(context, arguments);
      };
}

Array.prototype.sortedIndex = function (obj, iteratee, context) {
    iteratee = cb(iteratee, context);

    var low = 0, 
	high = this.length;
	
    while (low < high) {
        var mid = Math.floor((low + high) / 2);
        if (iteratee(this[mid]) < iteratee(obj)) low = mid + 1;
        else high = mid;
    }
    return high;
};
```


<br>

# 5、indexOf

sortedIndex 也完成了，现在我们尝试着去写一个 indexOf 和 lastIndexOf 函数，学习 findIndex 和 FindLastIndex 的方式，我们写一版：

### （1）第一版：

```js
// 第一版
Array.prototype.createIndexOfFinder = function (dir) {
  dir = dir || 1;
  
  return function (obj) {
    var arr = this;

    for (
      var i = dir > 0 ? 0 : arr.length - 1, 
      	  step = dir > 0 ? 1 : -1;
      i >= 0 && i < arr.length;
      i += step
    ) {
      if (obj === arr[i]) return i;
    }
    
    return -1;
  };
};

Array.prototype.indexOf = Array.prototype.createIndexOfFinder(1);
Array.prototype.lastIndexOf = Array.prototype.createIndexOfFinder(-1);

var result = [1, 2, 3, 4, 5].indexOf(2);
var result2 = [1, 2, 3, 4, 5].lastIndexOf(2);

console.log(result, result2); // 1 1
```

<br>


### （2）第二版

但是即使是数组的 indexOf 方法也可以多传递一个参数 fromIndex，从 MDN 中看到 fromIndex 的讲究可有点多：

>设定开始查找的位置。如果该索引值大于或等于数组长度，意味着不会在数组里查找，返回 -1。如果参数中提供的索引值是一个负值，则将其作为数组末尾的一个抵消，即 -1 表示从最后一个元素开始查找，-2 表示从倒数第二个元素开始查找 ，以此类推。 注意：如果参数中提供的索引值是一个负值，仍然从前向后查询数组。如果抵消后的索引值仍小于 0，则整个数组都将会被查询。其默认值为 0。

再看看 lastIndexOf 的 fromIndex：

>从此位置开始逆向查找。默认为数组的长度减 1，即整个数组都被查找。如果该值大于或等于数组的长度，则整个数组会被查找。如果为负值，将其视为从数组末尾向前的偏移。即使该值为负，数组仍然会被从后向前查找。如果该值为负时，其绝对值大于数组长度，则方法返回 -1，即数组不会被查找。

按照这么多的规则，我们尝试着去写第二版：

```js
// 第二版
Array.prototype.createIndexOfFinder = function (dir) {
  return function (obj, idx) {
    var arr = this;
    var i = 0,
      length = arr.length;

    if (typeof idx == "number") {
        if (dir > 0) {
            i = idx >= 0 ? idx : Math.max(length + idx, 0);
        }else {
            length = idx >= 0 ? Math.min(idx + 1, length) : idx + length + 1;
        }
    }

    for (idx = dir > 0 ? i : length - 1; idx >= 0 && idx < length; idx += dir) {
      if (obj === arr[idx]) return idx;
    }
    return -1;
  };
};

Array.prototype.indexOf = Array.prototype.createIndexOfFinder(1);
Array.prototype.lastIndexOf = Array.prototype.createIndexOfFinder(-1);
```

<br>

### （3）第三版

到此为止，已经很接近原生的 indexOf 函数了，但是 underscore 在此基础上还做了两点优化。

第一个优化是支持查找 NaN。

因为 NaN 不全等于 NaN，所以原生的 indexOf 并不能找出 NaN 的下标。

```js
[1, NaN].indexOf(NaN) // -1
```

那么我们该如何实现这个功能呢？

就是从数组中找到符合条件的值的下标嘛，不就是我们最一开始写的 findIndex 吗？

我们来写一下：

```js
// 第三版
Array.prototype.createIndexOfFinder = function (dir, predicate) {
    return function(item, idx){
	var arr = this;
        if () { ... }

        // 判断元素是否是 NaN
        if (item !== item) {
            // 在截取好的数组中查找第一个满足isNaN函数的元素的下标
            idx = predicate(arr.slice(i, length), isNaN)
            return idx >= 0 ? idx + i: -1;
        }

        for () { ... }
    }
}

Array.prototype.indexOf = Array.prototype.createIndexOfFinder(1, findIndex);
Array.prototype.lastIndexOf = Array.prototype.createIndexOfFinder(-1, findLastIndex);
```

### （4）第四版
第二个优化是支持对有序的数组进行更快的二分查找。

如果 indexOf 第三个参数不传开始搜索的下标值，而是一个布尔值 true，就认为数组是一个排好序的数组，这时候，就会采用更快的二分法进行查找，这个时候，可以利用我们写的 sortedIndex 函数。

在这里直接给最终的源码：

```js
// 第四版
Array.prototype.createIndexOfFinder = function (dir, predicate, sortedIndex) {
    return function(item, idx){
    	var array = this;
        var length = array.length;
        var i = 0;

        if (typeof idx == "number") {
            if (dir > 0) {
                i = idx >= 0 ? idx : Math.max(length + idx, 0);
            }
            else {
                length = idx >= 0 ? Math.min(idx + 1, length) : idx + length + 1;
            }
        }else if (sortedIndex && idx && length) {
            idx = sortedIndex(array, item);
            // 如果该插入的位置的值正好等于元素的值，说明是第一个符合要求的值
            return array[idx] === item ? idx : -1;
        }

        // 判断是否是 NaN
        if (item !== item) {
            idx = predicate(array.slice(i, length), isNaN)
            return idx >= 0 ? idx + i: -1;
        }

        for (idx = dir > 0 ? i : length - 1; idx >= 0 && idx < length; idx += dir) {
            if (array[idx] === item) return idx;
        }
        return -1;
    }
}

Array.prototype.indexOf = Array.prototype.createIndexOfFinder(1, findIndex, sortedIndex);
Array.prototype.lastIndexOf = Array.prototype.createIndexOfFinder(-1, findLastIndex);
```

值得注意的是：在 underscore 的实现中，只有 indexOf 是支持有序数组使用二分查找，lastIndexOf 并不支持。
