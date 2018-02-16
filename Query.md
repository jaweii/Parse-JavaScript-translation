# 查询

我们已经知道了`Parse.Query`是如何使用`get`从Parse云端查询一个对象的，下面还有许多其他方法，比如你可以一次查询多个对象，可以在一个查询上限制条件等。

## 基本查询

很多时候，`get`可能满足不了你的欲望。

所以，`Parse.Query`提供了其他的方法，可以让你查询一批对象，而不是一个对象。

最常用的方法是始创建一个`Parse.Query`，为其设置查询条件，然后使用`find`方法查询匹配的对象列表。比如，要查询一个玩家的`playerName`的得分，我们使用`equalTo`方法来设定“等于”查询条件：

```js
var GameScore = Parse.Object.extend("GameScore");
var query = new Parse.Query(GameScore);
query.equalTo("playerName", "Dan Stemkoski");
query.find({
  success: function(results) {
    alert("Successfully retrieved " + results.length + " scores.");
    // Do something with the returned Parse.Object values
    for (var i = 0; i < results.length; i++) {
      var object = results[i];
      alert(object.id + ' - ' + object.get('playerName'));
    }
  },
  error: function(error) {
    alert("Error: " + error.code + " " + error.message);
  }
});
```

---

## 查询条件

下面有几个方法可以约束`Parse.Query`的查询结果，比如你可以用`notEqualTo`方法，传入键值对来筛选结果：

```js
query.notEqualTo("playerName", "Michael Yabuti");
```

你可以给定多个约束条件，只有条件匹配的对象会被作为查询结果返回。在口语中，这和“且”的条件限制相似：

```js
query.notEqualTo("playerName", "Michael Yabuti");
query.greaterThan("playerAge", 18);
```

你可以限定查询结果的返回数量，默认情况下，返回数量是100：

```js
query.limit(10); // limit to at most 10 results
```

如果你只想查询一个结果，可以用更方便的`first`方法替代`find`方法：

```js
var GameScore = Parse.Object.extend("GameScore");
var query = new Parse.Query(GameScore);
query.equalTo("playerEmail", "dstemkoski@example.com");
query.first({
  success: function(object) {
    // Successfully retrieved the object.
  },
  error: function(error) {
    alert("Error: " + error.code + " " + error.message);
  }
});
```

你可以通过设置skip来跳过x个查询结果，这在分页功能中很有：

```js
query.skip(10); // skip the first 10 results
```

对于可排序的字段类型，如string、number、date，你可以指定字段类型的排序，控制返回结果的排序：

```js
// Sorts the results in ascending order by the score field
query.ascending("score");

// Sorts the results in descending order by the score field
query.descending("score");
```

对于可排序类型，你还可以在查询中使用比较：

```js
//wins < 50
query.lessThan("wins", 50);

//  wins <= 50
query.lessThanOrEqualTo("wins", 50);

// wins > 50
query.greaterThan("wins", 50);

// wins >= 50
query.greaterThanOrEqualTo("wins", 50);
```

如果你想查询与数组中任意值匹配的对象，可以使用`containedIn`方法，并提供一个可接受值的数组。这样可以一个请求完成多个请求的功能。比如，你想查询数组中玩家的分数结果：

```js
// Finds scores from any of Jonathan, Dario, or Shawn
query.containedIn("playerName",
                  ["Jonathan Walsh", "Dario Wunsch", "Shawn Simon"]);
```

如果你想查询和数组中任意值都不匹配的对象，你可以使用`notContainedIn`，并给定一个排除值的数组。比如，你想查询不在数组中的玩家分数：

```js
// Finds scores from anyone who is neither Jonathan, Dario, nor Shawn
query.notContainedIn("playerName",
                     ["Jonathan Walsh", "Dario Wunsch", "Shawn Simon"]);
```

如果你想查询指定字段已被设置的对象，你可以使用`exists`；反之，如果想查询未被设置的对象，你可以使用`doesNotExist`：

```js
// Finds objects that have the score set
query.exists("score");

// Finds objects that don't have the score set
query.doesNotExist("score");
```

你可以使用`meachesKeyInQuery`方法查询对象，对象中的某个键值是和另一个请求返回的一组结果中的键值匹配的。比如，如果你有一个包含了运动队的类，并且你保存了用户的老家信息在user类中，你现在要查询获胜队伍的成员家乡信息，你可以这样请求：

```js
var Team = Parse.Object.extend("Team");
var teamQuery = new Parse.Query(Team);
teamQuery.greaterThan("winPct", 0.5);
var userQuery = new Parse.Query(Parse.User);
userQuery.matchesKeyInQuery("hometown", "city", teamQuery);
userQuery.find({
  success: function(results) {
    // results has the list of users with a hometown team with a winning record
  }
});
```

反之，如果你要查询，对象中的某个键值是和另一个请求返回的一组结果中的键值是不匹配的，用`doesNotMatchKeyInQuery`。比如，查询战败队成员的家乡信息：

```js
var losingUserQuery = new Parse.Query(Parse.User);
losingUserQuery.doesNotMatchKeyInQuery("hometown", "city", teamQuery);
losingUserQuery.find({
  success: function(results) {
    // results has the list of users with a hometown team with a losing record
  }
});
```

你可以使用`select`限定返回的字段。如果你想查询只包含了`score`和`playerName`字段的数据记录\(还有指定的内置字段objectId、createdAt和updatedAt\)：

```
var GameScore = Parse.Object.extend("GameScore");
var query = new Parse.Query(GameScore);
query.select("score", "playerName");
query.find().then(function(results) {
  // each of results will only have the selected fields available.
});
```

剩下的字段你可以使用`fetch`方法，在需要的时候拉取：

```
query.first().then(function(result) {
  // only the selected fields of the object will now be available here.
  return result.fetch();
}).then(function(result) {
  // all fields of the object will now be available here.
});
```

### 比较查询 {#hash858517260}

| 逻辑操作 | Query 方法 |
| :--- | :--- |
| 等于 | `equalTo` |
| 不等于 | `notEqualTo` |
| 大于 | `greaterThan` |
| 大于等于 | `greaterThanOrEqualTo` |
| 小于 | `lessThan` |
| 小于等于 | `lessThanOrEqualTo` |

---

## 数组查询

对于数组类型的字段，你可以查询到`arrayKey`包含了2的对象：

```js
// Find objects where the array in arrayKey contains 2.
query.equalTo("arrayKey", 2);
```

你也可以这样查询到键值包含了2、3、4的的对象：

```js
// Find objects where the array in arrayKey contains all of the elements 2, 3, and 4.
query.containsAll("arrayKey", [2, 3, 4]);
```

---

## 字符串查询

> 如果你想实现一个通用的搜索功能，我们推荐你看看这篇博文：[Implementing Scalable Search on a NoSQL Backend](http://blog.parse.com/learn/engineering/implementing-scalable-search-on-a-nosql-backend/)

我们可以使用`startWith`限制特定开头的字符串值，类似与MySQL中的LIKE操作。这是索引过的，所以在大数据集中也有效：

```js
// Finds barbecue sauces that start with "Big Daddy's".
var query = new Parse.Query(BarbecueSauce);
query.startsWith("name", "Big Daddy's");
```

上面的例子将会查询到`name`以“Big Daddy's”开头的对象，"Big Daddy's" 和 "Big Daddy's BBQ"都会被匹配到，但是"big daddys's"或者"BBQ Sauce:Big Daddy's"不会。

使用正则查询是非常耗费性能的，尤其是超过10万条数据时。Parse限制了给定时间里特定应用可以执行多少次这样的查询。

---

## 关系查询

有几种方法可以查询关系型的数据。如果你想查询某字段为某个对象的话，可以直接像查询其他类型一样使用`equalTo`。比如每个`Comment`都有一个`post`字段指向`Post`，你可以这样查询特定`Post`的评论：

```js
// Assume Parse.Object myPost was previously created.
var query = new Parse.Query(Comment);
query.equalTo("post", myPost);
query.find({
  success: function(comments) {
    // comments now contains the comments for myPost
  }
});
```

如果你想查询某字段包含一个和另一个查询匹配的对象，你可以使用`matchesQuery`。为了查询包含图片的文章的评论，你可以这样：

```js
var Post = Parse.Object.extend("Post");
var Comment = Parse.Object.extend("Comment");
var innerQuery = new Parse.Query(Post);
innerQuery.exists("image");
var query = new Parse.Query(Comment);
query.matchesQuery("post", innerQuery);
query.find({
  success: function(comments) {
    // comments now contains the comments for posts with images.
  }
});
```

反之，你想查询某字段不包含和另一个查询匹配的对象，你可以使用`doesNotMatchesQuery`：

```js
var Post = Parse.Object.extend("Post");
var Comment = Parse.Object.extend("Comment");
var innerQuery = new Parse.Query(Post);
innerQuery.exists("image");
var query = new Parse.Query(Comment);
query.doesNotMatchQuery("post", innerQuery);
query.find({
  success: function(comments) {
    // comments now contains the comments for posts without images.
  }
});
```

你也可以通过`objectId`来查询关联对象：

```js
var post = new Post();
post.id = "1zEcyElZ80";
query.equalTo("post", post);
```

在某些情况下，你想要在一次查询中返回多个类型的关联对象，那么你可以使用`include`方法。比如说你想查询最新的10条评论，并同时查询到关联的文章：

```js
var query = new Parse.Query(Comment);

// Retrieve the most recent ones
query.descending("createdAt");

// Only retrieve the last ten
query.limit(10);

// 评论对象同时包含文章对象
query.include("post");

query.find({
  success: function(comments) {
    // Comments now contains the last ten comments, and the "post" field
    // has been populated. For example:
    for (var i = 0; i < comments.length; i++) {
      // This does not require a network access.
      var post = comments[i].get("post");
    }
  }
});
```

你也可以使用`post.author`的形式嵌套查询，如果你想查询包含了文章和文章作者的评论，你可以这样：

```js
query.include(["post.author"]);
```

你可以多次调用`include`，发起一个包含了多个字段的请求，这个功能在`Parse.Query`的其他查询方法中也有效，比如`first`和`get`。

---

## 统计查询

如果你只是想知道有多少个匹配的对象，但不需要拿到对象的详细信息，你可以使用`count`方法替代`find`方法。比如，你想知道特定玩家玩了多少个游戏：

```js
var GameScore = Parse.Object.extend("GameScore");
var query = new Parse.Query(GameScore);
query.equalTo("playerName", "Sean Plott");
query.count({
  success: function(count) {
    // The count request succeeded. Show the count
    alert("Sean has played " + count + " games");
  },
  error: function(error) {
    // The request failed
  }
});
```

---

## 组合查询

更复杂的查询情况下，你可能需要用到组合查询。一个组合查询是多个子查询的组合\(如"and或"or"\)。

需要注意的是，我们不支持在组合查询的子查询中使用GeoPint或者非过滤类型的约束条件。

### 或查询

如果你想查询多个请求中符合其中一个即可的数据，你可以使用Parse.Query.or方法构建一个由传入的子查询组成的或查询。比如你想查询胜利次数在指定范围的玩家，你可以这样：

```js
var lotsOfWins = new Parse.Query("Player");
lotsOfWins.greaterThan("wins", 150);

var fewWins = new Parse.Query("Player");
fewWins.lessThan("wins", 5);

var mainQuery = Parse.Query.or(lotsOfWins, fewWins);
mainQuery.find()
  .then(function(results) {
    // results contains a list of players that either have won a lot of games or won only a few games.
  })
  .catch(function(error) {
    // There was an error.
  });
```

### 与查询

如果你想查询和所有条件都符合的数据，通常只需要一次请求。你可以添加额外的条件，它实际上就是与查询：

```js
var query = new Parse.Query("User");
query.greaterThan("age", 18);
query.greaterThan("friends", 0);
query.find()
  .then(function(results) {
    // results contains a list of users both older than 18 and having friends.
  })
  .catch(function(error) {
    // There was an error.
  });
```

但如果这个世界真的有这么简单那就好了。有时你可能需要用到与组合查询，Parse.Query.and方法可以构建一个由传入的子程序组成的与查询。比如你想查询用户年龄为16或者18，并且ta的好友少于两人的用户,你可以这样：

```js
var age16Query = new Parse.Query("User");
age16Query.equalTo("age", 16);

var age18Query = new Parse.Query("User");
age18Query.equalTo("age", 18);

var friends0Query = new Parse.Query("User");
friends0Query.equalTo("friends", 0);

var friends2Query = new Parse.Query("User");
friends2Query.greaterThan("friends", 2);

var mainQuery = Parse.Query.and(
  Parse.Query.or(age16Query, age18Query),
  Parse.Query.or(friends0Query, friends2Query)
);
mainQuery.find()
  .then(function(results) {
    // results contains a list of users in the age of 16 or 18 who have either no friends or at least 2 friends
    // results: (age 16 or 18) and (0 or >2 friends)
  })
  .catch(function(error) {
    // There was an error.
  });
```

## 



