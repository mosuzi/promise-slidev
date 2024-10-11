---
layout: two-cols-title
columns: is-7
---

::title::

# resolve promise

```js
const resolvePromise = function(promise, x) {}
```

::left::

````md magic-move
```js {1-13}
const resolvePromise = function (promise, x) {
  // x 与 promise 相同，则以一个 TypeError 拒绝 promise
  if (x === promise) {
    rejectPromise(promise, 
      new TypeError('can not be the same promise'))
  } else if (isPromise(x)) {
    // 如果 x 是一个 promise
    if (x.status === STATUS.FULFILLED) {
      // 当 x 已满足时，以 x 的结果值满足 promise
      fulfillPromise(promise, x.value)
    } else if (x.status === STATUS.REJECTED) {
      // 当 x 被拒绝时，以 x 的拒因拒绝 promise
      rejectPromise(promise, x.reason)
    } else {
    }
  } else if (isObject(x) || isFunction(x)) {
  } else {
  }
}
```

```js {1,5-17}
const resolvePromise = function (promise, x) {
  if (x === promise) {
  } else if (isPromise(x)) {
    if (x.status === STATUS.FULFILLED) { ...
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
  } else {
  }
}
```

```js {1,4-14}
const resolvePromise = function (promise, x) {
  if (x === promise) {
  } else if (isPromise(x)) {
  } else if (isObject(x) || isFunction(x)) {
    // 如果 x 是一个对象或者函数
    let then // 准备接收 x.then
    let called = false // 忽略多次调用 x.then
    try {
      then = x.then
    } catch (e) {
      // 任何异常都会潜在拒绝 promise
      rejectPromise(promise, e)
      return
    }
    // 如果 then 是一个方法
    if (isFunction(then)) {}
    else {}
  } else {
  }
}
```

```js
if (isObject(x) || isFunction(x)) {
  // other code
  // 如果 then 是一个方法
  if (isFunction(then)) {
    try {
      // 尝试以 x 为 this 调用 then
      then.call(x, (v) => {
        if (!called) {
          (called = true, resolvePromise(promise, v))
        }
      }, (r) => {
        if (!called) {
          (called = true, rejectPromise(promise, r))
        }
      })
    } catch (e) {
      // 任何异常都会潜在拒绝 promise
      if (!called) {
        (called = true, rejectPromise(promise, e))
      }
    }
  }
}
```

```js {1,7-12}
const resolvePromise = function (promise, x) {
  if (x === promise) {
  } else if (isPromise(x)) {
  } else if (isObject(x) || isFunction(x)) {
    // other code
    if (isFunction(then)) {}
    else {
      fulfillPromise(promise, x)
    }
  } else {
    fulfillPromise(promise, x)
  }
}
```
````

::right::

```js
const isPromise = function (promise) {
  return promise instanceof MyPromise
}
```

<div v-click="2">

```js
const isObject = function(obj) {
  return Object.prototype.toString.call(obj)
    .toLocaleLowerCase() === '[object object]'
}
```

</div>
