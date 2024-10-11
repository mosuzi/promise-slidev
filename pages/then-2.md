---
layout: two-cols-title
---

::title::

# then

```js
promise2 = promise.then(onFulfilled, onRejected)
```

::left::

````md magic-move
```js
class MyPromise {
  constructor(fn) {
    // other code
  }
  then() {}
}
```

```js
class MyPromise {
  constructor(fn) {
    // other code
  }
  then(onFulfilled, onRejected) {
    const promise1 = this
    const promise2 = new MyPromise(() => {})
    return promise2
  }
}
```

```js {*|8-13}
class MyPromise {
  constructor(fn) {
    // other code
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
      //
    } else {
      //
    }
    return promise2
  }
}
```

```js {10-15}
class MyPromise {
  constructor(fn) {
    // other code
  }
  then(onFulfilled, onRejected) {
    const promise1 = this
    const promise2 = new MyPromise(() => {})
    if (promise1.status === STATUS.FULFILLED) {
      //
    } else if (promise1.status === STATUS.REJECTED) {
      if (isFunction(onRejected)) {
        // todo：异步调用后执行回调
      } else {
        rejectPromise(promise2, promise1.reason)
      }
    } else {
      //
    }
    return promise2
  }
}
```

```js {4-5,13-18}
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
    } else if (promise1.status === STATUS.REJECTED) {
    } else {
      promise1.fulfilledCallbacks.push((value) => {
        // 异步执行回调
      })
      promise1.rejectedCallbacks.push((reason) => {
        // 异步执行回调
      })
    }
    return promise2
  }
}
```

```js {9-14}
class MyPromise {
  constructor(fn) {}
  then(onFulfilled, onRejected) {
    const promise1 = this
    const promise2 = new MyPromise(() => {})
    if (promise1.status === STATUS.FULFILLED) {
    } else if (promise1.status === STATUS.REJECTED) {
    } else {
      onFulfilled = isFunction(onFulfilled) 
        ? onFulfilled 
        : (value) => { return value }
      onRejected = isFunction(onRejected) 
        ? onRejected 
        : (err) => { throw err }
      promise1.fulfilledCallbacks.push((value) => {})
      promise1.rejectedCallbacks.push((reason) => {})
    }
    return promise2
  }
}
```
````

::right::

<div v-click="5">

````md magic-move
```js
const isFunction = function(func) {
  return Object.prototype.toString.call(func)
    .toLocaleLowerCase() === '[object function]'
}
```

```js{8,13}
const isFunction = function(func) {
  return Object.prototype.toString.call(func)
    .toLocaleLowerCase() === '[object function]'
}

function fulfillPromise(promise, value) {
  // original code
  runCbs(promise.fulfilledCallbacks, value)
}

function rejectPromise(promise, reason) {
  // original code
  runCbs(promise.rejectedCallbacks, reason)
}
```

```js{6-11}
const isFunction = function(func) {
  return Object.prototype.toString.call(func)
    .toLocaleLowerCase() === '[object function]'
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
```
````

</div>
