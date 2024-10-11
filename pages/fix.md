# 修正

<br>

````md magic-move
```js {*|11}
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
        fulfillPromise(this, value)
      },
      (reason) => {
        rejectPromise(this, reason)
      }
    )
  }
  // @other code
}
```

```js {11-12|*}
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
        // fulfillPromise(this, value)
        resolvePromise(this, value)
      },
      (reason) => {
        rejectPromise(this, reason)
      }
    )
  }
  // @other code
}
```
````
