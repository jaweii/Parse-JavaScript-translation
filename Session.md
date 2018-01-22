# 会话

会话表示已登录用户的的实例，会话会在用户注册或登录时自动创建，也会在用户注销时自动删除。每个用户都有一个独特的`Session`，如果用户从一个已登录设备发起一个登录请求，那么用户原本的`Session`就会被自动覆盖。`Session`对象被存储在Parse的`Session`类中，你可以在Parse.com的数据浏览器上看到他们，我们提供了相应的API集合为你在应用中管理`Session`。

`Session`是Parse `Object`的子类，所以你可以像处理其他Parse对象一样查询、更新、删除`Session`。当你注册或登录用户时，Parse云端会自动创建session，你不需要手动的创建`Session`对象。删除`Session`会造成当前设备上使用此session的用户被注销。

不同于其他的Parse对象，`Session`类在云代码中没有触发器，所以你不需要为`Session`类注册`beforeSave`或`afterSave`。

---

## 会话属性

`Session`对象具有下面这些字段：

* `sessionToken`（只读）：用于验证Parse API请求的字符串标识，在请求的相应结果中，只有当前`Session`对象包含`sessionToken`。
* `user`（只读）：指向所属的用户对象
* `createdWith`（只读）：关于session是如何创建的信息（e.g `{ "action": "login", "authProvider": "password"}`）。

  * `action`可能的值有：`login`，`signup`，`create`，`upgrade`。当保存的`Session`对象是开发者手动创建的，值就会是`create`；当用户从旧的`sessionToken`更新到新的`sessionToken`，值就会是`upgrade`。
  * `authProvider` 可能的值有，`password`，`anonymous`，`facebook`，`twitter`。

* `restricted`（只读）：表示session是否受限制。

  * 受限制session对Parse上的`User`、`Session`、`Role`没有写权限，受限制的会话也不能读取未受限制的会话。
  * 通过用户注册或登录，由Parse云端自动创建的session，全都是未受限制的；由开发者通过客户端手段创建保存的new `Session`对象，都是受限制的。

* `expiresAt`（只读）：`Session`对象将被自动删除的日期（UTC），你可以在你应用的Parse面板设置页面进行设置（一年不活动过期或永不过期）。

* `installationId`（只能设置一次）：表示会话从哪里登录。对于Parse SDK，这个字段会在用户注册或登录时自动设置。除了`installationId`，所有的字段只能由Parse云端自动设置。但请记住，所有已登录设备都可以读取相同用户的其他session，除非你禁用表级权限。

---

## 处理无效的session token

使用可撤回session，如果你的`Session`对象被Parse云端删除了，那么你对应的当前用户session也会失效，这在某些情况下可能会用上，比如你从用户`Session`管理界面上，将用户强制注销，或者你手动的从云端删除了session。

session还可以通过设置自动过期来自动删除。 如果一个设备上的session token无法和Parse云端匹配上，那么所有来自此设备的请求都将返回错误信息：“Error 209: invalid session token”。

要处理这个错误，我们推荐写一个全局的功能函数，并在你的Parse请求发生错误时调用，然后你就可以在全局函数中处理无效“invalid session token”。同时你应该提示用户重新登录，这样用户就能获得新的session token。像这样：

```js
function handleParseError(err) {
  switch (err.code) {
    case Parse.Error.INVALID_SESSION_TOKEN:
      Parse.User.logOut();
      ... // 提示用户重新登录
      ... // If Express.js, redirect the user to the log in route
      break;

    ... // Other Parse API errors that you want to explicitly handle
  }
}

// 为所有api请求的报错调用全局错误处理方法
query.find().then(function() {
  ...
}, function(err) {
  handleParseError(err);
});
```

---

## 会话安全

`Session`对象只有user字段指定的用户可以访问，所有的`Session`对象都有一个ACL，只允许指定用户读写，你不能修改这个ACL。这意味着会话的查询只会返回与当前登录用户相匹配的会话。

当你使用login方法登录时，Parse云端将会自动创建一个未受限制的`Session`对象，使用signup、脸书、推特登录也是一样。

通过客户端SDK手动创建的`Session`对象始终是受限制的，你不能使用api手动创建一个未受限制的会话。

受限制的会话禁止创建、修改、更新`User`、`Session`、`Role`中的任何数据，受限制的会话也不能读取未受限制的会话。受限制的会话在“Parse for IoT”设备（例如 Arduino、Embedded C）中很有用，这在不可信的物理环境用的也比手机应用环境用的多。无论如何，记住受限制的会话仍然可以读取`User`、`Session`、`Role`类中的数据，并且可以像正常session一样读写其他class中的任何数据。这对IoT设备的安全和加密存储仍然非常重要。

如果你想阻止受限制会话修改`User`、`Session`、`Role`以外的class，你可以为在云代码中为其class写一个`beforeSave`处理器：

```js
Parse.Cloud.beforeSave("MyClass", function(request, response) {
  Parse.Session.current().then(function(session) {
    if (session.get('restricted')) {
      response.error('禁止写操作');
    }
    response.success();
  });
});
```

如果你想在Parse上为`Session`表设置表级权限（CLP），以限制session的读写，但不要在用户注册、登录、注销时限制Parse云自动增删session。我们推荐你禁用`Session`所有不需要的CLP。下面有一些`Session` CLP的用例：

* **Find、Delete** - 用以构建一个供用户查看其他登录的在线设备，并且注销其他设备的用户.的功能。如果你不需要个功能，你应该禁用这个权限。
* **Create **- 用于“Parse for IoT”应用，如果是的app是手机应用或web应用，如果你的IoT设备不需要用户会话，你应该禁用这个权限。
* **Get、Update、Add Field** - 除非你需要这些操作，否则你应该禁用这些权限。



