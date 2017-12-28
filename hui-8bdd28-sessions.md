# 会话

会话表示已登录用户的的实例，会话会在用户注册或登录时自动创建，也会在用户注销时自动删除。每个用户都有一个独特的`Session`，如果用户从一个已登录设备发起一个登录请求，那么用户原本的`Session`就会被自动覆盖。`Session`对象被存储在Parse的`Session`类中，你可以在Parse.com的数据浏览器上看到他们，我们提供了相应的API集合为你在应用中管理`Session`。

`Session`是Parse `Object `的子类，所以你可以像处理其他Parse对象一样查询、更新、删除`Session`。当你注册或登录用户时，Parse云端会自动创建session，你不需要手动的创建`Session`对象。删除`Session`会造成当前设备上使用此session的用户被注销。

不同于其他的Parse对象，`Session`类在云代码中没有触发器，所以你不需要为`Session`类注册`beforeSave`或`afterSave`。

