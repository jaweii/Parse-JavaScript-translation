```
查询
```

我们已经知道了`Parse.Query`是如何使用`get`从`Parse`查询一个对象的，下面还有许多其他方法，比如你可以一次查询多个对象，可以在一个查询上限制条件，等等。

#### 基本查询

很多时候，`get`不能完全满足你查询想要的对象。

`Parse.Query`提供了其他的方法，可以让你查询一个对象列表，而不是单单一个对象。

最常用的方法始创建一个`Parse.Query`，为其设置查询条件，然后使用`find`方法查询匹配的对象列表。比如，要查询一个唯一的`playerName`的得分，我们使用`equalTo`方法来设定查询条件：

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

#### 查询条件

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

如果你想精确的查询一个结果，你可以用更方便的`first`方法替代`find`方法：

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
// Restricts to wins < 50
query.lessThan("wins", 50);

// Restricts to wins <= 50
query.lessThanOrEqualTo("wins", 50);

// Restricts to wins > 50
query.greaterThan("wins", 50);

// Restricts to wins >= 50
query.greaterThanOrEqualTo("wins", 50);
```

如果你想查询与数组中任意值匹配的对象，你可以使用`containedIn`方法，并提供一个可接受值的数组。这样可以一个请求完成多个请求的功能。比如，你想查询由数组中任意玩家组成的分数结果：

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

你可以使用`meachesKeyInQuery`方法查询对象，对象中的某个键值是和另一个请求返回的一组结果中的键值匹配的。比如，如果你有一个包含了运动队的类，并且你保存了用户的老家信息在user类中，你现在用查询获胜队伍的成员家乡信息，你可以这样请求：

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



