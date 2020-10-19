# ref 和 computed

我们可以通过ref(data)来创建一个Ref，通过computed(() => {})来生成一个computed，在reactivity里面，ref和computed之间是有很多联系的

## ref

```
export function ref(value?: unknown) {
  return createRef(value)
}

function createRef(rawValue: unknown, shallow = false) {
  if (isRef(rawValue)) {
    return rawValue
  }
  let value = shallow ? rawValue : convert(rawValue)
  const r = {
    __v_isRef: true,
    get value() {
      track(r, TrackOpTypes.GET, 'value')
      return value
    },
    set value(newVal) {
      if (hasChanged(toRaw(newVal), rawValue)) {
        rawValue = newVal
        value = shallow ? newVal : convert(newVal)
        trigger(
          r,
          TriggerOpTypes.SET,
          'value',
          __DEV__ ? { newValue: newVal } : void 0
        )
      }
    }
  }
  return r
}
```
首先，入参rawValue如果已经是一个ref了，则直接返回，isRef的判断规则我们甚至不需要看他的实现，看下面返回的ref就知道了，在返回的ref里面有一个属性__v_isRef，就是用来标示该对象是否是一个ref对象的，那么isRef就很简单了。

```
export function isRef(r: any): r is Ref {
  return r ? r.__v_isRef === true : false
}
```
然后之后的代码是创建value，这里有个条件判断是基于shallow，这个是啥意思呢，这一章后面我们再讲，这里我们知道shallow是false，所以呢，会调用convert(newVal)，那么convert就是把这个对象转换成一个响应式对象

```
const convert = <T extends unknown>(val: T): T =>
  isObject(val) ? reactive(val) : val
```
之后就返回ref了，可以看到这是一个带有getter和setter的对象，其getter跟之前我们看reactive差不多，会调用track，而其setter则是根据value有木有变化来判断是否要触发effect，通过返回这样一个对象，我们在调用rev.value的时候就让vue知道我们现在运行的方法是依赖于这个ref的，在ref.value变化的时候，需要重新运行该方法。关于这个过程在reactive里面都讲了，那么这里就不重复讲了。

## computed

```
export function computed<T>(
  getterOrOptions: ComputedGetter<T> | WritableComputedOptions<T>
) {
  let getter: ComputedGetter<T>
  let setter: ComputedSetter<T>

  if (isFunction(getterOrOptions)) {
    getter = getterOrOptions
    setter = NOOP
  } else {
    getter = getterOrOptions.get
    setter = getterOrOptions.set
  }

  let dirty = true
  let value: T
  let computed: ComputedRef<T>

  const runner = effect(getter, {
    lazy: true,
    scheduler: () => {
      if (!dirty) {
        dirty = true
        trigger(computed, TriggerOpTypes.SET, 'value')
      }
    }
  })
  computed = {
    __v_isRef: true,
    [ReactiveFlags.IS_READONLY]:
      isFunction(getterOrOptions) || !getterOrOptions.set,

    // expose effect so computed can be stopped
    effect: runner,
    get value() {
      if (dirty) {
        value = runner()
        dirty = false
      }
      track(computed, TrackOpTypes.GET, 'value')
      return value
    },
    set value(newValue: T) {
      setter(newValue)
    }
  } as any
  return computed
}
```

computed相对复杂一点，但只要你理解了我们之前讲的reactive的代码，相信也是小菜一碟。首先判断我们传入的是否是一个方法，来对最终的getter和setter赋值，我们使用computed的时候可以有两种方式：
```
// 函数，只有getter
computed(() => {})

// 对象，getter和setter
computed({
  get() {}
  set() {}
})
```

接下来就是创建computed对象了，首先注意到，computed对象的类型是ComputedRef，也就是说，他其实也是个ref，其实很好理解，computed最终返回的是一个值，对于computed来说其本身是不关心值的内容的，这就跟ref一样了。

get的时候会先判断dirty，也就是说我们在获取值的时候可能他还没有更新，他就是在这个时候更新。这也是一种**延迟计算**。

# shallow

到这里为止，我们已经不止一次的看到shallow这个单词了，那么这到底是个什么东西呢？我们先来看一下跟shallow有关的API：

- shallowRef
- shallowReactive
- shallowReadonly
拢共是这三个，很显然这是对应于composition api的最主要的两种响应式数据reactive和ref的。

其实在看reactive的源码的时候我们已经看到过他的区别了，我把关键代码在这里再放一下：
```
const get = /*#__PURE__*/ createGetter()
const shallowGet = /*#__PURE__*/ createGetter(false, true)
```

在创建reactive的get handler的时候，会有两种选项，对于shallowGet我们传入了false, true，而在createGetter里面，我们看到这段代码：
```
if (shallow) {
  return res
}

if (isRef(res)) {
  return targetIsArray ? res : res.value
}

if (isObject(res)) {
  return isReadonly ? readonly(res) : reactive(res)
}
```

可以看到对于shallow的情况，在通过proxy获取到值之后，就直接返回了，也就是说返回的其实是原始对象的属性。而如果不是shallow的，则会返回reactive(res)，也就是说对于返回的对象，也会将其转换成响应式对象。

**一句话概括就是，shallowReactive返回的对象，我们通过state.obj获取的对象，如果我们对其进行属性读取，是不会引起追踪的！**

比如：
```
const Comp = () => {
  const state = reactive({
    obj: {
      a: 1
    }
  })

  const obj = state.obj

  const x = computed(() => {
    return obj.a + 1
  })
}
```

这个例子，我们的x会在obj.a变化之后发生变化。但是如果我们这里改用shallowReactive，那么这里的x是不会发生变化的。因为obj只是一个普通对象，无法进行追踪！

这就是shallow的意义，很多同学可能要问了，为什么我需要他？**很简单，有时候有些对象你不希望他被转换成reactive的，特别是你在开发一些插件的时候。**而且还有一点就是，你应该在不需要对于对象的属性使用reactive的情况下使用shallowReactive，因为shallowReactive的性能显然是会高一些，毕竟你在获取值的时候不需要执行一遍reactive。
