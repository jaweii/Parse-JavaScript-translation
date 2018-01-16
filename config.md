# 配置

`Parse.Config`是存储在Parse上的配置对象，是远程应用配置的最佳实践。它可以帮你实现全局设置、每日消息之类的功能。

要开始使用`Parse.Config`，你需要在Parse配置面板添加一些键值对到你的app，然后你就可以在客户端拉取到`Parse.Config`对象：

```js
Parse.Config.get().then(function(config) {
  var winningNumber = config.get("winningNumber");
  var message = "Yay! The number is " + winningNumber + "!";
  console.log(message);
}, function(error) {
  // Something went wrong (e.g. request timed out)
});
```

---

## 获取配置

`ParseConfig`即便在糟糕的网络下，也会尽可能的保持可用，默认配置会被缓存，以确保最后一次成功拉取的配置始终可用。在下面的例子中，我们从服务器拉取配置的最新版本，如果`get`失败，我们通过`current`回退到最近一次成功拉取的配置：

```js
Parse.Config.get().then(function(config) {
  console.log("Yay! Config was fetched from the server.");

  var welcomeMessage = config.get("welcomeMessage");
  console.log("Welcome Message = " + welcomeMessage);
}, function(error) {
  console.log("拉取失败，使用缓存配置");

  var config = Parse.Config.current();
  var welcomeMessage = config.get("welcomeMessage");
  if (welcomeMessage === undefined) {
    welcomeMessage = "Welcome!";
  }
  console.log("Welcome Message = " + welcomeMessage);
});
```

---

## 当前配置

每一个`Parse.Config`实例都是不可改变的，当你从云端获取到一个新的`Parse.Config`实例，这并不会修改已经存在的任何`Parse.Config`，但是会通过`Parse.Config.current()`新创建一个替换掉之前的，因此，你可以安全地传递任何`current()`对象，并且认定它不会自动改变。

如果每次要使用配置的时候都拉取一次，这可能过于麻烦，你可以通过`current()`的缓存来避免此麻烦，然后隔一段时间拉取一次就可以了。

```js
// 应用每过12小时重新更新配置
var refreshConfig = function() {
  var lastFetchedDate;
  var configRefreshInterval = 12 * 60 * 60 * 1000;
  return function() {
    var currentDate = new Date();
    if (lastFetchedDate === undefined ||
        currentDate.getTime() - lastFetchedDate.getTime() > configRefreshInterval) {
      Parse.Config.get();
      lastFetchedDate = currentDate;
    }
  };
}();
```

---

## 参数

`ParseConfig`支持`Parse.Object`支持的绝大多数数据类型：

string

* number
* Date
* Parse.File
* Parse.GeoPoint
* JS Array
* JS Object

目前配置支持最多100个参数，并且支持的所有参数总大小为128Kb。

