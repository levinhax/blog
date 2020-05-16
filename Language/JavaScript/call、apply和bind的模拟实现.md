### 一、call

call() 方法使用一个指定的 this 值和单独给出的一个或多个参数来调用一个函数。

```
function.call(thisArg, arg1, arg2, ...)
```

```
function Product(name, price) {
  this.name = name;
  this.price = price;
}

function Food(name, price) {
  Product.call(this, name, price);
  this.category = 'food';
}

console.log(new Food('cheese', 5).name); // cheese
```

#### 模拟实现一个call

- 将函数设为对象的属性
- 执行&删除这个函数
- 指定this到函数并传入给定参数执行函数
- 如果不传入参数，默认指向为 window

```
// 模拟 call bar.myCall(null);
//实现一个call方法：
Function.prototype.myCall = function(context) {
  //此处没有考虑context非object情况
  context.fn = this;
  let args = [];
  for (let i = 1, len = arguments.length; i < len; i++) {
    args.push(arguments[i]);
  }
  context.fn(...args);
  let result = context.fn(...args);
  delete context.fn;
  return result;
};
```

### 二、apply

apply() 方法调用一个具有给定this值的函数，以及作为一个数组（或类似数组对象）提供的参数。

*注意：call()方法的作用和 apply() 方法类似，区别就是call()方法接受的是参数列表，而apply()方法接受的是一个参数数组。*

```
const numbers = [5, 6, 2, 3, 7];
const max = Math.max.apply(null, numbers);
console.log(max); // 7
```

#### 模拟实现apply

```
// 模拟 apply
Function.prototype.myApply = function(context, arr) {
  var context = Object(context) || window;
  context.fn = this;

  var result;
  if (!arr) {
    result = context.fn();
  } else {
    var args = [];
    for (var i = 0, len = arr.length; i < len; i++) {
      args.push("arr[" + i + "]");
    }
    result = eval("context.fn(" + args + ")");
  }

  delete context.fn;
  return result;
};
```

### 三、bind

bind() 方法创建一个新的函数，在 bind() 被调用时，这个新函数的 this 被指定为 bind() 的第一个参数，而其余参数将作为新函数的参数，供调用时使用。它返回一个原函数的拷贝，并拥有指定的 this 值和初始参数。

```
function.bind(thisArg[, arg1[, arg2[, ...]]])
```

```
const module = {
  x: 42,
  getX: function() {
    return this.x;
  }
};

const unboundGetX = module.getX;
console.log(unboundGetX()); // undefined (The function gets invoked at the global scope)

const boundGetX = unboundGetX.bind(module);
console.log(boundGetX()); // 42
```

#### 模拟实现bind

- 返回一个函数，绑定this，传递预置参数
- bind返回的函数可以作为构造函数使用。故作为构造函数时应使得this失效，但是传入的参数依然有效

```
// mdn的实现
if (!Function.prototype.bind) {
  Function.prototype.bind = function(oThis) {
    if (typeof this !== 'function') {
      // closest thing possible to the ECMAScript 5
      // internal IsCallable function
      throw new TypeError('Function.prototype.bind - what is trying to be bound is not callable');
    }

    var aArgs   = Array.prototype.slice.call(arguments, 1),
        fToBind = this,
        fNOP    = function() {},
        fBound  = function() {
          // this instanceof fBound === true时,说明返回的fBound被当做new的构造函数调用
          return fToBind.apply(this instanceof fBound
                 ? this
                 : oThis,
                 // 获取调用时(fBound)的传参.bind 返回的函数入参往往是这么传递的
                 aArgs.concat(Array.prototype.slice.call(arguments)));
        };

    // 维护原型关系
    if (this.prototype) {
      // Function.prototype doesn't have a prototype property
      fNOP.prototype = this.prototype;
    }
    // 下行的代码使fBound.prototype是fNOP的实例,因此
    // 返回的fBound若作为new的构造函数,new生成的新对象作为this传入fBound,新对象的__proto__就是fNOP的实例
    fBound.prototype = new fNOP();

    return fBound;
  };
}
```

### 参考

- https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/call
- https://github.com/mqyqingfeng/Blog/issues/11
- https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/apply
- https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/bind
- https://github.com/mqyqingfeng/Blog/issues/12