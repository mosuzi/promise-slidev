---
title: 一步一步手写 Promise
date: 2024-10-10 15:10:47
cover: https://pics.mosuzi.com/blog/interview.webp
category: [开发碎片]
tag: [面试, js]
---
# 一步一步手写 Promise

> 根据 Promises/A+ 规范实现 Promise

## Promise/A+ 规范

Promises/A+ 规范（以下简称规范）可以在 [Promises/A+](https://promisesaplus.com/) 查看，需要中文可以选择 AI 翻译或者 [Promise/A+ 规范](https://tsejx.github.io/javascript-guidebook/standard-built-in-objects/control-abstraction-objects/promise-standard/)

简单来说，符合 Promises/A+ 规范的 promise（以下简称 promise ）表示异步操作的最终结果，其应该是一个具有 `then` 方法的对象或者函数，该 promise 的 then 方法的行为符合规范。promise 必须处于 `pending`、`fulfilled`、`rejected` 三种状态之一，且其状态只允许由 `pending` 转换为其他两种状态——即其他两种状态不允许改变。当处于 `fulfilled` 状态时，promise 具有一个不可修改的值，表示异步处理的结果值；当处于 `rejected` 状态时，promise 具有一个不可修改的拒因(reason)，表示异步处理被拒绝（或者说失败）的原因

## 构造

{% tabs 构造Promise %}

<!-- tab Promise 类基本骨架 -->

按照 ES6 中使用 Promise 的例子，此处将 Promise 定义为一个类

```js
class MyPromise {
  constructor() {}
  then() {}
}
```

<!-- endtab -->

<!-- tab 增加状态、最终值和拒因 -->

根据规范，Promise 拥有三种状态（语意上为**待定**、**已满足**、**被拒绝**），拥有异步处理的值和拒因，则构造函数应该写为

```js
class MyPromise {
  constructor() {
    this.status = 'pending'
    this.value = undefined
    this.reason = undefined
  }
  then() {}
}
```

<!-- endtab -->

<!-- tab 回调方法 -->

在实际使用 Promise 时，会在构造函数里传入一个接收两个函数参数的回调方法：

1. 第一个参数方法负责将 promise 置为 `fulfilled` 状态
2. 第二个参数方法负责将 promise 置为 `rejected` 状态

于是改造为以下写法，其中假设 `fulfillPromise` 方法可以实现上述功能1、`rejectPromise` 方法可以实现上述功能2

```js
class MyPromise {
  constructor(fn) {
    this.status = 'pending'
    this.value = undefined
    this.reason = undefined
    fn((value) => {
      fulfillPromise(this, value)
    }, (reason) => {
      rejectPromise(this, reason)
    })
  }
  then() {}
}
```

<!-- endtab -->

<!-- tab 实现修改 promise 状态的方法 -->

前述的 `fulfillPromise` 方法和 `rejectPromise` 方法根据规范要求，只需要将处于 `pending` 状态的 promise 的状态修改，并赋予对应的值或拒因即可

考虑到实现这两个方法时需要频繁交互 promise 的状态，因此将 promise 的三种状态提出为一个映射，避免每次都手写字符串做匹配或赋值

```js
const STATUS = {
  PENDING: 'pending',
  FULFILLED: 'fulfilled',
  REJECTED: 'rejected'
}

function fulfillPromise(promise, value) {
  // 不允许 pending 态以外的状态改变
  if (promise.status !== STATUS.PENDING) return
  promise.status = STATUS.FULFILLED
  promise.value = value
}

function rejectPromise(promise, reason) {
  // 不允许 pending 态以外的状态改变
  if (promise.status !== STATUS.PENDING) return
  promise.status = STATUS.REJECTED
  promise.reason = reason
}
```

到此，简单的 Promise 构造方法实现完毕

<!-- endtab -->

{% endtabs %}

## then 方法

此处需要回顾一下规范中的 `then` 方法

`then` 方法接收两个参数：

```js
promise2 = promise.then(onFulfilled, onRejected)
```

1. 其中的 `onFulfilled` 和 `onRejected` 都是可选参数，并且当它们各自不为函数时，会被忽略。它们各自只能在 promise 状态发生{% hideInline 相应变化（指变为 fulfilled 时调用 onFulfilled 并传入 promise 的值；变为 rejected 时调用 onRejected 并传入 promise 的拒因）,相应变化 %}后被调用，且只能被调用一次
2. `then` 方法可以被调用多次。当 promise 被满足时，按照原始调用顺序依次执行回调 `onFulfilled`，当 promise 被拒绝时，按照原始调用顺序依次执行回调 `onRejected`
3. `then` 方法必须返回一个 promise

由以上第1点得知，`onFulfilled` 或 `onRejected` 调用时：需要判断是否为函数，不为函数则忽略——**返回的新 promise 的值或拒因与当前 promise 相同**；需要判断 promise 的状态，避免 promise 不为 `pending` 时状态发生改变

由以上第2点可以得知，promise 需要有两个回调队列，满足依次回调 `onFulfilled` 或者 `onRejected`。而且在 `fulfillPromise` 和 `rejectPromise` 中应该依次调用相应的回调队列

由此得到：

```js
const isFunction = function(func) {
  return Object.prototype.toString.call(func).toLocaleLowerCase() === '[object function]'
}

// 执行回调队列
const runCbs = function(cbs, value) {
  cbs.forEach((cb) => {
    cb(value)
  })
}

function fulfillPromise(promise, value) {
  // original code
  runCbs(promise.fulfilledCallbacks, value)
}

function rejectPromise(promise, reason) {
  // original code
  runCbs(promise.rejectedCallbacks, reason)
}

class MyPromise {
  constructor(fn) {
    // other code
    this.fulfilledCallbacks = []
    this.rejectedCallbacks = []
  }
  then(onFulfilled, onRejected) {
    const promise1 = this
    const promise2 = new MyPromise(() => {})
    if (promise1.status === STATUS.FULFILLED) {
      if (isFunction(onFulfilled)) {
        // todo: 异步调用后执行回调
      } else {
        fulfillPromise(promise2, promise1.value)
      }
    } else if (promise1.status === STATUS.REJECTED) {
      if (isFunction(onRejected)) {
        // todo：异步调用后执行回调
      } else {
        rejectPromise(promise2, promise1.reason)
      }
    } else {
      // 此处若不做默认值处理，则会导致异步调用方法时报错，按照规范应该忽略
      onFulfilled = isFunction(onFulfilled) ? onFulfilled : (value) => { return value }
      onRejected = isFunction(onRejected) ? onRejected : (err) => { throw err }
      promise1.fulfilledCallbacks.push(() => {
        // 异步执行回调
      })
      promise1.rejectedCallbacks.push(() => {
        // 异步执行回调
      })
      /**
       * * 等待 promise 变为 fulfilled 或 rejected 后依次执行相应回调
       */
    }
    return promise2
  }
}
```

上述代码中的 `todo: 异步调用后执行回调` 意味着需要调用回调方法执行回调。规范中 2.2.4 提到，必须在[执行上下文](https://es5.github.io/#x10.3)堆栈仅包含平台代码时调用 `onFulfilled` 和 `onRejected` ，换句话说就是**需要异步执行回调**。这里简单使用 `setTimeout` 模拟一下，基本框架为：

```js
setTimeout(() => { // 
  try {
    /**
     * 此处在 promise 是 rejected 状态时:
     * 1. 调用的方法由 onFulfilled 改为 onRejected
     * 2. 传入的参数由 promise1.value 改为  promise1.reason
     */
    const x = onFulfilled(promise1.value) 
    resolvePromise(promise2, x)
  } catch(error) {
    rejectPromise(promise2, error)
  }
}, 0)
```

这里的 `resolvePromise` 方法，表示根据回调方法的返回值，决议(resolve)一个 promise

如果说 `then` 方法是一个 promise 对象的核心，那么这个 `resolvePromise` 方法就是 `then` 方法的核心，下文会提到，此处先按下不表

于是将这个异步调用也抽象成一个 `simulateAsyncCall` 方法：

```js
function simulateAsyncCall(promise, onCall, value) {
  setTimeout(() => {
    try {
      const x = onCall(value)
      resolvePromise(promise, x)
    } catch (e) {
      rejectPromise(promise, e)
    }
  }, 0)
}
```

然后补全 `todo` 处的代码为：

```js
then(onFulfilled, onRejected) {
  const promise1 = this
  const promise2 = new MyPromise(() => {})
  if (promise1.status === STATUS.FULFILLED) {
    if (isFunction(onFulfilled)) {
      simulateAsyncCall(promise2, onFulfilled, promise1.value)
    } else {
      fulfillPromise(promise2, promise1.value)
    }
  } else if (promise1.status === STATUS.REJECTED) {
    if (isFunction(onRejected)) {
      simulateAsyncCall(promise2, onRejected, promise1.reason)
    } else {
      rejectPromise(promise2, promise1.reason)
    }
  } else {
    // promise1.status === STATUS.PENDING
    // 此处若不做默认值处理，则会导致异步调用方法时报错，按照规范应该忽略
    onFulfilled = isFunction(onFulfilled) ? onFulfilled : (value) => { return value }
    onRejected = isFunction(onRejected) ? onRejected : (err) => { throw err }
    promise1.fulfilledCallbacks.push((value) => {
      simulateAsyncCall(promise2, onFulfilled, value)
    })
    promise2.rejectedCallbacks.push((reason) => {
      simulateAsyncCall(promise2, onRejected, reason)
    })
  }
  return promise2
}
```

{% hideToggle 此时的完整代码 %}

```js
// 使用 Object.freeze 避免对象被篡改
const STATUS = Object.freeze({
  PENDING: 'pending',
  FULFILLED: 'fulfilled',
  REJECTED: 'rejected'
})

const isFunction = function (func) {
  return Object.prototype.toString.call(func).toLocaleLowerCase() === '[object function]'
}

const runCallbacks = function (cbs, value) {
  // 顺次执行回调
  cbs.forEach((cb) => cb(value))
}

/**
 * 将 promise 置为 fulfilled 状态（如果可行的话）
 * @param {*} promise 准备改变的 promise 对象
 * @param {*} value promise 变成 fulfilled 状态时的结果值
 */
const fulfillPromise = function (promise, value) {
  // 不允许 pending 态以外的状态改变
  if (promise.status !== STATUS.PENDING) return
  promise.value = value
  promise.status = STATUS.FULFILLED
  runCallbacks(promise.fulfilledCallbacks, value)
}

/**
 * 将 promise 置为 rejected 状态（如果可行的话）
 * @param {*} promise 准备改变的 promise 对象
 * @param {*} reason promise 变成 rejected 状态时的拒因
 * @returns 
 */
const rejectPromise = function (promise, reason) {
  // 不允许 pending 态以外的状态改变
  if (promise.status !== STATUS.PENDING) return
  promise.status = STATUS.REJECTED
  promise.reason = reason
  runCallbacks(promise.rejectedCallbacks, reason)
}

/**
 * 决议 promise，根据 value 选择将 promise 置为 fulfilled 或 rejected 状态
 * @param {*} promise 要决议的 promise
 * @param {*} value 决议 promise 需要的值
 */
const resolvePromise = function(promise, value) {}

/**
 * 模拟异步调用并决议 promise
 * @param {*} promise 要决议的 promise
 * @param {*} func 决议 promise 时调用的回调方法
 * @param {*} value 决议 promise 时传入回调方法的参数，可以是结果值，也可以是拒因
 */
const simulateAsyncCall = function (promise, func, value) {
  setTimeout(() => {
    try {
      const x = func(value)
      resolvePromise(promise, x)
    } catch (e) {
      // 异步调用发生异常，立即以异常拒绝 promise
      rejectPromise(promise, e)
    }
  }, 0)
}

class MyPromise {
  constructor(fn) {
    this.status = STATUS.PENDING
    this.value = undefined
    this.reason = undefined
    this.fulfilledCallbacks = []
    this.rejectedCallbacks = []
    fn(
      (value) => {
        resolvePromise(this, value)
      },
      (reason) => {
        rejectPromise(this, reason)
      }
    )
  }
  then(onFulfilled, onRejected) {
    const promise1 = this
    const promise2 = new MyPromise(() => {})
    if (promise1.status === STATUS.FULFILLED) {
      if (isFunction(onFulfilled)) {
        simulateAsyncCall(promise2, onFulfilled, promise1.value)
      } else {
        /**
         * promise1 的状态是 fulfilled 并且 onFulfilled 不为函数
         * 按照规范，应该将返回的 promise2 也置为与 promise1 相同的状态
         */
        fulfillPromise(promise2, promise1.value)
      }
    } else if (promise1.status === STATUS.REJECTED) {
      if (isFunction(onRejected)) {
        simulateAsyncCall(promise2, onRejected, promise1.reason)
      } else {
        /**
         * promise1 的状态是 rejected 并且 onRejected 不为函数
         * 按照规范，应该将返回的 promise2 也置为与 promise1 相同的状态
         */
        rejectPromise(promise2, promise1.reason)
      }
    } else {
      // 这里的 else 相当于 else if (promise1.status === STATUS.PENDING)
      // 此处若不做默认值处理，则会导致异步调用方法时报错，按照规范应该忽略
      onFulfilled = isFunction(onFulfilled)
        ? onFulfilled
        : (value) => {
            return value
          }
      onRejected = isFunction(onRejected)
        ? onRejected
        : (err) => {
            throw err
          }
      // 将异步回调推入回调队列
      promise1.fulfilledCallbacks.push((value) => {
        simulateAsyncCall(promise2, onFulfilled, value)
      })
      promise1.rejectedCallbacks.push((reason) => {
        simulateAsyncCall(promise2, onRejected, reason)
      })
    }
    // then 方法必须返回一个 promise
    return promise2
  }
}
```

{% endhideToggle %}

## `resolve`：决议 promise

实现 `resolve` 之前，先要引入一个之前用不到的术语(Terminology)——`thenable`

> `thenable` 是一个定义 then 方法的对象或函数

听起来很像 promise？没错，按照规范的说法，这个 `thenable` 对象**至少有点像 promise（at least somewhat like a promise）**

规范区分 promise 和 thenable 想来是为了兼容不规范的 promise 实现

规范用了大篇幅介绍决议过程。决议过程将 promise 和 一个 x 值作为输入，因此可以假设决议方法 `resolvePromise` 的定义为：

```js
const resolvePromise = function(promise, x) {}
```

决议时，执行以下逻辑：

1. 如果 promise 和 x 是同一个值，则以一个 `TypeError` 拒绝 promise
2. 如果 x 是另一个 promise，则使用 x 的状态决议 promise
3. 如果 x 是一个对象或者函数，则尝试读取 `x.then`
   - 若读到的不是方法则以 x 为值满足 promise
   - 若是则以 x 为 this 调用它，传入与 promise 的 `then` 方法相似的两个回调函数，只不过这两个回调函数会调用 `resolvePromise` 和 `rejectPromise` 来处理 promise 的状态
   - 调用 `x.then` 时如果多次调用 `resolvePromise` 或者 `rejectPromise` 则应该只执行第一次调用而忽略后续调用
4. 如果 x 不是对象也不是函数，则以 x 为值满足 promise
5. 决议中的任何异常均会导致 promise 被以该异常拒绝（前提是 promise 仍处于 pending 状态）

对于第2点，所谓的使用 x 的状态决议 promise, 即指 x 已满足则 promise 也被满足, x 被拒绝则 promise 也被拒绝，x 待定则 promise 也待定，直到 x 被满足或者被拒绝。这个过程是很符合直觉的

对于第3点中的 `x.then` 是一个方法的情况，其实按照规范描述写代码可能会更直接一些

由此可以得到代码：


```js
const isObject = function(obj) {
  return Object.prototype.toString.call(obj).toLocaleLowerCase() === '[object object]'
}

const isPromise = function (promise) {
  return promise instanceof MyPromise
}

const resolvePromise = function (promise, x) {
  // x 与 promise 相同，则以一个 TypeError 拒绝 promise
  if (x === promise) {
    rejectPromise(promise, new TypeError('resolving promise must not use the same promise'))
  } else if (isPromise(x)) {
    // 如果 x 是一个 promise
    if (x.status === STATUS.FULFILLED) {
      // 当 x 已满足时，以 x 的结果值满足 promise
      fulfillPromise(promise, x.value)
    } else if (x.status === STATUS.REJECTED) {
      // 当 x 被拒绝时，以 x 的拒因拒绝 promise
      rejectPromise(promise, x.reason)
    } else {
      // 当 x 待定时，等待 x 被满足或者被拒绝
      x.then(
        (v) => {
          // x 被满足，则以 x 的结果值满足 promise
          fulfillPromise(promise, v)
        },
        (r) => {
          // x 被拒绝，则以 x 的拒因拒绝 promise
          rejectPromise(promise, r)
        }
      )
    }
  } else if (isObject(x) || isFunction(x)) {
    // 如果 x 是一个对象或者函数
    let then // 准备接收 x.then，此处设变量存储的原因可以见规范 3.5
    let called = false // 由于需要忽略多次调用 x.then，因此需要设置变量控制
    try {
      then = x.then
    } catch (e) {
      // 任何异常都会潜在拒绝 promise，此处有可能是无法取值导致的异常
      rejectPromise(promise, e)
      return
    }
    // 如果 then 是一个方法
    if (isFunction(then)) {
      try {
        // 尝试以 x 为 this 调用 then
        then.call(
          x,
          (v) => {
            // 此处传入的是 x.then 的 'onFulfilled' 方法，内部调用 resolvePromise 决议 promise
            if (!called) {
              called = true
              resolvePromise(promise, v)
            }
          },
          (r) => {
            // 此处传入的是 x.then 的 'onRejected' 方法，内部调用 rejectPromise 拒绝 promise
            if (!called) {
              called = true
              rejectPromise(promise, r)
            }
          }
        )
      } catch (e) {
        // 任何异常都会潜在拒绝 promise，此处还应该避免 x.then 的多次调用
        if (!called) {
          called = true
          rejectPromise(promise, e)
        }
      }
    } else {
      // 如果 then 不是一个方法，则以 x 为结果值满足 promise
      fulfillPromise(promise, x)
    }
  } else {
    // 如果 x 不是一个对象或者函数，则以 x 为结果值满足 promise
    fulfillPromise(promise, x)
  }
}
```

## 测试

Github 上的 [Promises/A+](https://github.com/promises-aplus) 除了提供了 Promises/A+ 规范以外，还提供了[测试自定义的 Promise 的方法](https://github.com/promises-aplus/promises-tests)

全局安装命令：

```shell
npm install promises-aplus-tests -g
```

局部安装可以参见上述仓库，这里不再赘述

安装完成后，对自定义的 Promise 文件增加如下修改：

```js
MyPromise.deferred = function () {
  const deferred = {}
  deferred.promise = new MyPromise((resolve, reject) => {
    deferred.resolve = resolve
    deferred.reject = reject
  })
  return deferred
}

module.exports = MyPromise
```

接着运行以下命令即可看到测试结果：

```shell
promises-aplus-tests promise.js
```

## 修正

可以看到有一些用例，没有通过。最终排查到的结果，是在构造 Promise 执行传入的同步方法时，调用了 `fulfillPromise` 而不是 `resolvePromise` 方法，改动如下：

```diff
// @other code
class MyPromise {
  constructor(fn) {
    this.status = STATUS.PENDING
    this.value = undefined
    this.reason = undefined
    this.fulfilledCallbacks = []
    this.rejectedCallbacks = []
    fn(
      (value) => {
-        fulfillPromise(this, value)
+        resolvePromise(this, value)
      },
      (reason) => {
        rejectPromise(this, reason)
      }
    )
  }
  // @other code
}
```

当控制台输出 <span style="color: #86B42B;">872 passing</span> 即表示所有用例通过，当前的自定义 Promise 完整实现了 Promises/A+ 规范

![all tests passing](https://pics.mosuzi.com/blog/promises-tests-all-passing.png)

## 完善

`isFunction` 和 `isObject` 方法均使用了 `Object.prototype.toString.call()` 这样的判断类型的方式，由此可以再抽象出一个 `isType` 方法，并将 `isFunction` 和 `isObject` 做如下改造：

```js
const isType = function(type) {
  return function(obj) {
    return Object.prototype.toString.call(obj).toLocaleLowerCase() === '[object ' + type + ']'
  }
}

const isObject = isType('object')

const isFunction = isType('function')
```

因此最终的 promise.js 文件内容为：

```js
// 使用 Object.freeze 避免对象被篡改
const STATUS = Object.freeze({
  PENDING: 'pending',
  FULFILLED: 'fulfilled',
  REJECTED: 'rejected'
})

const isType = function (type) {
  return function (obj) {
    return Object.prototype.toString.call(obj).toLocaleLowerCase() === '[object ' + type + ']'
  }
}

const isObject = isType('object')

const isFunction = isType('function')

const isPromise = function (promise) {
  return promise instanceof MyPromise
}

const runCallbacks = function (cbs, value) {
  // 顺次执行回调
  cbs.forEach((cb) => cb(value))
}

/**
 * 将 promise 置为 fulfilled 状态（如果可行的话）
 * @param {*} promise 准备改变的 promise 对象
 * @param {*} value promise 变成 fulfilled 状态时的结果值
 */
const fulfillPromise = function (promise, value) {
  // 不允许 pending 态以外的状态改变
  if (promise.status !== STATUS.PENDING) return
  promise.value = value
  promise.status = STATUS.FULFILLED
  runCallbacks(promise.fulfilledCallbacks, value)
}

/**
 * 将 promise 置为 rejected 状态（如果可行的话）
 * @param {*} promise 准备改变的 promise 对象
 * @param {*} reason promise 变成 rejected 状态时的拒因
 * @returns
 */
const rejectPromise = function (promise, reason) {
  // 不允许 pending 态以外的状态改变
  if (promise.status !== STATUS.PENDING) return
  promise.status = STATUS.REJECTED
  promise.reason = reason
  runCallbacks(promise.rejectedCallbacks, reason)
}

/**
 * 决议 promise，根据 value 选择将 promise 置为 fulfilled 或 rejected 状态
 * @param {*} promise 要决议的 promise
 * @param {*} x 决议 promise 需要的值
 */
const resolvePromise = function (promise, x) {
  // x 与 promise 相同，则以一个 TypeError 拒绝 promise
  if (x === promise) {
    rejectPromise(promise, new TypeError('resolving promise must not use the same promise'))
  } else if (isPromise(x)) {
    // 如果 x 是一个 promise
    if (x.status === STATUS.FULFILLED) {
      // 当 x 已满足时，以 x 的结果值满足 promise
      fulfillPromise(promise, x.value)
    } else if (x.status === STATUS.REJECTED) {
      // 当 x 被拒绝时，以 x 的拒因拒绝 promise
      rejectPromise(promise, x.reason)
    } else {
      // 当 x 待定时，等待 x 被满足或者被拒绝
      x.then(
        (v) => {
          // x 被满足，则以 x 的结果值满足 promise
          fulfillPromise(promise, v)
        },
        (r) => {
          // x 被拒绝，则以 x 的拒因拒绝 promise
          rejectPromise(promise, r)
        }
      )
    }
  } else if (isObject(x) || isFunction(x)) {
    // 如果 x 是一个对象或者函数
    let then // 准备接收 x.then，此处设变量存储的原因可以见规范 3.5
    let called = false // 由于需要忽略多次调用 x.then，因此需要设置变量控制
    try {
      then = x.then
    } catch (e) {
      // 任何异常都会潜在拒绝 promise，此处有可能是无法取值导致的异常
      rejectPromise(promise, e)
      return
    }
    // 如果 then 是一个方法
    if (isFunction(then)) {
      try {
        // 尝试以 x 为 this 调用 then
        then.call(
          x,
          (v) => {
            // 此处传入的是 x.then 的 'onFulfilled' 方法，内部调用 resolvePromise 决议 promise
            if (!called) {
              called = true
              resolvePromise(promise, v)
            }
          },
          (r) => {
            // 此处传入的是 x.then 的 'onRejected' 方法，内部调用 rejectPromise 拒绝 promise
            if (!called) {
              called = true
              rejectPromise(promise, r)
            }
          }
        )
      } catch (e) {
        // 任何异常都会潜在拒绝 promise，此处还应该避免 x.then 的多次调用
        if (!called) {
          called = true
          rejectPromise(promise, e)
        }
      }
    } else {
      // 如果 then 不是一个方法，则以 x 为结果值满足 promise
      fulfillPromise(promise, x)
    }
  } else {
    // 如果 x 不是一个对象或者函数，则以 x 为结果值满足 promise
    fulfillPromise(promise, x)
  }
}

/**
 * 模拟异步调用并决议 promise
 * @param {*} promise 要决议的 promise
 * @param {*} func 决议 promise 时调用的回调方法
 * @param {*} value 决议 promise 时传入回调方法的参数，可以是结果值，也可以是拒因
 */
const simulateAsyncCall = function (promise, func, value) {
  setTimeout(() => {
    try {
      const x = func(value)
      resolvePromise(promise, x)
    } catch (e) {
      // 异步调用发生异常，立即以异常拒绝 promise
      rejectPromise(promise, e)
    }
  }, 0)
}

class MyPromise {
  constructor(fn) {
    this.status = STATUS.PENDING
    this.value = undefined
    this.reason = undefined
    this.fulfilledCallbacks = []
    this.rejectedCallbacks = []
    fn(
      (value) => {
        resolvePromise(this, value)
      },
      (reason) => {
        rejectPromise(this, reason)
      }
    )
  }
  then(onFulfilled, onRejected) {
    const promise1 = this
    const promise2 = new MyPromise(() => {})
    if (promise1.status === STATUS.FULFILLED) {
      if (isFunction(onFulfilled)) {
        simulateAsyncCall(promise2, onFulfilled, promise1.value)
      } else {
        /**
         * promise1 的状态是 fulfilled 并且 onFulfilled 不为函数
         * 按照规范，应该将返回的 promise2 也置为与 promise1 相同的状态
         */
        fulfillPromise(promise2, promise1.value)
      }
    } else if (promise1.status === STATUS.REJECTED) {
      if (isFunction(onRejected)) {
        simulateAsyncCall(promise2, onRejected, promise1.reason)
      } else {
        /**
         * promise1 的状态是 rejected 并且 onRejected 不为函数
         * 按照规范，应该将返回的 promise2 也置为与 promise1 相同的状态
         */
        rejectPromise(promise2, promise1.reason)
      }
    } else {
      // 这里的 else 相当于 else if (promise1.status === STATUS.PENDING)
      // 此处若不做默认值处理，则会导致异步调用方法时报错，按照规范应该忽略
      onFulfilled = isFunction(onFulfilled)
        ? onFulfilled
        : (value) => {
            return value
          }
      onRejected = isFunction(onRejected)
        ? onRejected
        : (err) => {
            throw err
          }
      // 将异步回调推入回调队列
      promise1.fulfilledCallbacks.push((value) => {
        simulateAsyncCall(promise2, onFulfilled, value)
      })
      promise1.rejectedCallbacks.push((reason) => {
        simulateAsyncCall(promise2, onRejected, reason)
      })
    }
    // then 方法必须返回一个 promise
    return promise2
  }
}

MyPromise.deferred = function () {
  const deferred = {}
  deferred.promise = new MyPromise((resolve, reject) => {
    deferred.resolve = resolve
    deferred.reject = reject
  })
  return deferred
}

module.exports = MyPromise
```
