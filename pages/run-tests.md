# 测试

使用 [Promise tests](https://github.com/promises-aplus/promises-tests) 测试自定义的 Promise

````md magic-move
```shell
npm install promises-aplus-tests -g
```

```js
// promise.js 追加
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

```shell
promises-aplus-tests promise.js
```
````

<div v-click="3">

![all tests passing](https://pics.mosuzi.com/blog/promises-tests-all-passing.png)

</div>
