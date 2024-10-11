# then

```js
promise2 = promise.then(onFulfilled, onRejected)
```

<br>

<v-clicks>

1. 其中的 `onFulfilled` 和 `onRejected` 都是可选参数，并且当它们各自不为函数时，会被忽略。它们各自只能在 promise 状态发生相应变化后被调用，且只能被调用一次
2. `then` 方法可以被调用多次。当 promise 被满足时，按照原始调用顺序依次执行回调 `onFulfilled`，当 promise 被拒绝时，按照原始调用顺序依次执行回调 `onRejected`
3. `then` 方法必须返回一个 promise

</v-clicks>

