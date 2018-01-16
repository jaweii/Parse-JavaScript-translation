# 角色

随着你的应用的规模增长，你可能发现你需要更高颗粒度的权限管理来控制用户访问，而不是用户链接ACL的方法管理。要解决这个需求，Parse提供了基于用户角色的权限管理。角色提供了一种合理的方法，通过访问权限对用户分组。角色对象包含了用户和其他角色。任何角色被允许访问的数据，那么角色所包含的用户和角色也可以访问该数据。

比如，在具有角色扮演的应用中，可能有一些用户身份是主人，主人有权限领取皮鞭、项圈等；可能还有一些用户身份是宠物，无权限领取那些物件。如果一个主人有很多宠物，一个个给宠物设置权限是很麻烦的。通过角色，就不必手动的为每个用户授予访问每个资源的权限。

我们提供了一个专门的类`Parse.Role`，它在你的客户端代码中代表角色对象，`Parse.Role`是`Parse.Object`的子类，并且具有`Parse.Object`的所有功能，比如灵活的模型、自动持久化、键值接口。`Parse.Object`的所有方法，`Parse.Role`都具备，不同的是，`Parse.Role`有角色管理功能。

---

## Parse.Role属性

`Parse.Role`的这些属性`Parse.Object`没有：

* name：角色名，必需，并且只能在创建的时候设置。这个值是由字母、数字、空格或下划线组成，name将被用来识别角色，而不需要objectId。
* users：继承此角色被授予权限的用户。
* roles：继承此角色被授予权限的角色。

---

## 角色安全

`Parse.Role`使用的是和Parse上其他对象一样的安全模型（ACL），不同的是它需要被显示的设置。通常，只有权限足够大的用户，比如管理员，才可以创建和修改角色，所以你应该定义合理的ACL。记住，如果你授予一个用户对角色的写权限，那么这个用户可以添加其他用户到这个角色中，甚至删除所有角色。

要创建一个新的`Parse.Role`，你要这样写：

```js
// 为此角色禁用写权限
var roleACL = new Parse.ACL();
roleACL.setPublicReadAccess(true);
var role = new Parse.Role("Administrator", roleACL);
role.save();
```

你可以通过`Parse.Role`上的users和roles关联对象，添加用户和角色来继承你的新角色权限：

```js
var role = new Parse.Role(roleName, roleACL);
role.getUsers().add(usersToAddToRole);
role.getRoles().add(rolesToAddToRole);
role.save();
```

当你指派ACL到你的角色时应尤其小心，确保只有有权修改的用户可以修改数据。

---

## 基于角色的对象安全

现在你为你的应用创建了一组角色，你可以在ACL中使用他们定义对象权限。每一个`Parse.Object`可以指定一个`Parse.ACL`，它提供了访问控制列表，可以标示哪些用户和角色可以读写对象。

给予一个对象某角色的读写权限非常简单：

```js
var moderators = /* 查询到的 Parse.Role */;
var wallPost = new Parse.Object("WallPost");
var postACL = new Parse.ACL();
postACL.setRoleWriteAccess(moderators, true);
wallPost.setACL(postACL);
wallPost.save();
```

你也可以不使用查询获取角色，直接给ACL指定角色名：

```js
var wallPost = new Parse.Object("WallPost");
var postACL = new Parse.ACL();
postACL.setRoleWriteAccess("Moderators", true);
wallPost.setACL(postACL);
wallPost.save();
```

---

## 角色等级

如上所述，一个角色可以包含另一个角色，两个角色间建立了父子关系，这种关系的结果就是，当父对象的权限被承认后，所有的子对象也会被承认。

这些类型的关系通常在具有用户管理的应用中存在，比如论坛。用户的小部分子集是管理员，具备最高级别的权限，可以创建新板块，设置全局消息等。还有部分用户是版主，他们负责确保用户创建的内容是合适的。任何具备管理员权限的用户也会具备版主的权限。要完成这样的关系，你要使你的管理员角色成为版主角色的子角色：

```js
var administrators = /* 你的管理员角色 */;
var moderators = /* 你的版主角色 */;
moderators.getRoles().add(administrators);
moderators.save();
```



