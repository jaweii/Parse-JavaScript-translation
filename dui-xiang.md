# 对象

#### Parse.Object {#parseobject}

Parse的对象存储是建立在`Parse.Object`上，每一个`Parse.Object`包含了JSON格式的键值对。它的数据是无模式的，这意味着你无需提前为每一个`Parse.Object` 指定存在的键。你只需要设置任何你想要的键值对就行了，我们的后端都会存储它。

举例说明，假设正在跟中游戏的高分，一个`Parse.Object` 可以包含：

```
score: 1337, playerName: "Sean Plott", cheatMode: false
```

键名必须是字母和数字组成的string类型的，值可以是string、boolean、array和任何可以被JSON编码的类型。



