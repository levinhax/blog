## Proxy（代理）

Proxy 对象用于定义基本操作的自定义行为（如属性查找、赋值、枚举、函数调用等）。

```
const p = new Proxy(target, handler)
```

- target: 要使用 Proxy 包装的目标对象（可以是任何类型的对象，包括原生数组，函数，甚至另一个代理）
- handler: 一个通常以函数作为属性的对象，各属性中的函数分别定义了在执行各种操作时代理 p 的行为

Proxy翻译是代理的意思，我们调用new 创建一个和目标（traget）对象一致的虚拟化对象，然后去拦截并改变js引擎的底层操作，比如一些不可枚举、不可写入的属性。拦截的行为是个函数，可以修改js对象的内置行为，用于响应拦截的特定操作，我们通常将它称为陷阱。
```
      let target = {};
      let proxy = new Proxy(target, {});
      proxy.name = "proxyName"; // 操作转发到目标
      console.log(proxy.name); // proxyName
      console.log(target.name); // proxyName

      target.name = "targetName";
      console.log(proxy.name); // targetName
      console.log(target.name); // targetName
```
如上，将 proxyName 赋值给proxy.name时，代理就会将该操作转发给目标，执行name属性的创建；但是，它只是做转发而不会存储该属性；在他们之间存在一个相互引用，target.name 设置一个新值后，proxy.name 值也随之改变。

```
      const handler = {
        get: function(obj, prop) {
          console.log("handler get: ", prop);
          if (prop == "id") {
            return `id_${obj[prop]}`;
          }
          return obj[prop];
        },
        set: function(obj, prop, value) {
          if (typeof value !== "string") {
            throw new Error("Only string value can be stored in this object!");
          } else {
            obj[prop] = value;
          }
        }
      };
      const initobj = {
        id: 1,
        name: "Foo Bar"
      };
      const proxiedObject = new Proxy(initobj, handler);
      console.log(proxiedObject.id); // id_1
      console.log(proxiedObject.name); // Foo Bar
      proxiedObject.name = '12';
      proxiedObject.name = 12;
```

## Reflect（反射）

Reflect(反射)对象是给底层操作提供默认行为的方法的集合，这些操作可以被proxy代理重写。

Reflect与Proxy是相辅相成的，只要是Proxy对象的方法，就能在Reflect对象上找到对应的方法。

Proxy代理 | 被重写的行为 | Reflect反射
--|--|--
get | 获取对象身上某个属性的值 | Reflect.get()
set | 将值分配给属性 | Reflect.set()
has | 判断一个对象是否存在某个属性，和 in 运算符 的功能完全相同 | Reflect.has()
apply | 函数调用操作，可传入一个数组作为调用参数 | Reflect.apply()
construct | 对构造函数进行 new 操作 | Reflect.construct()
defineProperty | Object.defineProperty() | Reflect.defineProperty()
deleteProperty | delete运算符 | Reflect.deleteProperty()
getOwnPropertyDescriptor | Object.getOwnPropertyDescriptor()自有属性描述符 | Reflect.getOwnPropertyDescriptor()
getPrototypeOf | Object.getPrototypeOf()返回指定对象的原型 | Reflect.getPrototypeOf()
setPrototypeOf | Object.setPrototypeOf() | Reflect.setPrototypeOf()
isExtensible | Object.isExtensible()是否可扩展 | Reflect.isExtensible()
ownKeys | 自身属性键数组 | Reflect.ownKeys()
preventExtensions | 阻止新属性添加到对象 | Reflect.preventExtensions()

```
      const handler = {
        get: function(obj, prop) {
          // return obj[prop];
          return Reflect.get(obj, prop);
        },
        set: function(obj, prop, value) {
          // obj[prop] = value;
          return Reflect.set(obj, prop, value);
        },
        has(obj, prop) {
          return Reflect.has(obj, prop);
        }
      };
      const initobj = {
        id: 1,
        name: "initName"
      };
      const proxiedObject = new Proxy(initobj, handler);

      console.log(proxiedObject.name);
      console.log(Reflect.get(proxiedObject, "name"));
      console.log(Reflect.has(proxiedObject, "name"));
```

```
      class Person {
        constructor(name) {
          console.log("name: ", name);
        }
      }

      new Person("leon");
      Reflect.construct(Person, ["leon"]);


      const func = function(a, b) {
        console.log(this, a, b);
      };
      func.apply = () => {
        console.log("apply");
      };
      func.apply({ name: "leon" }, [1, 2]); // apply 调用的是自己身上的方法
      Function.prototype.apply.call(func, { name: "leon" }, [1, 2]); // { name: 'leon' } 1 2
      Reflect.apply(func, { name: "leon" }, [1, 2]); // { name: 'leon' } 1 2


      Reflect.apply(Math.floor, undefined, [1.75]); // 1
      Reflect.apply(String.fromCharCode, undefined, [104, 101, 108, 108, 111]); // "hello"
      Reflect.apply(RegExp.prototype.exec, /ab/, ["confabulation"]).index; // 4
      Reflect.apply("".charAt, "ponies", [3]); // "i"
```

**Reflect的核心就是将原Object的部分方法放到了Reflect上，未来的新方法将只部署在Reflect对象上**

参考：

- https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy
- https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Reflect
