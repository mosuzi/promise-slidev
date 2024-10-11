# 完善

<br>

````md magic-move
```js
const isFunction = function (func) {
  return Object.prototype.toString.call(func).toLocaleLowerCase() === '[object function]'
}

const isObject = function(obj) {
  return Object.prototype.toString.call(obj).toLocaleLowerCase() === '[object object]'
}
```

```js
const isType = function (type) {
  return function (obj) {
    return Object.prototype.toString.call(obj).toLocaleLowerCase() === '[object ' + type + ']'
  }
}

const isObject = isType('object')

const isFunction = isType('function')

```
````
