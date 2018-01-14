# 对象关系

对象关系有三种。一对一关系，即一个对象关联另一个对象的关系；一对多关系，即多个对象关联一个对象的关系；多对多关系，即多个对象中复杂的关系。

有4种方法在Parse中建立对象关系：

#### 一对多关系 {#一对多}

如果你想使用指针或数组来实现一对多的关系，有几个因素需要考虑。首先要考虑，这种关系中涉及多少个对象，如果子对象数量多余100个，推荐使用一对多的关系，如果少于100个，使用数组可能更方便，尤其是你想根据父对象一次获取所有子对象的时候。

_使用指针_

假设我们有一个游戏应用，要跟踪玩家的分数和成就，在Parse中，我们可以把这些数据保存在Game对象中。如果这个游戏非常成功，所有玩家可能会存储几千个Game对象到系统中，对于这样的情况，子对象的数量可能任意大，指针就是最合适的选择。

要实现这样的游戏应用，我们要确保每个Game对象都关联一个用户对象，我们可以这样实现：

```js
var game = new Parse.Object("Game");
game.set("createdBy", Parse.User.current());
```

我们可以通过匹配用户对象来查询用户的所有Game对象：

```js
var query = new Parse.Query("Game");
query.equalTo("createdBy", Parse.User.current());
```

如果要获取Game对象的创建者，可以通过指针字段获取：

```js
// Game对象
var game = ...

// Game对象创建者
var user = game.get("createdBy");
```

大多数情况下，实现一对多的关系，使用指针都是最好的选择。

_使用数组_

当我们已知子对象的数量很少时，使用数组是个好办法。`includeKey`参数提供一些便捷性，可以让你在获得父对象的同时获得所有的子对象。不过，如果子对象数量很大的话，会造成响应时间增长。

假设在我们的游戏中，要跟踪玩家在游戏中积累的武器，并且要限制武器的数量。我们已知武器的数量不大，并且需把用户的武器按顺序显示在屏幕中。在这个例子中，使用数组就是很好的方法。

现在让我们来为用户对象创建一个`weaponsList`\(武器列表\)字段，然后存储一些`Weapon`\(武器\)到`weaponsList`中：

```js
// 假设我们有4个武器对象
var scimitar = ...
var plasmaRifle = ...
var grenade = ...
var bunnyRabbit = ...

//插入武器到对象
var weapons = [scimitar, plasmaRifle, grenade, bunnyRabbit];

// 为用户设置weaponsList
var user = Parse.User.current();
user.set("weaponsList", weapons);
```

如果以后想再获得`Weapon`对象，只需要一行代码即可：

```js
var weapons = Parse.User.current().get("weaponsList")
```

有时我们fetch一个对象，想要同时拉取父对象的子对象，只需要为`Parse.Query`添加`includeKey`即可：

```js
// 用户对象的查询对象
var userQuery = new Parse.Query(Parse.User);

// configure any constraints on your query...
// for example, you may want users who are also playing with or against you

// 告诉查询对象想要同时拉取到哪个子对象
userQuery.include("weaponsList");

// 执行查询
userQuery.find({
  success: function(results){
    // results contains all of the User objects, and their associated Weapon objects, too
  }
});
```

你也可以通过子对象获取父对象，比如，我们要查询哪个用户具有给定的子对象，我们可以为查询对象增加一个这样的条件：

```
// 增加武器列表是否包含某武器的条件
userQuery.equalTo("weaponsList", scimitar);

// 或者传入武器数组查找包含多个武器的用户
userQuery.containedIn("weaponsList", arrayOfWeapons);
```

#### 多对多关系



