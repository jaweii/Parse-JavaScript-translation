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

启用应用设置中的邮箱验证，可以让应用为已验证用户保留部分体验。邮箱验证提那家了一个`emailVerified`字段在`Parse.User`对象上，当`Parse.User`的`email`被设置或修改了，`emailVerified`就会被设置成`false`，Parse会发送一封链接邮件给用户，验证通过后`emailVerified`就会被设置为`true`。

有三种`emailVerified`状态需要考虑：

1. `true` - 用户通过Parse发送的验证邮件验证成功。用户账号首次创建的时候，值永远不会为`true`。
2. `false `- 用户还没有验证ta的邮箱。
3. `missing `- 当邮箱验证未启用，或者`Parse.User`没有`email`值。



