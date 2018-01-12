# 配置

Parse.Config是存储在Parse上的配置对象，是远程应用配置的最佳实践。它可以帮你实现每日消息之类的功能。

要开始使用Parse.Config，你需要**在**Parse配置面板添加一些键值对到你的app，然后你就可以在客户端拉取到Parse.Config对象：

```
Parse.Config.get().then(function(config) {
  var winningNumber = config.get("winningNumber");
  var message = "Yay! The number is " + winningNumber + "!";
  console.log(message);
}, function(error) {
  // Something went wrong (e.g. request timed out)
});
```

---

#### 获取配置



