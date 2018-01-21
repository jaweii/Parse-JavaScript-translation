# 对象关系

对象关系有三种。一对一关系，即一个对象关联另一个对象的关系；一对多关系，即多个对象关联一个对象的关系；多对多关系，即多个对象中复杂的关系。

有4种方法在Parse中建立对象关系：

## 一对多关系

如果你想使用指针或数组来实现一对多的关系，有几个因素需要考虑。首先要考虑，这种关系中涉及多少个对象，如果子对象数量多余100个，推荐使用一对多的关系，如果少于100个，使用数组可能更方便，尤其是你想根据父对象一次获取所有子对象的时候。

### _使用指针_

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

### _使用数组_

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

```js
// 增加武器列表是否包含某武器的条件
userQuery.equalTo("weaponsList", scimitar);

// 或者传入武器数组查找包含多个武器的用户
userQuery.containedIn("weaponsList", arrayOfWeapons);
```

---

## 多对多关系

现在我们来处理多对多关系。假设我们有一个阅读应用，我们需要构建`Book`对象和`Author`对象，一个作者可以写多本书，并且一本书可以有多个作者，这就是一个多对多关系的场景，你必须从数组、对象关系、指针三个方案中做出选择。

决策的关键点在于你是否有父子对象以外的信息要保存。

如果没有，使用关系对象或者使用数组是最简单的方案。通常，使用数组会产生更高的并发和较少的请求。

如果有，那么使用指针指向关联对象更合适。

### _使用Parse.Realtion_

使用Parse.Relation，我们可以在`Book`对象和一些`Author`对象中建立关联。你可以在数据浏览器中，在`Book`对象上创建一个relation类型的`authors`字段。然后，我们可以为Book对象关联一些author：

```js
// 假设有这么几个author对象
var authorOne = ...
var authorTwo = ...
var authorThree = ...

// 构建book对象
var book = new Parse.Object("Book");

// 为book对象关联作者
var relation = book.relation("authors");
relation.add(authorOne);
relation.add(authorTwo);
relation.add(authorThree);

// 保存
book.save();
```

要获取一本书的作者列表，可以创建一个查询：

```js
// 假设有一个book对象
var book = ...

// 通过relation方法获取relation对象
var relation = book.relation("authors");

// 通过relation对象构建查询对象
var query = relation.query();

// 执行查询即可
```

假如你想获取一个作者参与编辑过的书籍列表， 只需要为查询对象增加条件即可：

```js
// 假设有一个作者对象
var author = ...

// 构建查询对象
var query = new Parse.Query("Book");

// 增加约束条件
query.equalTo("authors", author);
```

### _使用指针_

假设我们要在用户间构建一个following\(我关注的\)/follower\(关注我的\)关系，一个用户可以关注多个用户，就像社交平台一样。在我们的应用中，我们不只需要知道谁关注了谁，还需要知道是什么时候关注的。要有序的记录这些数据，你必须创建一个独立的class表，这个表我们将命名为`Follow`，它有`form`字段和`to`字段，都是指向一个用户的指针。除了这些，你还可以添加一个`Date`类型的字段`date`用来记录时间。

现在，如果你想保存两个用户的关注信息，只需要在`Follow`表中设置`from`、`to`和`date`的值即可：

```js
var otherUser = ...

// 构建Follow对象
var follow = new Parse.Object("Follow");
follow.set("from", Parse.User.current());
follow.set("to", otherUser);
follow.set("date", Date());
follow.save();
```

如果你想查找你关注的所有用户，可以创建一个`Follow`的查询对象：

```js
var query = new Parse.Query("Follow");
query.equalTo("from", Parse.User.current());
query.find({
  success: function(users){
    ...
  }
});
```

要查找关注我的所有用户也同样简单，通过`to`字段查找即可：

```js
// 构建查询对象
var query = new Parse.Query("Follow");
query.equalTo("to", Parse.User.current());
query.find({
  success: function(users){
    ...
  }
});
```

### _使用数组_

在多对多关系中使用数组和在一对多关系中使用数组很像，关系一方的对象用一个数组字段包含另一方的一些对象。

还是假设我们有一个阅读应有，有`Book`和`Author`两个对象，`Book`对象有一个authors数组字段包含了`Book`的所有`Author`，数组在这里就非常合适，因为一本书的作者不会超过100个人。不过一个作者可能会写100本以上的书。

下面是我们保存`Book`和`Author`关系的示例：

```js
// 假设有一个作者对象
var author = ...

// 假设有个Book对象
var book = ...

// 把author对象添加到authors数组中
book.add("authors", author);
```

因为`authors`是一个数组，你应该在查询`Book`的时使用`includeKey`参数把`author`包含进来，以便服务器返回`Book`对象时同时包含作者：

```js
// 构建Book查询对象
var bookQuery = new Parse.Query("Book");

// 告诉查询对象包含authors字段
bookQuery.include("authors");

// execute the query
bookQuery.find({
  success: function(books){
    ...
  }
});
```

要根据拿到的`Book`对象获取`authors`数组可以直接使用`get`方法：

```js
var authorList = book.get("authors")
```

最后，如果你想根据一个用户对象，查询用户所有参与编辑过的书，直接使用`include`方法即可：

```js
// 构建查询对象
var bookQuery = new Parse.Query("Book");

// 增加约束条件
bookQuery.equalTo("authors", author);

// 告诉查询对象要包含作者
bookQuery.include("authors");

// 执行查询
bookQuery.find({
  success: function(books){
    ...
  }
});
```

---

## 一对一关系

在某些情况下，要将一个对象分成两个对象，一对一关系就很合适。这种例子可能很少，但是也有如下两个例子：

* **限制某些用户数据的可见**。在这个场景中，你要把对象一分为二，其中一部分数据对其他用户可见，关联的另一部分数据对其他用户不可见，并且被ACL保护。
* **分割对象大小**。在这个场景中，你的原始对象大小超过了允许的128K，所以你决定创建第二个对象存储额外的数据。所以设计好你的数据模型很重要，可以避免对象过大要分割对象。如果你无法避免这样做，你可以考虑将大文件存储到Parse.File中。

最后，感谢你阅读了这么多，我们为这复杂程度向你致歉。虽然对象关系是比较难的部分，但是也有好的一面：

它可比人际关系简单多了。

