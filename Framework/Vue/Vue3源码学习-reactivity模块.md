reactivity 模块是 Vue3.0 的响应式系统，它有以下几个文件：

- baseHandlers.ts
- collectionHandlers.ts
- computed.ts
- effect.ts
- index.ts
- operations.ts
- reactive.ts
- ref.ts

小提示：
*阅读源码，可以先过一遍该模块下的 API，了解一下有哪些功能；然后再看一遍相关的单元测试，单元测试一般会把所有的功能细节都测一边；等对源码的功能有了了解，再去阅读源码的细节*

### setup函数

在这之前呢还需要知道一个函数 setup

- setup 是使用 Composition API 的入口
- setup 可以返回一个对象，该对象的属性会被合并到渲染上下文，并可以在模板中直接使用
- setup 也可以返回 render 函数

### reactive

reactive 接收一个普通对象然后返回该普通对象的响应式代理，它基于 ES2015 的 Proxy 实现，返回的代理对象不等于原始对象。建议仅使用代理对象而避免依赖原始对象。

```
<template>
  <div>
    <div>{{ state.num }}</div>
    <button @click="increment">count++</button>
  </div>
</template>

<script>
import { reactive } from "vue";

export default {
  setup() {
    let state = reactive({
      num: 0
    });

    const increment = () => state.num++;

    return {
      state,
      increment
    };
  }
};
</script>
```
*不要解构返回的代理对象，那样会使其失去响应性*

如果我们真的想展开state的属性，可以使用toRef和toRefs这两个API，进行转换成ref对象
```
<template>
  <div>
    <div>{{ num }}</div>
    <button @click="increment">count++</button>
  </div>
</template>

<script>
import { reactive, toRefs } from "vue";

export default {
  setup() {
    let state = reactive({
      num: 0
    });

    const increment = () => state.num++;

    return {
      ...toRefs(state),
      increment
    };
  }
};
</script>
```

接下来我们看一下 reactive 源码

打开vue-next/packages/reactivity/src/reactive.ts
```
export function reactive<T extends object>(target: T): UnwrapNestedRefs<T>
export function reactive(target: object) {
  // if trying to observe a readonly proxy, return the readonly version.
  if (target && (target as Target)[ReactiveFlags.IS_READONLY]) {
    return target
  }
  return createReactiveObject(
    target,
    false,
    mutableHandlers,
    mutableCollectionHandlers
  )
}
```

```
function createReactiveObject(
  target: Target,
  isReadonly: boolean,
  baseHandlers: ProxyHandler<any>,
  collectionHandlers: ProxyHandler<any>
) {
  // 前面都是一些对象是否已经proxy等的判断逻辑，这里就不展示了

  const observed = new Proxy(
    target,
    collectionTypes.has(target.constructor) ? collectionHandlers : baseHandlers
  );
  def(
    target,
    isReadonly ? ReactiveFlags.READONLY : ReactiveFlags.REACTIVE,
    observed
  );
  return observed;
}
```
那么这里最重要的就是new Proxy了，可以说理解 vue3 的响应式原理过程就是理解这个proxy创建的过程，而了解这个过程，主要就是看第二个参数，在这里就是collectionHandlers或者baseHandlers，大部分是后者，前者主要针对，Set、Map 等。

那么我们就来看baseHandlers：
```
export const mutableHandlers: ProxyHandler<object> = {
  get,
  set,
  deleteProperty,
  has,
  ownKeys,
};
```

#### get

可见 vue 代理了这几个操作，那么我们一个个看这几个操作做了啥：
```
function get(target: object, key: string | symbol, receiver: object) {
    if (key === ReactiveFlags.IS_REACTIVE) {
      return !isReadonly
    } else if (key === ReactiveFlags.IS_READONLY) {
      return isReadonly
    } else if (
      key === ReactiveFlags.RAW &&
      receiver ===
        (isReadonly
          ? (target as any)[ReactiveFlags.READONLY]
          : (target as any)[ReactiveFlags.REACTIVE])
    ) {
      return target
    }

    // 关于数组的一些特殊处理
    const targetIsArray = isArray(target)
    if (targetIsArray && hasOwn(arrayInstrumentations, key)) {
      return Reflect.get(arrayInstrumentations, key, receiver)
    }

    // 获取请求值
    const res = Reflect.get(target, key, receiver)

    if (
      isSymbol(key)
        ? builtInSymbols.has(key)
        : key === `__proto__` || key === `__v_isRef`
    ) {
      return res
    }

    if (!isReadonly) {
      track(target, TrackOpTypes.GET, key)
    }

    if (shallow) {
      return res
    }

    if (isRef(res)) {
      // ref unwrapping, only for Objects, not for Arrays.
      return targetIsArray ? res : res.value
    }

    if (isObject(res)) {
      // Convert returned value into a proxy as well. we do the isObject check
      // here to avoid invalid value warning. Also need to lazy access readonly
      // and reactive here to avoid circular dependency.
      return isReadonly ? readonly(res) : reactive(res)
    }

    return res
}
```

这里的重点就在于track(target, TrackOpTypes.GET, key);，这个是我们正常获取值的时候都会执行到的代码：
```
export function track(target: object, type: TrackOpTypes, key: unknown) {
  if (!shouldTrack || activeEffect === undefined) {
    return;
  }
  let depsMap = targetMap.get(target);
  if (!depsMap) {
    targetMap.set(target, (depsMap = new Map()));
  }
  let dep = depsMap.get(key);
  if (!dep) {
    depsMap.set(key, (dep = new Set()));
  }
  if (!dep.has(activeEffect)) {
    dep.add(activeEffect);
    activeEffect.deps.push(dep);
    if (__DEV__ && activeEffect.options.onTrack) {
      activeEffect.options.onTrack({
        effect: activeEffect,
        target,
        type,
        key,
      });
    }
  }
}
```
第一个if，就是根据环境变量的shouldTrack来判断是否要进行跟踪，可以参考processComponent的内容。因为在执行processComponent里面的setup的时候，我们特地关闭了track，而那时候就是把shouldTrack改为了false。

let depsMap = targetMap.get(target)这行，targetMap是一个 map，用来记录所有的响应对象。之后如果目前没有记录该对象，那么就重新记录。

这个 target 对应的也是一个 map，他会对每个 key 建立一个 set。

最后要记录这次的 effect 了，这里所谓的effect是什么呢？就是当前正在调用这个对象的函数。比如：
```
const Comp = {
  setup() {
    const state = reactive({ a: 1 });
    return () => {
      return <div>{state.a}</div>;
    };
  },
};
```
在我们执行 return 的 render 方法的时候，我们调用了state.a，这个时候 proxy 就调用了track，那么这时候的activeEffect就是这个 render 方法。这里的effect就是，当state.a改动的时候，我们需要重新执行该 render 方法来进行渲染。

在执行render方法的时候，我们执行这样一句代码instance.update = effect(function componentEffect()...)，就是在这里调用的effect方法里面，把activeEffect记录为componentEffect，而这个componentEffect里面则运行了render方法。

这个getHandler里面有句代码也挺有意思的：
```
if (isObject(res)) {
  return isReadonly ? readonly(res) : reactive(res);
}
```
在获取属性的时候，如果返回的值是对象的时候，才会对其执行reactive，这也就是延迟处理，而且readonly的话是不执行reactive的。

#### set

```
function createSetter(shallow = false) {
  return function set(
    target: object,
    key: string | symbol,
    value: unknown,
    receiver: object
  ): boolean {
    const oldValue = (target as any)[key];
    if (!shallow) {
      value = toRaw(value);
      if (!isArray(target) && isRef(oldValue) && !isRef(value)) {
        oldValue.value = value;
        return true;
      }
    } else {
      // in shallow mode, objects are set as-is regardless of reactive or not
    }

    const hadKey = hasOwn(target, key);
    const result = Reflect.set(target, key, value, receiver);
    // don't trigger if target is something up in the prototype chain of original
    if (target === toRaw(receiver)) {
      if (!hadKey) {
        trigger(target, TriggerOpTypes.ADD, key, value);
      } else if (hasChanged(value, oldValue)) {
        trigger(target, TriggerOpTypes.SET, key, value, oldValue);
      }
    }
    return result;
  };
}
```

这里不论是否是shallow，我们当作是 reactive 做的，那么唯一需要做区别的是ref，ref要通过ref.value来设置，而不是直接替换 value

而这里最重要的毫无疑问是trigger(target, TriggerOpTypes.ADD, key, value);，这就是用来触发我们之前在get的时候保存的effect的。

```
export function trigger(
  target: object,
  type: TriggerOpTypes,
  key?: unknown,
  newValue?: unknown,
  oldValue?: unknown,
  oldTarget?: Map<unknown, unknown> | Set<unknown>
) {
  const depsMap = targetMap.get(target);
  if (!depsMap) {
    // never been tracked
    return;
  }

  const effects = new Set<ReactiveEffect>();
  const add = (effectsToAdd: Set<ReactiveEffect> | undefined) => {
    if (effectsToAdd) {
      effectsToAdd.forEach((effect) => {
        if (effect !== activeEffect || !shouldTrack) {
          effects.add(effect);
        } else {
          // the effect mutated its own dependency during its execution.
          // this can be caused by operations like foo.value++
          // do not trigger or we end in an infinite loop
        }
      });
    }
  };

  if (type === TriggerOpTypes.CLEAR) {
    // collection being cleared
    // trigger all effects for target
    depsMap.forEach(add);
  } else if (key === "length" && isArray(target)) {
    depsMap.forEach((dep, key) => {
      if (key === "length" || key >= (newValue as number)) {
        add(dep);
      }
    });
  } else {
    // schedule runs for SET | ADD | DELETE
    if (key !== void 0) {
      add(depsMap.get(key));
    }
    // also run for iteration key on ADD | DELETE | Map.SET
    const isAddOrDelete =
      type === TriggerOpTypes.ADD ||
      (type === TriggerOpTypes.DELETE && !isArray(target));
    if (
      isAddOrDelete ||
      (type === TriggerOpTypes.SET && target instanceof Map)
    ) {
      add(depsMap.get(isArray(target) ? "length" : ITERATE_KEY));
    }
    if (isAddOrDelete && target instanceof Map) {
      add(depsMap.get(MAP_KEY_ITERATE_KEY));
    }
  }

  const run = (effect: ReactiveEffect) => {
    if (effect.options.scheduler) {
      effect.options.scheduler(effect);
    } else {
      effect();
    }
  };

  effects.forEach(run);
}
```

**收集该改动的依赖，然后分别执行这些依赖**

vue 在这里做了一些操作类型的区分：
```
export const enum TriggerOpTypes {
  SET = "set",
  ADD = "add",
  DELETE = "delete",
  CLEAR = "clear",
}
```

#### 其他可变处理器

```
function deleteProperty(target: object, key: string | symbol): boolean {
  const hadKey = hasOwn(target, key);
  const oldValue = (target as any)[key];
  const result = Reflect.deleteProperty(target, key);
  if (result && hadKey) {
    trigger(target, TriggerOpTypes.DELETE, key, undefined, oldValue);
  }
  return result;
}

function has(target: object, key: string | symbol): boolean {
  const result = Reflect.has(target, key);
  track(target, TrackOpTypes.HAS, key);
  return result;
}

function ownKeys(target: object): (string | number | symbol)[] {
  track(target, TrackOpTypes.ITERATE, ITERATE_KEY);
  return Reflect.ownKeys(target);
}

export const mutableHandlers: ProxyHandler<object> = {
  get,
  set,
  deleteProperty,
  has,
  ownKeys,
};
```
可见has，ownKeys是读取操作，所以调用跟踪，而delete则显然是修改操作，所以进行trigger
