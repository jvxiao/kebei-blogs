从Vue开始较大范围在前端应用开始，关于Vue一些基础知识的讨论和面试问题就在开发圈子里基本上就跟前几年的股票和基金一样，楼下摆摊卖酱香饼的阿姨都能说上几句那种。找过前端开发工作或者正在找开发工作的前端都知道，面试官基本上都有那么几个常问的问题，而网上呢也有那么一套可以用来背诵的“八股文”，自己懂多少没有关系，应付面试官还是够的，可以算是屡试不爽吧。

背诵面试八股文无可厚非的，可以说是每一个找工作的人都干过和必须干的事情，因为我们都要工作，都要恰饭。只有恰上饭，才能去谈些伟大的理想。背“八股文”本是一种捷径，尤其是本身对一门技术不是特别了解的开发者，就是那种刚刚能使用它那种。

在众多关于Vue的面试“八股文”中，今天讲的是其中最常问的一个--Vue中的依赖收集。本文也将从代码层面，讲清楚关于依赖收集的几个问题。
-  收集的依赖是什么？（what）
-  怎么收集的依赖？  （how）
-  什么时候收集？     (when)

至于为什么要收集依赖(why)，现在就可以先告诉答案。**收集依赖，其核心作用是在数据发生变化的时候可以做出相应的动作，比如刷新视图**，为了执行这一动作，我们就得知道是谁在什么时候发生了变化，所以我们要收集依赖。


下面我们结合代码，尽可能通俗的讲解关于上述的三个问题：



在搞清楚依赖收集之前，先把源码中几个概念性的东西说明一下，建议下载[Vue3源码](https://github.com/vuejs/core)进行对照着看：
-  **Dep**: 本质上是一个Map实例，同时在map实例上绑定一个celanup函数和一个computed属性。
- **ReactiveEffect**: 相当于2.x版本中的Watcher类, 里头有一个deps数组，用来存dep, 每个实例里面都有一个track_id用来标识唯一性。

- **effect函数**： 里头实例化一个ReactiveEffect对象，同时绑定一些options, 返回值是一个runner,实际上是对ReactiveEffect对象行为的一种业务封装。

下面以一行简单的代码开始关于依赖收集的探索。
```Javascript
const num = ref(1);
```
```Javascript
// packages/reactivity/src/ref.ts
export function ref<T>(value: T): Ref<UnwrapRef<T>>
export function ref<T = any>(): Ref<T | undefined>
export function ref(value?: unknown) {
  return createRef(value, false)
}
```
ref函数主要是对createRef做了一个函数包装，主要内容看到createRef函数。

```Javascript
// packages/reactivity/src/ref.ts
function createRef(rawValue: unknown, shallow: boolean) {
  if (isRef(rawValue)) {
    return rawValue
  }
  return new RefImpl(rawValue, shallow)
}
```
createRef函数在这里对原始数据rawValue做了一个判断，如果数据本身就是响应式数据了，就直接返回它本身，如果不是，就返回一个实例化的RefImpl对象。


```Javascript
// packages/reactivity/src/ref.ts
class RefImpl<T> {
  private _value: T
  private _rawValue: T

  public dep?: Dep = undefined
  public readonly __v_isRef = true

  constructor(
    value: T,
    public readonly __v_isShallow: boolean,
  ) {
    this._rawValue = __v_isShallow ? value : toRaw(value)
    this._value = __v_isShallow ? value : toReactive(value)
  }

  get value() {
    trackRefValue(this)
    return this._value
  }

  set value(newVal) {
    const useDirectValue =
      this.__v_isShallow || isShallow(newVal) || isReadonly(newVal)
    newVal = useDirectValue ? newVal : toRaw(newVal)
    if (hasChanged(newVal, this._rawValue)) {
      this._rawValue = newVal
      this._value = useDirectValue ? newVal : toReactive(newVal)
      triggerRefValue(this, DirtyLevels.Dirty, newVal)
    }
  }
}
```
重点来了，RefImple类里头，才是真正包含了从原始数据变成响应式数据，以及收集依赖的逻辑。在一个refImpl实例中，里面有一个dep对象，初始值是undefined， 这个dep会这trackRefValue函数执行的过程中被赋值。

下面代码从17-21(get value())行，就是依赖收集的过程：当一个ref型响应式数据通过.value访问时，会触发RefImpl实例中的getter。它会首先执行一个trackValue函数，然后再返回_value值，所以接下来重点看关注trackValue函数，所以**依赖是在数据被访问的时候触发的**。

```Javascript
// packages/reactivity/src/ref.ts
export function trackRefValue(ref: RefBase<any>) {
  if (shouldTrack && activeEffect) {
    ref = toRaw(ref)
    trackEffect(
      activeEffect,
      (ref.dep ??= createDep(
        () => (ref.dep = undefined),
        ref instanceof ComputedRefImpl ? ref : undefined,
      )),
      __DEV__
        ? {
            target: ref,
            type: TrackOpTypes.GET,
            key: 'value',
          }
        : void 0,
    )
  }
}u
```

trackRefValue函数中有两个变量，shouldTrack和activeEffect，暂时我们不去理会它们，只要知道shouldTrack是一个布尔值，activeEffect是一个RectiveEffect实例。

在shouldTrack值为true且activeEffect有值的情况下，首先会将ref转成原始值，然后再执行trackEffect函数。

在执行trackEffect函数的中，第一个是activeEffect, 在任意时刻它在全局是具有唯一性的；第二个是ref.dep, 其中给ref.dep的赋值函数createDep返回一个Dep实例，前面说过的，本质是个map; 第三个函数是个对象，是关于开发环境下debug的一些配置。

在这里，我们可以看到，之前说个的ref实例中原来是undefined的ref.dep赋值，就在此处。

```Javascript
// packages/reactivity/src/effect.ts
export function trackEffect(
  effect: ReactiveEffect,
  dep: Dep,
  debuggerEventExtraInfo?: DebuggerEventExtraInfo,
) {
  if (dep.get(effect) !== effect._trackId) {
    dep.set(effect, effect._trackId)
    const oldDep = effect.deps[effect._depsLength]
    if (oldDep !== dep) {
      if (oldDep) {
        cleanupDepEffect(oldDep, effect)
      }
      effect.deps[effect._depsLength++] = dep
    } else {
      effect._depsLength++
    }
    if (__DEV__) {
      effect.onTrack?.(extend({ effect }, debuggerEventExtraInfo!))
    }
  }
}
```
trackEffect函数绝对是依赖收集重头戏中的重头戏。

首先上来就是一个判断，dep, 也就是ref中的dep，本质是个map，判断里面是否存在对应的effect, 如果没有，就执行接下来的操作。

dep将effect也就是activeEffect作为键，其_trackId作为值添加到dep，**所以我们说的收集的依赖指的就是effect对象**。同时我们得到了一个关于**dep和effect之间的第一关系，即一个dep可以对应多个effect**。

接着，将effects实例中deps数组中最后一个值取出来与当前的dep值进行比对，看是否是同一个值如果不是同一个值，而且oldDep是有值的，那么就执行cleanupDepEffect操作。如果oldDep为空值，就跳过这一步，直接往effect.deps中添加dep。因此，我们在这里得到了关于dep和effect第二个结论，**一个effect可以对应多个dep**。

代码还有一部分，接着往下看，在oldDep不等于当前dep的时候，直接对effec_depsLength进行加操作，也就是说，effect.deps值没有变，但是_depsLength值却超出了deps数组边界的情况，这也就是为什么上面要判断oldDep是否存在的原因。


由上面上面两个结论我们可以得出，一个dep中可以对应多个effect, 一个effect也可以对应多个dep, 因此dep和effect的关系是多对多的关系。

### 总结

-  收集的依赖是什么？（what）
  > 我们常说的收集的依赖是effect对象

-  怎么收集的依赖？  （how）
 > 判断当前数据dep中有没有activeEffct, 没有就加进去。把大象关进冰箱里要几步！！！
-  什么时候收集？     (when)
> 在数据被访问时，触发getter，进行依赖收集
