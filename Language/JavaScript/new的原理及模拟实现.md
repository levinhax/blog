### new的过程

new 运算符创建一个用户定义的对象类型的实例或具有构造函数的内置对象的实例。new 关键字会进行如下的操作：

1. 创建一个新对象
2. 将构造函数的作用域赋给新对象(因此this就指向了这个新对象)
3. 执行构造函数中的代码(为这个新对象添加属性)
4. 返回新对象

```
        function Person(name, age, job) {
            this.name = name;
            this.age = age;
            this.job = job;
            this.sayName = function () {
                console.log('name: ', this.name)
            }
        }

        const person1 = new Person('levin', 20, 'Software Engineer');
```

即:

1. 创建一个空对象person1
2. 绑定原型，person1.__proto__ = Person.prototype
3. 调用 Person() 函数，并把空对象 person1 当做 this 传入，即 Person.call(person1)
4. 如果 Person() 函数执行完自己 return 一个 object 类型，那么返回此变量，否则返回 this，注意：如果构造函数返回基本类型值，则不影响，还是返回 this

### 模拟new

- new产生的实例可以访问Constructor里的属性，也可以访问到Constructor.prototype中的属性，前者可以通过apply来实现，后者可以通过将实例的proto属性指向构造函数的prototype来实现
- 我们还需要判断返回的值是不是一个对象，如果是一个对象，我们就返回这个对象，如果没有，我们该返回什么就返回什么

```
        function New() {
            let obj = new Object();
            // 取出第一个参数，就是我们要传入的构造函数；此外因为shift会修改原数组，所以arguments会被去除第一个参数
            let Constructor = [].shift.call(arguments);
            // 将obj的原型指向构造函数，这样obj就可以访问到构造函数原型中的属性
            obj.__proto__ = Constructor.prototype;
            // 使用apply改变构造函数this的指向到新建的对象，这样obj就可以访问到构造函数中的属性
            const ret = Constructor.apply(obj, arguments);
            // 要返回obj
            return (typeof ret === "object" || typeof ret === "function") && ret !== null ? ret : obj;
        }

        function Person(name) {
            this.name = name;
        }

        Person.prototype.getName = function () {
            return this.name;
        }

        const p1 = New(Person, 'leon');
        console.log(p1);
        console.log(p1.getName());
```

### 参考

- https://juejin.im/post/590a99015c497d005852cf26
- https://juejin.im/post/5bde7c926fb9a049f66b8b52
- https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/new