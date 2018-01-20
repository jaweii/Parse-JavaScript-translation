# 性能

随着你的应用规模增长，你将需要确保应用在负载和使用规模增长的情况下运行良好，本章将介绍如何将你的应用性能最优化。当你使用Parse服务快速构建你的原型时，你不需要考虑性能，但当你开始设计你的应用时，你很有必要考虑性能。我们强烈建议你在发布你的应用前，确保其遵循了我们的建议。

通过以下几点，可以提高你的应用性能：

* 编写高效的查询
* 编写受约束的查询
* 使用客户端缓存
* 使用云代码
* 避免count查询
* 使用高效的搜索技术

请将以上牢记心中。

下面我们来详细了解每一点。

## 编写高效的查询

Parse对象被存储在数据库中，Parse查询将根据你编写的查询条件获取对象，为了避免每个查询都查看给定的class表的所有数据，可以为数据库使用索引，索引是指满足给定标准的项目排序列表，索引能有帮助，是因为索引允许数据库高效的查询，并在不需要查看所有数据的情况下返回匹配结果。索引的大小很小，所以可以在内存中使用，所以查找更快。

## 索引

在使用Parse服务时，你需要管理你的数据库并保持索引，如果你没有索引，每个查询将必须遍历表中每个数据才能返回匹配结果，反之，如果你有恰当的索引，遍历的数据将很少。

查询的约束的有效性排序如下：

* 等于
* 包含
* 小于、小于等于、大于、大于等于
* 匹配字符串开头
* 不等于
* 不包含
* 其他

来看看下面的查询，获取`GameScore`对象：

```js
var GameScore = Parse.Object.extend("GameScore");
var query = new Parse.Query(GameScore);
query.equalTo("score", 50);
query.containedIn("playerName",
    ["Jonathan Walsh", "Dario Wunsch", "Shawn Simon"]);
```

在`score`字段上创建索引，会比在`playerName`上创建索引创造更小的搜索空间。

当检查数据类型使，布尔值的熵非常低，并且没有很好的索引，使用下面的查询约束：

```js
query.equalTo("cheatMode", false);
```

`cheatMode`可能的值有`true`和`false`，如果在`cheatMode`上建立索引，用处并不大，因为要返回查询结果，可能要遍历50%的对象。

数据类型根据键值空间的预期熵排序：

* GeoPoints
* Array
* Pointer
* Date
* String
* Number
* Other

即时最好的索引策略也可能被不够好的查询打败。

## 高效的查询设计

编写高效的查询，意味着充分利用索引的好处。下面的查询约束，会使索引无效：

* 不等于
* 不包含

另外某些场景下，下面的查询，如果他们不能充分利用索引的好处，可能会造成查询相应变慢。

* 正则表达式
* 按某字段排序

_不等于_

假设，你正在记录一个游戏中的`GameScore`表的高分，你想获取除了某个玩家以外的所有玩家分数，你可以编写这样的查询：

```js
var GameScore = Parse.Object.extend("GameScore");
var query = new Parse.Query(GameScore);
query.notEqualTo("playerName", "Michael Yabuti");
query.find().then(function(results) {
  // Retrieved scores successfully
});
```

这个查询不能充分利用索引的好处，数据库必须对比`GameScore`表中的每一个对象，才能返回结果，随着class表的条目增长，查询时间会变长。你应该和那个字段的其他字段匹配，而不是查询是否缺少值，这样才可以让数据库使用索引，查询会更快。

比如说，用户表有一个`state`字段，它可能的值有SignedUp、Verified和invited，下面这个查找所有最少使用过一次应用的用户的查询，是较慢的方式：

```js
var query = new Parse.Query(Parse.User);
query.notEqualTo("state", "Invited");
```

如果是给查询设置"包含条件"的约束，查询就会较快：

```js
query.containedIn("state", ["SignedUp", "Verified"]);
```

有时候你可应该重写你的查询，我们回到上一个`GameScore`的例子，假设我们要查询比指定玩家最高分高的玩家，我们可以做些不同的，先拿到指定玩家的最高分，然后执行下面查询：

```js
var GameScore = Parse.Object.extend("GameScore");
var query = new Parse.Query(GameScore);
// 先拿到 Michael Yabuti 的highScore
query.greaterThan("score", highScore);
query.find().then(function(results) {
  // Retrieved scores successfully
});
```

查询的重写取决于你的使用场景，这有时可能意味着要重新设计数据模型。

_不包含_

类似于"不等于"，"不包含"查询约束不能使用索引，你应该尝试使用互补的“包含”查询约束。回到之前的用户例子，如果`state`字段还有一个“Blocked”值表示被封的用户，我们现在要查询活动的用户，下面这样是低效的：

```js
var query = new Parse.Query(Parse.User);
query.notContainedIn("state", ["Invited", "Blocked"]);
```

使用对应的“包含”查询约束是高效的：

```js
query.containedIn("state", ["SignedUp", "Verified"]);
```

这意味着根据你的架构设置，你要相应的重写查询，甚至重写架构。

_正则表达式_

处于性能考虑，应该尽量避免使用正则表达式，MongoDB对于字符串部分匹配的执行不是很高效。使用正则查询对性能的消耗是很高的，尤其是在表数据超过10万条后。你需要考虑在特定时间内限制多少这样的查询，你的应用才能正常运行。

正则表达式不支持索引，比如下面的查询，通过给定的`playerName`字段查找数据，字符串查询时不区分大小写的，并且不能被索引：

```js
query.matches("playerName", "Michael", “i”);
```

下面的查询区分大小写，会查询每个相应字段包含了给定字符串的对象，并且不能被索引：

```js
query.contains("playerName", "Michael");
```

上面两个查询都是低效的，事实上，`matches`和`contains`查询约束并没有包含在我们的查询指南中，我们也不推荐使用他们。根据你的使用场景，你应该切换到使用下面的查询约束，它可以使用索引：

```js
query.startsWith("playerName", "Michael");
```

上面查询会根据给定的字符串查找数据，因为使用了所有，所有查询会更快。

作为最佳时间，当你使用正则表达式约束时，你要确保查询中的其他约束能将结果控制在数百个以内，以保证查询效率。如果你必须使用`matches`或`contains`约束，那么尽可能使用大小写敏感和锚查询，比如：

```
query.matches("playerName", "^Michael");
```

大多数场景下都会使用到包含正则表达式的搜索，稍候会详细介绍更高效的查询方法。

## 编写受约束的查询

编写受约束的查询可以保证只有客户端需要的数据被返回，这在数据使用受限和网络连接不可信的环境下非常重要。在查询章节介绍了你可以添加到查询的约束类型，用以限制返回的数据。增加约束时，你应专注设计高效的查询。

你可以使用skip和limit来获取需要的数据，查询条数限制默认为100条：

```js
query.limit(10); // 限制条数10
```

如果你在GeoPoints上遇到问题，确保你指定了合理的半径：

```js
var query = new Parse.Query(PlaceObject);
query.withinMiles("location", userGeoPoint, 10.0);
query.find().then(function(placesObjects) {
  // Get a list of objects within 10 miles of a user's location
});
```

你可以使用select进一步的限制返回哪些字段：

```js
var GameScore = Parse.Object.extend("GameScore");
var query = new Parse.Query(GameScore);
query.select("score", "playerName");
query.find().then(function(results) {
  // each of results will only have the selected fields available.
});
```

## 客户端缓存

对于IOS和Android应用，你可以开启查询缓存，详情查看[IOS](http://docs.parseplatform.org/ios/guide/#caching-queries)和[Android](http://docs.parseplatform.org/android/guide/#caching-queries)指南。查询缓存可以提高你的应用性能，尤其是在你想显示客户端从Parse最后一次拉取到的数据的时候。

## 使用云代码

云代码可以让你在Parse服务上运行JavaScript代码，而不是在客户端运行。

你可以使用云代码减少客户端逻辑，使应用性能感知上更好。你还可以在云代码中创建对象操作的触发器，这在验证数据和净化数据时很有用。你也可以使用云代码修改相关的对象，或启动其他处理，比如发送推送通知。

我们已经看过了通过编写受约束的查询限制返回数据，你也可以使用[云函数](http://docs.parseplatform.org/cloudcode/guide/#cloud-functions)来为你的应用限制返回的数据数量。在下面的例子中，我们定义了一个云函数来获取一部电影的平均分：

```js
Parse.Cloud.define("averageStars", function(request, response) {
  var Review = Parse.Object.extend("Review");
  var query = new Parse.Query(Review);
  query.equalTo("movie", request.params.movie);
  query.find().then(function(results) {
    var sum = 0;
    for (var i = 0; i < results.length; ++i) {
      sum += results[i].get("stars");
    }
    response.success(sum / results.length);
  }, function(error) {
    response.error("movie lookup failed");
  });
});
```

你也可以在客户端执行一个`Review`表的查询，只返回`stars`字段，并在客户端计算结果，随着一部电影的reviews（重看次数）增加，你可以看到返回到客户端的数量也在增加。

在你考虑优化你的查询时，你会发现你必须修改这个查询，有时候甚至是你在你把应用提交到应用市场以后。不通过修改客户端来修改查询代码是可能的，如果你使用了云函数，你就是要重新设计你的架构，也可以在云代码中完成

上面的查询平均分的云函数例子，在客户端可以像这样调用它：

```js
Parse.Cloud.run("averageStars", { "movie": "The Matrix" }).then(function(ratings) {
  // ratings is 4.5
});
```

如果以后你需要修改数据架构，你的客户端可以保持不变，只要你返回一个表示得分结果的数字就行了。















