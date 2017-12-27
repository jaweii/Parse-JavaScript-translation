# 用户

在许多应用的核心功能中，都有用户账户的概念，它让用户以一种安全的方式访问自己的信息。我们提供了专门的用户类，通过调用`Parse.User`可以自动处理大量用户管理的功能。

通过这个类，你将可以在你的应用中使用用户管理功能。

`Parse.User`是`Parse.Object`的子类，它们具有一样的特征，比如灵活的模式，自动持久化，和键值接口。`Parse.Object`上的所有方法，`Parse.User`也有，不同的是，`Parse.User`还具备一些额外的用户操作方法。

---

#### 用户属性

Parse.User具备的一些属性将它和`Parse.Object`区分开来：

* username:用户名，必须
* password:密码，必须
* email:邮箱，可选

我们将通过使用`Parse.User`的各个使用例子来一一详细了解。

---

#### 注册

你的应用可能首先要做的就是要求用户中注册，下面的代码是典型的注册示例：

```js
var user = new Parse.User();
user.set("username", "my name");
user.set("password", "my pass");
user.set("email", "email@example.com");

// other fields can be set just like with Parse.Object
user.set("phone", "415-392-0202");

user.signUp(null, {
  success: function(user) {
    // Hooray! Let them use the app now.
  },
  error: function(user, error) {
    // Show the error message somewhere and let the user try again.
    alert("Error: " + error.code + " " + error.message);
  }
});
```

此调用是异步在你的Parse应用上创建一个新用户。在此之前，它会检查你的账号和邮箱是否未被使用。另外，在云端密码也用bcrypt进行了加密，我们不会将密码明文存储，也不会将密码明文传输给客户端。

需要注意的是，我们使用的是`signUp`方法，而不是`save`方法。新的`Parse.User`应该始终是通过`signUp`方法创建的，而后对用户的更新才可以用`save`方法完成。

如果注册失败，你应该查看返回的错误对象，最有可能的情况是用户名或邮箱已经被别的用户使用，你应该告知用户，要求用户使用不同的用户名或邮箱。

你也可以决定使用邮箱地址作为用户名，只需要要求用户输入他们的邮箱，不过邮箱是保存在`username`属性上，`Parse.User`会正常工作。我们将会在密码重置章节回顾这是如何处理的。

---

#### 登录

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

#### 邮箱验证

启用应用设置中的邮箱验证，可以让应用为已验证用户保留部分体验。邮箱验证添加一个`emailVerified`字段在`Parse.User`对象上，当`Parse.User`的`email`被设置或修改了，`emailVerified`就会被设置成`false`，Parse会发送一封链接邮件给用户，验证通过后`emailVerified`就会被设置为`true`。

有三种`emailVerified`状态需要考虑：

1. `true` - 用户通过Parse发送的验证邮件验证成功。用户账号首次创建的时候，值永远不会为`true`。
2. `false`- 用户还没有验证ta的邮箱。
3. `missing`- 当邮箱验证未启用，或者`Parse.User`没有`email`值。

---

#### 当前用户

如果用户每次打开你的应用都要登录，那是很麻烦的 。你可以通过使用缓存的当前`Parse.User`对象，避免这个麻烦。

请注意，这个功能在Node.js环境中，默认是禁止的，以阻止在服务端配置上有状态的用法。

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

#### 设置当前用户

如果你已经创建了自己的身份验证程序，或者用户通过其他方式登录到了服务端，你可以传入session token到`become`方法，这个方法会在设置当前用户前确认session token可用。

```js
Parse.User.become("session-token-here").then(function (user) {
  // The current user is now set to user.
}, function (error) {
  // The token could not be validated.
});
```

---

#### 用户安全

`Parse.User`类默认情况下是安全的，它的数据默认只有用户自己可以修改，其他用户只有读的权限，除非`Parse.User`经过权限验证可以被修改，否则只读。

特别是，除非你通过了`logIn`或者`signUp`的验证，否则你无法使用`save`或者`delete`方法。这确保了只有用户自己可以修改自己的数据。

下面的代码说明了此安全策略：

```js
var user = Parse.User.logIn("my_username", "my_password", {
  success: function(user) {
    user.set("username", "my_new_username");  // attempt to change username
    user.save(null, {
      success: function(user) {
        // This succeeds, since the user was authenticated on the device

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

如果你想检查一个`Parse.User`是否通过验证，你可以使用`authenticated`方法。你不需要去检查那些通过`authenticated`方法获取的`Parse.User`的`authenticated`。

---

#### 对象安全

有些应用在`Parse.User`上的安全策略也可以应用在其他对象上，对于任意一个对象，你可以指定哪个用户可以读取和修改。为了支持这种安全类型，每一个对象都有一个`access control list`，通过`Parse.ACL`类实现。

要指定一个对象只有某个用户可以读写，最简单的方法始使用`Parse.ACL`。这是通过为`Parse.User`初始化一个`Parse.ACL`完成的：`new Parse.ACL(user)`生成一个`Parse.ACL`，它限制了`user`的访问。当一个对象被保存后，它的ACL也会被更新，就像其他的属性一样。

那么，要创建一个只有当前用户可以访问的私人笔记，可以这样：

```
var Note = Parse.Object.extend("Note");
var privateNote = new Note();
privateNote.set("content", "This note is private!");
privateNote.setACL(new Parse.ACL(Parse.User.current()));
privateNote.save();
```



