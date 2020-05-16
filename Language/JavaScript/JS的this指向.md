### 什么是this指向

ECMAScript规范中这样写：

```
this 关键字执行为当前执行环境的 ThisBinding
```

MDN中描述:

```
In most cases, the value of this is determined by how a function is called.
在绝大多数情况下，函数的调用方式决定了this的值。
```

可以看出，在JavaScript中，this的指向是调用时决定的，而不是创建时决定的，这就会导致this的指向会让人迷惑，简单来说，**this具有运行期绑定的特性**。

### this指向有哪几种

1. 全局环境中，this默认绑定到window；
2. 由上下文对象调用，绑定到上下文对象；
    - 在一个函数上下文中，this由调用者提供，由调用函数的方式来决定。**如果调用者函数，被某一个对象所拥有，那么该函数在调用时，内部的this指向该对象。如果函数独立调用，那么该函数内部的this，则指向undefined**。但是在非严格模式中，当this指向undefined时，它会被自动指向全局对象。
3. 通过call()、apply()、bind()方法把对象绑定到this上；
4. new调用，绑定到新创建的对象；
   - 构造函数通常不使用return关键字，它们通常初始化新对象，当构造函数的函数体执行完毕时，它会显式返回。在这种情况下，构造函数调用表达式的计算结果就是这个新对象的值。
   - 如果构造函数使用return语句但没有指定返回值，或者返回一个原始值，那么这时将忽略返回值，同时使用这个新对象作为调用结果。
   - 如果构造函数显式地使用return语句返回一个对象，那么调用表达式的值就是这个对象。

*箭头函数不使用上面的绑定规则，根据外层作用域来决定this，继承外层函数调用的this绑定*

### 改变函数内部 this 指针的指向函数

1. apply：调用一个对象的一个方法，用另一个对象替换当前对象。例如：B.apply(A, arguments);即A对象应用B对象的方法；
2. call：调用一个对象的一个方法，用另一个对象替换当前对象。例如：B.call(A, args1,args2);即A对象调用B对象的方法
3. bind除了返回是函数以外，它的参数和call一样

### 箭头函数

所有的箭头函数都没有自己的this，都指向外层。

MDN中描述:

```
An arrow function does not create its own this, the this value of the enclosing execution context is used.
箭头函数会捕获其所在上下文的this值，作为自己的this值。
```

1. 箭头函数没有this，所以需要通过查找作用域链来确定this的值，这就意味着如果箭头函数被非箭头函数包含，this绑定的就是最近一层非箭头函数的this，
2. 箭头函数没有自己的arguments对象，但是可以访问外围函数的arguments对象
3. 不能通过new关键字调用，同样也没有new.target值和原型

### 参考

- https://www.jianshu.com/p/d647aa6d1ae6
- https://juejin.im/post/5c0c87b35188252e8966c78a