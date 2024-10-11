---
layout: two-cols-title
---
::title::

# 构造 Promise

::left::

````md magic-move
```js
class MyPromise {
  constructor() {}
  then() {}
}
```

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

```js
class MyPromise {
  constructor(fn) {
    this.status = STATUS.PENDING
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
````

::right::

<div v-click="3">

````md magic-move
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
````

</div>
