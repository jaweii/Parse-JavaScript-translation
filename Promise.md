# Promises

除了回调，Parse JavaScript SDK的所有异步方法，都返回一个Promise。通过promise，你的代码会比在回调中嵌套代码清晰很多。

## 举例

假设我们想保存一个Parse.Object，这是一个异步的操作，回调式的写法是这样的：

```js
object.save({ key: value }, {
  success: function(object) {
    // the object was saved.
  },
  error: function(object, error) {
    // saving the object failed.
  }
});
```

而在Promise的链式写法中，是这样的：

```js
object.save({ key: value }).then(function(obj){
    //保存成功
}).catch(err=>{
    //保存失败
})


//译者补充两个写法
object.set('key',value)
object.save().then(obj=>{/* ... */}).catch(function(err){/*...*/})

//或
object.set('key',value).save(obj=>{/*...*/},err=>{/*...*/})
```

---

如果你尚不了解Promise用法，可以参考此教程[Promise对象 ](http://es6.ruanyifeng.com/#docs/promise)。

