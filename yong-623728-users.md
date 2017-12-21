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

#### 登录

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

需要注意的是，我们使用的是`signUp`方法，而不是`save`方法。新的`Parse.User`应该始终是通过`signUp`方法创建的，而后对用户的更新才可以`save`方法完成。

