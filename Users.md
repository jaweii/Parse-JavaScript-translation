# 用户

在许多应用的核心功能中，都有用户账户的概念，它让用户以一种安全的方式访问自己的信息。我们提供了专门的用户类，通过调用`Parse.User`可以实现很多用户管理的功能。

`Parse.User`是`Parse.Object`的子类，它们具有一样的特征，比如灵活的模式，自动持久化，和键值接口。`Parse.Object`上的所有方法，`Parse.User`也有，不同的是，`Parse.User`还具备一些额外的用户操作方法。

---

## 用户属性

Parse.User具备一些`Parse.Object`没有的属性：

* username:用户名，必须
* password:密码，必须
* email:邮箱，可选

我们将通过使用`Parse.User`的各个使用例子来一一详细了解。

---

## 注册

你的应用可能首先要做的就是要求用户注册，下面的代码是典型的注册示例：

```js
var user = new Parse.User();
user.set("username", "my name");
user.set("password", "my pass");
user.set("email", "email@example.com");

// 还可以设置电话号码
user.set("phone", "415-392-0202");

user.signUp(null, {
  success: function(user) {
    // 注册成功.
  },
  error: function(user, error) {
    // Show the error message somewhere and let the user try again.
    alert("Error: " + error.code + " " + error.message);
  }
});
```

此调用是异步在你的Parse应用上创建一个新用户。在此之前，它会检查你的账号和邮箱是否被使用。另外，在云端密码也用bcrypt进行了加密，我们不会将密码明文存储，也不会将密码明文传输给客户端。

需要注意的是，我们使用的是`signUp`方法，而不是`save`方法。新的`Parse.User`应该始终是通过`signUp`方法创建的，而后对用户信息的更新才可以用`save`方法完成。

如果注册失败，你应该查看返回的错误对象，最有可能的情况是用户名或邮箱已经被别的用户使用，你应该告知用户，要求用户使用不同的用户名或邮箱。

你也可以决定使用邮箱地址作为用户名，只需要要求用户输入他们的邮箱，不过邮箱是保存在`username`属性上，`Parse.User`会正常工作。我们将会在密码重置章节回顾这是如何处理的。

---

## 登录

当然，在我们允许用户注册以后，你将来还需要他们登录，以访问他们的账户。为了实现此功能，你可以使用用户类的`logIn`方法：

```js
Parse.User.logIn("myname", "mypass", {
  success: function(user) {
    // Do stuff after successful login.
  },
  error: function(user, error) {
    // The login failed. Check error to see why.
  }
});
```

---

## 邮箱验证

启用应用设置中的邮箱验证，可以让应用为已验证用户保留部分体验。邮箱验证添加一个`emailVerified`字段在`Parse.User`对象上，当`Parse.User`的`email`被设置或修改了，`emailVerified`就会被设置成`false`，Parse会发送一封链接邮件给用户，验证通过后`emailVerified`就会被设置为`true`。

`emailVerified`有三种状态：

1. `true` - 用户通过Parse发送的验证邮件验证成功。用户账号首次创建的时候，值永远不会为`true`。
2. `false`- 用户还没有验证ta的邮箱。
3. `missing`- 当邮箱验证未启用，或者`Parse.User`没有`email`值。

---

## 当前用户

如果用户每次打开你的应用都要登录，那是很麻烦的 。你可以通过使用缓存的当前`Parse.User`对象，避免这个麻烦。

请注意，这个功能在Node.js环境中，默认是禁止的，这是为了阻止在服务端配置上有状态的用法。

无论何时，当你使用`signup`或`login`方法后，用户登录信息都被缓存到了`localStorage`中，你可以把它当做用户会话使用，以此判断用户是否登录。

```js
var currentUser = Parse.User.current();
if (currentUser) {
    // do stuff with the user
} else {
    // show the signup or login page
}
```

你可以通过注销用户来清除当前用户：

```js
Parse.User.logOut().then(() => {
  var currentUser = Parse.User.current();  // this will now be null
});
```

---

## 设置当前用户

如果你已经创建了自己的身份验证程序，或者用户通过其他方式登录到了服务端，你可以传入session token到`become`方法，这个方法会在设置当前用户前确认session token是否可用。

```js
Parse.User.become("session-token-here").then(function (user) {
  // The current user is now set to user.
}, function (error) {
  // The token could not be validated.
});
```

---

## 用户安全

`Parse.User`类默认情况下是安全的，它的数据默认只有用户自己可以修改，其他用户只有读的权限，除非`Parse.User`权限设置了允许其他用户修改。

特别注意，除非你通过了`logIn`或者`signUp`的验证，否则你无法使用`save`或者`delete`方法。这确保了只有用户自己可以修改自己的数据。

下面的代码说明了此安全策略：

```js
var user = Parse.User.logIn("my_username", "my_password", {
  success: function(user) {
    user.set("username", "my_new_username");  //修改用户名
    user.save(null, {
      success: function(user) {
        // 修改成功

        // Get the user from a non-authenticated method
        var query = new Parse.Query(Parse.User);
        query.get(user.objectId, {
          success: function(userAgain) {
            userAgain.set("username", "another_username");
            userAgain.save(null, {
              error: function(userAgain, error) {
                // This will error, since the Parse.User is not authenticated
              }
            });
          }
        });
      }
    });
  }
});
```

从`Parse.User.current()`获取的`Parse.User`永远是通过了验证的。

如果你想检查一个`Parse.User`是否通过登录验证，你可以使用`authenticated`方法。你不需要去检查那些通过`authenticated`方法获取的`Parse.User`。

---

## 对象安全

有些应用在`Parse.User`上的安全策略也可以应用在其他对象上，对于任意一个对象，你可以指定哪个用户可以读取和修改。为了支持这种安全类型，每一个对象都有一个`access control list`，通过`Parse.ACL`类实现。

要指定一个对象只有某个用户可以读写，最简单的方法始使用`Parse.ACL`。这是通过为`Parse.User`初始化一个`Parse.ACL`完成的：`new Parse.ACL(user)`生成一个`Parse.ACL`，它限制了`user`的访问。当对象被保存后，它的ACL会被更新，就像其他的属性一样。

那么，要创建一个只有当前用户可以访问的私人笔记，可以这样：

```js
var Note = Parse.Object.extend("Note");
var privateNote = new Note();
privateNote.set("content", "This note is private!");
privateNote.setACL(new Parse.ACL(Parse.User.current()));
privateNote.save();
```

这个note将只有当前用户可以访问，并且，当前用户登录的任何设备都可以访问。这个功能在你想启用跨设备访问用户数据时非常有用，权限也可以给多个用户授权，你可以使用`setReadAccess`和`setWriteAccess`为`Parse.ACL`添加独有的权限。比如，你有一条消息要发送给几个用户，他们中的每一个用户都有权限读取和修改：

```js
var Message = Parse.Object.extend("Message");
var groupMessage = new Message();
var groupACL = new Parse.ACL();

// 用户数组
for (var i = 0; i < userList.length; i++) {
  groupACL.setReadAccess(userList[i], true);
  groupACL.setWriteAccess(userList[i], true);
}

groupMessage.setACL(groupACL);
groupMessage.save();
```

你也可以使用`setPublicReadAccess`和`setPublicWriteAccess`一次授权所有用户，这在留言板中发布评论这种模式中很有用。要创建一篇文章只有它的作者可以编辑，但是所有人可以访问，可以这样：

```js
var publicPost = new Post();
var postACL = new Parse.ACL(Parse.User.current());
postACL.setPublicReadAccess(true);
publicPost.setACL(postACL);
publicPost.save();
```

禁止的操作，比如删除一个对象，但是你没有修改的权限，会导致一个`Parse.Error.OBJECT_NOT_FOUND`错误码。出于安全的目的，这样可以阻止客户端判断哪些对象是受保护的，哪些对象是完全不存在的。

---

## 重置密码

现实中，只要你已将用户密码传入系统，用户就会忘记密码。鉴于这种情况，我们的库提供了一种方法让他们重设密码。

要触发重置密码的流程，先询问用户的邮箱地址，然后调用：

```js
Parse.User.requestPasswordReset("email@example.com", {
  success: function() {
  // 重置密码请求已发出
  },
  error: function(error) {
    // Show the error message somewhere
    alert("Error: " + error.code + " " + error.message);
  }
});
```

这会尝试用给定的邮箱和用户的`email`或`username`字段进行匹配，并且会发送一封密码重置的邮件。你可以选择让用户用邮箱作为他们的用户名，也可以把邮箱单独的存在`email`字段。

密码重置流程如下：

1. 用户通过输入他们的邮箱来请求重置密码
2. Parse发送一封 邮件到他们的邮箱，其中包含一个密码重置的链接
3. 用户点击链接，将被引导到指定的Parse页面，在这个页面上重置密码
4. 用户输入新的密码后，他们的密码就被重置为新密码

注意在此流程中的消息传递所引用的应用名，是你创建Parse应用时的app名。

---

## 查询

要查询用户，你可以为用户创建一个新Parse.Query：

```js
var query = new Parse.Query(Parse.User);
query.equalTo("gender", "female");  // find all the women
query.find({
  success: function(women) {
    // Do stuff
  }
});
```

---

## 联合查询

假如你在开发一个博客应用，想要提交一篇新博文，以及查询作者的所有文章，你可以这样：

```js
var user = Parse.User.current();

// 提交一篇新博文
var Post = Parse.Object.extend("Post");
var post = new Post();
post.set("title", "My New Post");
post.set("body", "This is some great content.");
post.set("user", user);
post.save(null, {
  success: function(post) {
    // 查询作者的所有文章
    var query = new Parse.Query(Post);
    query.equalTo("user", user);
    query.find({
      success: function(usersPosts) {
        // userPosts contains all of the posts by the current user.
      }
    });
  }
});
```

## 手机验证码登录

参考译者的另外写的教程[Parse服务的手机验证码用户注册/登录教程](http://buzhundong.com/post/Parse%E6%9C%8D%E5%8A%A1%E7%9A%84%E6%89%8B%E6%9C%BA%E9%AA%8C%E8%AF%81%E7%A0%81%E7%94%A8%E6%88%B7%E6%B3%A8%E5%86%8C-%E7%99%BB%E5%BD%95%E6%95%99%E7%A8%8B.html)

## 脸书登录

## 领英登录



