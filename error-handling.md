# 错误处理

Parse的大多数JavaScript函数会在回调中返回一个成功或失败的对象，类似于Backbone中的"options"对象。

`success`和`error`这两个回调中，每当操作完成且没有错误，就会调用`seccess`在`save`或`get`中，`success`通常会是一个Parse.Object；在`find`中，会是一个Parse.Object数组。当与云端交互结束后出现了任何错误，就会调用`error`，`error`可能是连接云端的问题，也可能是执行请求操作的问题。

我们来看一个例子，在下面代码中，我们尝试去拉取一个云端不存在的对象，云端将会返回一个`error`，我们要根据`error`信息做相应的处理：

```js
var query = new Parse.Query(Note);
query.get("aBcDeFgH", {
  success: function(results) {
    // 不会被调用
    alert("Everything went fine!");
  },
  error: function(model, error) {
    // error是一个包含了错误信息的Parse.Error实例
    if (error.code === Parse.Error.OBJECT_NOT_FOUND) {
      alert("Uh oh, we couldn't find the object!");
    }
  }
});
```

下面代码中的查询也会失败，因为设备没有连接到云端。我们需要增加一点代码来处理这个错误：

```js
var query = new Parse.Query(Note);
query.get("thisObjectIdDoesntExist", {
  success: function(results) {
    // 不会被调用
    alert("Everything went fine!");
  },
  error: function(model, error) {
    // error是一个包含了错误信息的Parse.Error实例
    if (error.code === Parse.Error.OBJECT_NOT_FOUND) {
      alert("Uh oh, we couldn't find the object!");
    } else if (error.code === Parse.Error.CONNECTION_FAILED) {
      alert("Uh oh, we couldn't even connect to the Parse Cloud!");
    }
  }
});
```

对于像`save`和`signUp`这样会影响特定Parse.Object的方法，`error`方法的第一个参数将会是Parse.Object本身，第二个参数将会是Parse.Error对象。这是为了和类Backbone框架兼容。

要查看Parse.Eroor错误码列表，请查看错误码章节。



