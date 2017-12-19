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

但是，当你使用extends时，SDK将不会自动识别你的子类。如果你想要请求返回的对象使用你的Parse.Object子类，你将需要注册这个子类，就像我们在其他平台做的一样：

```js
// 指定Monster子类以后
Parse.Object.registerSubclass('Monster', Monster);
```

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



