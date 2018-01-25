# 云代码使用指南

云代码，就是运行在云端，而非客户端的函数代码。

云代码是构建在ParseServer中的，默认云代码的入口文件是`./cloud/main.js`。

## 云函数

就像你在代码中调用别人的函数一样，调用云函数就是调用云端的函数。

假设我们在云代码文件中定义一个`averageStart`函数，功能是通过传入的电影名返回评论数量：

```js
Parse.Cloud.define("averageStars", function(request, response) {
  const query = new Parse.Query("Review");
  query.equalTo("movie", request.params.movie);
    .find()
    .then((results) => {
      let sum = 0;
      for (let i = 0; i < results.length; ++i) {
        sum += results[i].get("stars");
      }
      response.success(sum / results.length);
    })
    .catch(() =>  {
      response.error("movie lookup failed");
    });
});
```

然后，在客户端可以这样调用`averageStarts`方法：

```js
Parse.Cloud.run('averageStars', { movie: 'The Matrix' }).then(function(ratings) {
  // ratings = 4.5
});
```

## Cloud Jobs

云作业，当你需要定义一个执行时间很长的云函数，并且不用等待它响应，你就可以使用云作业。我们来定义一个云作业：

```js
Parse.Cloud.job("myJob", function(request, status) {
  // 请求参数
  const params = request.params;
  //请求头
  const headers = request.headers;

  // get the parse-server logger
  const log = request.log;

  // 更新作业状态
  status.message("I just started");
  doSomethingVeryLong().then(function(result) {
    //作业完成
    status.success("I just finished");
  })
  .catch(function(error) {
    // 作业失败
    status.error("There was an error");
  });
});
```

云作业需要提供master key才能调用，你可以通过REST API调用：

```
curl -X POST -H 'X-Parse-Application-Id: appId' -H 'X-Parse-Master-Key: masterKey' https://my-parse-server.com/parse/jobs/myJob
```

## 触发器

注册对应class表的钩子后，对应的事件将触发对应的钩子。

在下面触发器中，你需要注意，如果你要注册ParseServer预定义的class表，比如User表，你不应该给第一个参数传入字符\(“Review”\)，而是传入对象，比如`Parse.User`。

### beforeSave

当云端收到保存某对象的请求后，在保存某对象前，会触发此方法。

若在方法中调用response.error，对象将不会被保存，客户端返回错误；若在方法中调用response.success，对象将会被成功保存。

```js
//注册Review表的beforeSave触发器，有对象被保存前，触发此方法。
Parse.Cloud.beforeSave("Review", function(request, response) {
  if (request.object.get("stars") < 1) {
    response.error("少于1星，不能获取");
  } else if (request.object.get("stars") > 5) {
    response.error("多余5星，不能获取");
  } else {
    response.success();
  }
});
```

我们还可以在函数中修改对象，比如，将评论内容限制在140字以内：

```js
Parse.Cloud.beforeSave("Review", function(request, response) {
  var comment = request.object.get("comment");
  if (comment.length > 140) {
    // Truncate and add a ...
    request.object.set("comment", comment.substring(0, 137) + "...");
  }
  response.success();
});
```

### afterSave

在某对象保存后，会触发此方法。

假设你要在一条文章评论保存后，更新文章评论数量的字段：

```js
Parse.Cloud.afterSave("Comment", function(request) {
  const query = new Parse.Query("Post");
  query.get(request.object.get("post").id)
    .then(function(post) {
      post.increment("comments");
      return post.save();
    })
    .catch(function(error) {
      console.error("Got an error " + error.code + " : " + error.message);
    });
});
```

不管上面方法中有没有报错，客户端都会返回成功。

### beforeDelete

在对象被删除前，会触发此方法。

假设你有一个相册应用，当用户发起了删除某个相册的操作，需要先检查这个相册里有没有照片，如果有就不允许删除，可以这样：

```js
Parse.Cloud.beforeDelete("Album", function(request, response) {
  const query = new Parse.Query("Photo");
  query.equalTo("album", request.object);
  query.count()
    .then(function(count) {
      if (count > 0) {
        response.error("Can't delete album if it still has photos.");
      } else {
        response.success();
      }
    })
    .catch(function(error) {
      response.error("Error " + error.code + " : " + error.message + " when getting photo count.");
    });
});
```

若在方法中调用response.error，对象将不会被保存，客户端返回错误；若在方法中调用response.success，对象将会被成功保存。

### afterDelete

在对象被删除后，会触发此方法。

假设你在一篇博文被删除后，同时删除这篇博文的所有评论，可以这样：

```js
Parse.Cloud.afterDelete("Post", function(request) {
  const query = new Parse.Query("Comment");
  query.equalTo("post", request.object);
  query.find()
    .then(Parse.Object.destroyAll)
    .catch(function(error) {
      console.error("Error finding related comments " + error.code + ": " + error.message);
    });
});
```

### beforeFind

在find查询进入后，会触发此方法。

如果你想为查询加上额外的限制、条件、你可以注册这个触发器。

#### 例子

```js
// 属性
Parse.Cloud.beforeFind('MyObject', function(req) {
  let query = req.query; //  Parse.Query
  let user = req.user; //  user
  let triggerName = req.triggerName; // beforeFind
  let isMaster = req.master; // 如果使用查询masterKey
  let isCount = req.count; // 如果是count查询
  let logger = req.log; //logger
  let installationId = req.installationId; // installationId
});

// 限制返回字段
Parse.Cloud.beforeFind('MyObject', function(req) {
  let query = req.query; // Parse.Query
  query.select(['key1', 'key2']);
});

// 异步支持
Parse.Cloud.beforeFind('MyObject', function(req) {
  let query = req.query;
  return aPromise().then((results) => {
    // do something with the results
    query.containedIn('key', results);
  });
});

// 返回一个不同的查询
Parse.Cloud.beforeFind('MyObject', function(req) {
  let query = req.query;
  let otherQuery = new Parse.Query('MyObject');
  otherQuery.equalTo('key', 'value');
  return Parse.Query.or(query, otherQuery);
});

// 拒绝查询
Parse.Cloud.beforeFind('MyObject', function(req) {
  // throw an error
  throw new Parse.Error(101, 'error');

  // rejecting promise
  return Promise.reject('error');
});
```



