# 对象

#### Parse.Object {#parseobject}

Parse的对象存储是建立在`Parse.Object`上，每一个`Parse.Object`包含了JSON格式的键值对。它的数据是无模式的，这意味着你无需提前为每一个`Parse.Object` 指定存在的键。你只需要设置任何你想要的键值对就行了，我们的后端都会存储它。

举例说明，假设你正在跟踪游戏的高分，一个`Parse.Object` 可以包含：

```
score: 1337, playerName: "Sean Plott", cheatMode: false
```

键名必须是字母和数字组成的string类型的，值可以是string、boolean、array和任何可以被JSON编码的类型。

每一个`Parse.Object` 都是一个特定子类的实例，且具备类名。

你可以通过不同的实例来区分不同的数据。为了让你的代码更加诗意，我们推荐你使用驼峰命名，就像gameScore或者GameScore。

---

要创建一个新的子类，使用`Parse.Object.extend` 方法即可。每个`Parse.Query`都会返回一个和`Parse.Object`具有相同类名的新子类。如果你熟悉`Backbone.Model`，那么你已经知道如何去使用`Parse.Object`了，它们创建和修改的设计是一样的。

```js
// 创建一个新的Parse.Object子类
var GameScore = Parse.Object.extend("GameScore");

// 创建一个新的子类实例
var gameScore = new GameScore();

// 你也可以使用backbone的语法直接创建.
var Achievement = Parse.Object.extend({
  className: "Achievement"
});
```

你可以添加额外的方法和属性到你的`Parse.Object`子类：

```js
// 一个复杂的Parse.Object子类
var Monster = Parse.Object.extend("Monster", {
  // 实例方法
  hasSuperHumanStrength: function () {
    return this.get("strength") > 18;
  },
  // 实例属性
  initialize: function (attrs, options) {
    this.sound = "Rawr"
  }
}, {
  // 类方法
  spawn: function(strength) {
    var monster = new Monster();
    monster.set("strength", strength);
    return monster;
  }
});

var monster = Monster.spawn(200);
alert(monster.get('strength'));  // Displays 200.
alert(monster.sound); // Displays Rawr.
```

如果想为任意一个对象创建一个实例，你也可以使用`Parse.Object`直接构造。`new Parse.Object(className)`将会创建一个指定类名的实例。

如果你的项目已经使用了ES6，恭喜！从1.6.0版本开始，JavaScript SDK 已经和ES6类兼容，你可以使用`extends`关键词继承`Parse.Object`：

```js
class Monster extends Parse.Object {
  constructor() {
    // Pass the ClassName to the Parse.Object constructor
    super('Monster');
    // All other initialization
    this.sound = 'Rawr';
  }

  hasSuperHumanStrength() {
    return this.get('strength') > 18;
  }

  static spawn(strength) {
    var monster = new Monster();
    monster.set('strength', strength);
    return monster;
  }
}
```

但是，当你使用extends时，SDK将不会自动识别你的子类。如果你想要查询返回的对象使用你的Parse.Object子类，你将需要注册这个子类，就像我们在其他平台做的一样：

```js
// 指定Monster子类以后
Parse.Object.registerSubclass('Monster', Monster);
```

---

#### 保存对象 {#保存对象}

假设你想保存之前描述的对象`GameScore`到Parse服务器，这个接口和`BackBone.Model`类似，即`save`方法：

```js
var GameScore = Parse.Object.extend("GameScore");
var gameScore = new GameScore();

gameScore.set("score", 1337);
gameScore.set("playerName", "Sean Plott");
gameScore.set("cheatMode", false);

gameScore.save(null, {
  success: function(gameScore) {
    // 保存成功
    alert('New object created with objectId: ' + gameScore.id);
  },
  error: function(gameScore, error) {
    // 包含了错误码和错误信息
    alert('Failed to create new object, with error code: ' + error.message);
  }
});
```

在上面代码运行后，你可能会想确认是否真的保存成功了，你可以在Parse的数据浏览器中查看，你将会看到类似下面的数据：

```js
objectId: "xWMyZ4YEGZ", score: 1337, playerName: "Sean Plott", cheatMode: false,
createdAt:"2011-06-10T18:33:42Z", updatedAt:"2011-06-10T18:33:42Z"
```

---

#### 取得对象 {#取得对象}

比保存数据到云端更有趣的是，从云端再次取回数据。如果你已经有了`objectId`，你可以通过`Parse.Query`来获得完整的`Parse.Object`。

```js

var GameScore = Parse.Object.extend("GameScore");
var query = new Parse.Query(GameScore);
query.get("xWMyZ4YEGZ", {
  success: function(gameScore) {
    // The object was retrieved successfully.
  },
  error: function(object, error) {
    // The object was not retrieved successfully.
    // error is a Parse.Error with an error code and message.
  }
});
```

通过`get`方法，即可从`Parse.Object`中获得值：

```js
var score = gameScore.get("score");
var playerName = gameScore.get("playerName");
var cheatMode = gameScore.get("cheatMode");
```

有三个值的键名是预留的属性，无法通过`get`方法获取，也不能通过`set`方法修改：

```js
var objectId = gameScore.id;
var updatedAt = gameScore.updatedAt;
var createdAt = gameScore.createdAt;
```

如果你需要更新一个已有的对象，你可以调用`fetch`方法，获取云端的最新数据：

```js
myObject.fetch({
  success: function(myObject) {
    // The object was refreshed successfully.
  },
  error: function(myObject, error) {
    // The object was not refreshed successfully.
    // error is a Parse.Error with an error code and message.
  }
});
```

---

#### 更新对象

更新对象非常简单，只需要设置一些新数据，然后调用`save`方法就可以了：

```js
// Create the object.
var GameScore = Parse.Object.extend("GameScore");
var gameScore = new GameScore();

gameScore.set("score", 1337);
gameScore.set("playerName", "Sean Plott");
gameScore.set("cheatMode", false);
gameScore.set("skills", ["pwnage", "flying"]);

gameScore.save(null, {
  success: function(gameScore) {
    // Now let's update it with some new data. In this case, only cheatMode and score
    // will get sent to the cloud. playerName hasn't changed.
    gameScore.set("cheatMode", true);
    gameScore.set("score", 1338);
    gameScore.save();
  }
});
```

Parse会自动判断哪些数据是修改过的，所有只有”脏”字段会被发送到云端。

##### 计数器

上述的例子中包含了常用的使用情况，但score字段是一个计数器，我们需要不断的更新玩家最后的分数，上述的方法虽然也可用，但是过于麻烦，而且如果你有多个客户端同时更新，可能回导致一些问题。

为了存储计数器类型的数据，Parse提供了原子级增减的方法，`increment`和`decrement` 。所以，计数器的更新我们可以这样写：

```js
gameScore.increment("score");
gameScore.save();
```

 你可以给`increment`和`decrement`传入第二个参数，作为要增减的数值，如果没有传入，默认是1。

#####  数组

为了存储数组型数据，我们提供了三个方法，可以原子级操作给定键值的数组：

* add 增加一个值到对象的数组字段的尾部。
* addUnique 如果指定数组字段中不包含这个值，才插入这个值到数组中。不保证插入的位置。













