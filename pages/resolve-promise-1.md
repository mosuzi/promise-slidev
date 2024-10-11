# resolve promise

```js
const resolvePromise = function(promise, x) {}
```

<br>

<v-clicks depth="2">

1. 如果 promise 和 x 是同一个值，则以一个 `TypeError` 拒绝 promise
2. 如果 x 是另一个 promise，则使用 x 的状态决议 promise
3. 如果 x 是一个对象或者函数，则尝试读取 `x.then`
   - 若读到的不是方法则以 x 为值满足 promise
   - 若是则以 x 为 this 调用它，传入与 promise 的 `then` 方法相似的两个回调函数，只不过这两个回调函数会调用 `resolvePromise` 和 `rejectPromise` 来处理 promise 的状态
   - 调用 `x.then` 时如果多次调用 `resolvePromise` 或者 `rejectPromise` 则应该只执行第一次调用而忽略后续调用
4. 如果 x 不是对象也不是函数，则以 x 为值满足 promise
5. 决议中的任何异常均会导致 promise 被以该异常拒绝（前提是 promise 仍处于 pending 状态）

</v-clicks>
