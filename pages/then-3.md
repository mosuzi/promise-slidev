# then

规范中 2.2.4 提到，必须在[执行上下文](https://es5.github.io/#x10.3)堆栈仅包含平台代码时调用 `onFulfilled` 和 `onRejected`

<div v-click="1">

**需要异步执行回调**

</div>

<div v-click="2">

````md magic-move
```js
setTimeout(() => { // 
  try {
    const x = onFulfilled(promise1.value) 
    resolvePromise(promise2, x)
  } catch(error) {
    rejectPromise(promise2, error)
  }
}, 0)
```

```js
setTimeout(() => { // 
  try {
    const x = onFulfilled(promise1.value) 
    resolvePromise(promise2, x)
  } catch(error) {
    rejectPromise(promise2, error)
  }
}, 0)
```

```js
setTimeout(() => { // 
  try {
    const x = onFulfilled(promise1.value) 
    resolvePromise(promise2, x)
  } catch(error) {
    rejectPromise(promise2, error)
  }
}, 0)
```

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

```js {1,4-5,10-11}
then(onFulfilled, onRejected) {
  // other code
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
  }
  return promise2
}
```

```js {1,8-13}
then(onFulfilled, onRejected) {
  // other code
  if (promise1.status === STATUS.FULFILLED) {
  } else if (promise1.status === STATUS.REJECTED) {
  } else {
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
````

</div>
