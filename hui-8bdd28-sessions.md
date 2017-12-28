# 会话

会话表示已登录用户的的实例，会话会在用户注册或登录时自动创建，也会在用户注销时自动删除。每个用户都有一个独特的`Session`，如果用户从一个已登录设备发起一个登录请求，那么用户原本的`Session`就会被自动覆盖。`Session`对象被存储在Parse的`Session`类中，你可以在Parse.com的数据浏览器上看到他们，我们提供了相应的API集合为你在应用中管理`Session`。

`Session`是Parse `Object`的子类，所以你可以像处理其他Parse对象一样查询、更新、删除`Session`。当你注册或登录用户时，Parse云端会自动创建session，你不需要手动的创建`Session`对象。删除`Session`会造成当前设备上使用此session的用户被注销。

不同于其他的Parse对象，`Session`类在云代码中没有触发器，所以你不需要为`Session`类注册`beforeSave`或`afterSave`。

---

#### 会话属性

`Session`对象具有下面这些字段：

* `sessionToken`（只读）：用于验证Parse API请求的字符串标识，在请求的相应结果中，只有当前`Session`对象包含`sessionToken`。
* `user`（只读）：指向所属的用户对象
* `createdWith`（只读）：关于session是如何创建的信息（e.g `{ "action": "login", "authProvider": "password"}`）。
  * `action`可能的值有：`login`，`signup`，`create`，`upgrade`。当保存的`Session`对象是开发者手动创建的，值就会是`create`；当用户从旧的`sessionToken`更新到新的`sessionToken`，值就会是`upgrade`。
  * `authProvider` 可能的值有，`password`，`anonymous`，`facebook`，`twitter`。





