# 对象

## Parse.Object

Parse的对象存储是建立在`Parse.Object`上，每一个`Parse.Object`包含了JSON格式的键值对，它的数据是无模式的，这意味着你无需提前为每一个`Parse.Object` 指定存在的键，你只需要设置任何你想要的键值对就行了，后端会自动创建并存储它。

举例说明，假设你正在记录游戏玩家的分数，一个`Parse.Object` 可以包含：

```
score: 1337, playerName: "Sean Plott", cheatMode: false
```

键名必须是字母和数字组成的string类型的，键值可以是string、boolean、array和任何可以被JSON编码的类型。

每一个`Parse.Object` 都是一个使用表名构造的表记录对象。你可以通过不同的实例来区分不同的数据。为了让你的代码更加诗意，我们推荐你使用驼峰命名，就像gameScore或者GameScore。

---

要创建一个新的子类，使用`Parse.Object.extend` 方法即可。如果你熟悉`Backbone.Model`，那么你已经知道如何去使用`Parse.Object`了，它们创建和修改的设计是一样的。

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

## 保存对象

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

## 取得对象

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

## 更新对象

更新对象超级简单，只需要设置一些新数据，然后调用`save`方法就可以了：

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

### 计数器

上述的例子中包含了常用的使用情况，但score字段是一个计数器，我们需要不断的更新玩家最后的分数，上述的方法虽然也可用，但是过于麻烦，而且如果你有多个客户端同时更新，可能回导致一些问题。

为了存储计数器类型的数据，Parse提供了原子级增减的方法，`increment`和`decrement` 。所以，计数器的更新我们可以这样写：

```js
gameScore.increment("score");
gameScore.save();
```

你可以给`increment`和`decrement`传入第二个参数，作为要增减的数值，如果没有传入，默认是1。

### 数组

为了存储数组型数据，我们提供了三个方法，可以原子级操作给定键值的数组：

* `add`增加一个元素到对象的数组字段的尾部。
* `addUnique`如果指定数组字段中不包含这个元素，才插入这个元素到数组中。不保证插入的位置。
* `remove`从给定对象的数组字段中移除所有元素。

举例说明，比如我们要添加一些元素到“skills”字段中，可以这样：

```js
gameScore.addUnique("skills", "flying");
gameScore.addUnique("skills", "kungfu");
gameScore.save();
```

注意，目前不支持在同一次`save`中，原子级`add`或`remove`数组字段中的元素，你必须在每次不同的数组操作后调用`save`。

---

## 删除对象

从云端删除一个对象：

```js
myObject.destroy({
  success: function(myObject) {
    // The object was deleted from the Parse Cloud.
  },
  error: function(myObject, error) {
    // The delete failed.
    // error is a Parse.Error with an error code and message.
  }
});
```

你可以用`unset`方法从对象中删除一个字段：

```js
// 删除playerName字段
myObject.unset("playerName");

// Saves the field deletion to the Parse Cloud.
// If the object's field is an array, call save() after every unset() operation.
myObject.save();
```

请注意，不推荐使用object.set\(null\)的方式从对象中删除字段，这可能会造成意外的问题。

---

## 关系型数据

一个对象可能和其他对象存在关联，比如，一个博客应用中，一篇文章\(Post\)的对象，可能关联着许多评论\(Comment\)对象。Parse支持所有的关系类型，包括一对一，一对多，多对多。

### 一对一和一对多

一对一和一对多关系，是通过将`Parse.Object`的值设为其他对象来模型化的，比如，每个`Comment`对象，都对应了一个`Post`对象。

要创建一篇新文章和一条评论，你可以这样写：

```js
// Declare the types.
var Post = Parse.Object.extend("Post");
var Comment = Parse.Object.extend("Comment");

// 创建Post
var myPost = new Post();
myPost.set("title", "I'm Hungry");
myPost.set("content", "Where should we go for lunch?");

// 创建Comment
var myComment = new Comment();
myComment.set("content", "Let's do Sushirrito.");

// 设置关系
myComment.set("parent", myPost);

// 保存
myComment.save();
```

在内部，Parse框架只会把引用对象存储在一处，以保证一致性。你也可以通过objectId来指向关联对象：

```js
var post = new Post();
post.id = "1zEcyElZ80";

myComment.set("parent", post);
```

默认情况下，在拉取一个对象时，是不会将关联的对象一起拉取的，关联的对象需要再次拉取：

```js
var post = fetchedComment.get("parent");
post.fetch({
  success: function(post) {
    var title = post.get("title");
  }
});
```

### 多对多关系

多对多关系是通过`Parse.Relation`模型化的。这和存储一个对象到数组字段中类似，不同之处在于，你不需要一次拉取这个关系中的所有对象。另外这允许你拓展多于数组字段的对象。

假如，一个`User`可能有许多她喜欢的的`Posts`，那么你可以把`User`喜欢的`Posts`作为`relation`，要添加一篇`Post`到她喜欢的列表中，你可以这么做：

```js
var user = Parse.User.current();
var relation = user.relation("likes");
relation.add(post);
user.save();
```

你可以这样从Parse.Relation中删除一篇文章：

```js
relation.remove(post);
user.save();
```

你可以多次使用`add`和`remove`再调用`save`：

```js
relation.remove(post1);
relation.remove(post2);
user.save();
```

你也可以通过传入一个`Parse.Object`数组来`add`或者`remove`：

```js
relation.add([post1, post2, post3]);
user.save();
```

默认情况下，关联的对象列表是没有拉取到本地，你可以使用query返回的Parse.Query得到用户喜欢的文字列表：

```js
relation.query().find({
  success: function(list) {
    // list contains the posts that the current user likes.
  }
});
```

如果你只想要`Posts`的部分子集，你可以添加额外的条件到`query`：

```js
var query = relation.query();
query.equalTo("title", "I'm Hungry");
query.find({
  success:function(list) {
    // list contains post liked by the current user which have the title "I'm Hungry".
  }
});
```

更多关于`Parse.Query`的详情，请查看指南的查询\(Queries\)章节。`Parse.Relation`的查询和`Parse.Object`的查询类似，所以`Parse.Object`的任何查询，都可以用在`Parse.Relation`上。

---

## 数据类型

目前为止，我们使用的键值的类型有String、Number、和Parse.Object。Parse也支持Date和null。你还可以存储JSON Object 和JSON Array到你的Parse.Object。下面列出的，是对象的字段支持的所有类型：

* String =&gt;`String`
* Number =&gt;`Number`
* Bool =&gt;`bool`
* Array =&gt;`JSON Array`
* Object =&gt;`JSON Object`
* Date =&gt;`Date`
* File =&gt;`Parse.File`
* Pointer =&gt; other`Parse.Object`
* Relation =&gt;`Parse.Relation`
* Null =&gt;`null`

一些例子：

```js
var number = 42;
var bool = false;
var string = "the number is " + number;
var date = new Date();
var array = [string, number];
var object = { number: number, string: string };
var pointer = MyClassName.createWithoutData(objectId);

var BigObject = Parse.Object.extend("BigObject");
var bigObject = new BigObject();
bigObject.set("myNumber", number);
bigObject.set("myBool", bool);
bigObject.set("myString", string);
bigObject.set("myDate", date);
bigObject.set("myArray", array);
bigObject.set("myObject", object);
bigObject.set("anyKey", null); // this value can only be saved to an existing key
bigObject.set("myPointerKey", pointer); // shows up as Pointer <MyClassName> in the Data Browser
bigObject.save();
```

我们不推荐在对象中存储大件的二进制文件，比如图片和文档。对象应该在128字节以内。

我们推荐你使用Parse.File来存储文件、图片，和其他文件 。你可以初始化一个Parse.File对象设置你的文件。具体可以看文件\(Files\)章节。

关于Parse处理数据的更多信息，请查看文档的数据\(Data\)章节。

## 



