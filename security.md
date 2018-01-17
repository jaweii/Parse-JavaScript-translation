# 安全

随着应用的逐步完善，你会想使用Parse的安全功能来保护数据。本章就是介绍如何保证应用安全。

如果你的应有有潜在隐患，不只是开发者受到损失，你的用户也有可能受到损失。阅读本章，可以在你发布应用前，预防潜在隐患。

## 客户端 VS. 服务端

当你的应用首次连接云端时，它会通过App ID和client key来标识自己，这些都不是秘密，他们本身也不能保证应用安全。App ID和client key作为你应用的一部分，任何人都可以通过反编译或者网络监听来得到它们，尤其是在JavaScript中，只需要在浏览器中查看源文件就可以找到你的密钥。

这就是为什么Parse有很多安全功能来帮助你保护数据安全。

client key在应用发布后就给了用户，所以任何可以用客户端密钥完成的事情都可以由公众，甚至是恶意的黑客来完成。

另一方面，master key定义了一种安全的机制。使用master key可以忽略应用的所有安全机制，比如表级别权限、ACL。master key就像服务器的root权限，所以你应该保管好你的master key，就像保管你的root密码一样。

你应该限制客户端的权限，把敏感的操作在云代码中用master key完成。你将在[Implementing Business Logic in Cloud Code](http://docs.parseplatform.org/js/guide/#implementing-business-logic-in-cloud-code)这一节学会使用云代码。

最后，建议在你服务器使用HTTPS和SSL，避免中间人攻击和运营商劫持，当然，不使用的话Parse也能正常工作。

## 表级别权限

第二个安全级别在于数据和结构级别， 执行这一级别的安全级别将会限制客户端对云端数据的访问。这在你首次使用Parse应用时就已经设置好了，以便你更高效的开发。举例：

* 客户端可以在云端创建新的class表
* 客户端可以给class表增加新字段
* 客户端可以查询和修改云端的数据

你可以配置这些权限，应用到你应用的每个用户，或者指定的用户/角色。角色是指包含其他用户或其他角色的组，你可以将角色指派给一个对象来限制对象的使用。授权给角色的任何权限，也会同样的授权给角色的子角色和子用户，这使你可以为你的应用创建权限级别。

当你自信有了正确的class表和表关系，你应该参考下面的建议。

几乎所有class表的权限都应该做一定程度调整。对于所有对象都具有相同权限的对象，表级别权限设置将会是最有效的，比如，某个场景中有个class表，里面的数据所有都可以读，但只有一个人可以写。

_禁止创建class表_

在一开始，你就可以配置你的应用，使客户端无法在云端创建新class表，这在你的Parse服务配置里将`allowClientClassCreation`设置为`false`即可。一旦被设置后，就只能通过数据浏览器或者master key才能创建新的数据表，这可以阻止攻击者创建无数无意义的class表填充你的数据库。

你可以从[Configuring your ParseServer](https://github.com/parse-community/parse-server#configuration) 的Readme中查阅Parse服务的配置。

_配置表级别权限_

Parse允许你为每个class表指定哪些操作是允许的，从而限制客户端访问或修改class表的方法。要修改这些设置，进入数据浏览器，选择class表，然后点击"Security"按钮。

你可以为选中的class表配置客户端执行一下操作的能力：

* **Read**
  * **Get**:使用Get的权限，如果用户知道对象id，则可以通过id拉取对象。
  * **Find**:任何人使用查找权限可以查找表中的所有对象，无论他们是否知道对象id。任何表使用Find:Public权限后，将会对公众完全可读，除非你为表中对象设置了另外的ACL。
* **Write**
  * **Update**: 任何人使用Update权限可以修改表中的任何对象字段，除非你为表中对象设置了另外的ACL。对于公开只读数据，比如游戏等级或资产，你应该为数据禁用Update权限。
  * **Create**: 和Update一样，任何人可以在class表中创建新对象，除非你为表中对象设置了另外的ACL。你可能也需要把这个权限禁用。
  * **Delete**: 使用这个权限，任何人只需要对象id，就可以删除表中的对象。除非你为表中对象设置了另外的ACL。
* **Add Fields**:** **Parse表的式是根据创建的对象自建的，这在开发过程中很方便，你不需要在后端做任何修改就能增加新字段；但是，当你的应用上线后，还要通过客户端增加新字段是很少见的，所以你应该在你的应用发布后禁用此权限。

对于上面的每一个操作，你可以授权个每一个用户，也可以锁定权限给一组角色和用户。比如，一个class表要对所有用户可用，应该被设置只读，只启用get和find。一个日志表应该被设置为只写，只允许创建。要审核用户生成的内容，你应该提供update和delete权限给指定用户组和角色组。

# 对象级的权限控制

当你锁定了你的模型和表级权限后，就该思考用户如何访问数据了。对象级的权限控制可以让一个用户的数据和其他用户保持分离，因为有时不同的对象需要被不同的人访问。比如这个场景，一个用户的私人数据应该只有他自己能够访问。

Parse还支持匿名用户的概念，所以你的应用可以保存和保护不需要登录的匿名用户的数据。

当一个用户登录你的应用后，Parse会为用户生成一个session，通过这个session，用户可以增加和修改他们的数据，但是不能修改其他人的数据。

_访问控制列表_

要控制哪些人可以访问数据，最简单的办法就是使用访问控制列表，即ACL，。ACL背后的思想就是为每一个对象增加一个具有某权限的用户/角色列表。一个用户需要某对象的读权限，或者这个用户在具有读权限的角色中，这个用户才可以拉取到对象的数据，同样的，一个用户具有对象的写权限，或者在具有写权限的角色中，才可以修改或删除对应的对象。

当你有了用户以后，你就可以开始使用ACL了。你要记住，用户可以通过传统用户名/密码方式注册，也可以使用Fackbook或twitter等第三方平台登录，甚至使用Parse的匿名用户功能。要为当前用户设置ACL，使其对公众不可读，你可以这样做：

```
var user = Parse.User.current();
user.setACL(new Parse.ACL(user));
```

很多应用都会这样，如果你要保存敏感的用户信息，比如邮箱地址和电话号码，你需要为用户设置一个ACL，以便保护用户的私人数据不会被其他人拿到，如果一个对象没有ACL，那么所有人都可以读写这个对象的数据，用户对象除外。我们永远不会允许用户修改其他用户的数据，但是默认用户可以读取其他用户的数据。如果开发者有更新其他用户对象的需求，可以使用master key完成。

如果你想让用户的部分数据公开，另一部分数据不公开，最好的办法就是分割成两个对象，在公开的对象上增加一个指向私密对象的指针字段。

```
var privateData = Parse.Object.extend("PrivateUserData");
privateData.setACL(new Parse.ACL(Parse.User.current()));
privateData.set("phoneNumber", "555-5309");

Parse.User.current().set("privateData", privateData);
```

当然，你也可以为对象设置不同的读写权限。比如，创建一个作者可写，公众可读的ACL：

```
var acl = new Parse.ACL();
acl.setPublicReadAccess(true);
acl.setWriteAccess(Parse.User.current().id, true);
```

有时基于每个用户的权限管理是不方便的，并且你想按特定权限给用户分组，就像设置管理员，使用角色功能就可以完成。角色是一种特殊的对象类型，它可以让你创建一个角色，通过ACL给角色指派权限。角色最方便的就是你无需为每个受角色约束的对象做修改，就可以为对象的用户权限做增删操作。要创建一个只有管理员可写的对象可以这样做：

```
var acl = new Parse.ACL();
acl.setPublicReadAccess(true);
acl.setRoleWriteAccess("admins", true);
```

当然，上面的代码是假设你已经创建了一个名为"admins"的角色，当你在开发应用时，你要创建一组特殊的角色，这将会很有用。角色还可以自由的创建和修改，比如这个场景，在两个用户建立连接后，为"friendOf\_\_"增加一个新的朋友。

这些仅仅是开始。应用可以通过ACL和表级权限执行所有种类复杂的权限控制。比如：

* 对于私人数据，只有自己可以读写。
* 消息面板上的文字，只有作者和管理员可以修改，公众只读。
* 对于只有开发者通过master key才可以访问的日志数据，ACL可以拒绝所有权限。
* 管理员创建的数据，比如每日消息，只有管理员可以修改，公众只读。
* 一个用户发送给另一个用户的数据，只有他们可以读写。

下面是一个公众可读，自己可写的ACL格式：

```
{
    "*": { "read":true },
    "UserObjectId": { "read" :true, "write": true }
}
```

下面是一个使用了角色的ACL格式：

```
{
    "role:RoleName": { "read": true },
    "UserObjectId": { "read": true, "write": true }
}
```

_指针类型权限_

指针类型是一种特殊的表级权限，它基于这些对象上的指针字段指向的用户，在class表中的每个对象上创建了一个虚拟ACL。比如，一个使用了owner字段的class表中，owner上设置了一个只读权限，class表中的每个对象将会对owner指向的用户只读。假设有一个有sender\(发送方\)和receiver\(接收方\)字段的class表，receiver字段上使用了只读权限，sender字段上使用了读写权限，那么receiver指向的用户对于这个表中所有的对象只读，sender指向的用户对于这个表中所有的用户可以读写。

需要注意的是这个ACL并没有真的在每个对象上创建，任何已经存在的ACL并不会因为你增删了指针类型权限而有任何修改，并且任何尝试和一个对象进行交互的用户都只能和这个对象进行交互，对象已经存在的ACL也会允许其操作。

有时将指针权限和ACL混合使用会造成混论，所以我们推荐你使用指针权限就不要使用ACL。幸运的是，如果你决定使用云代码或着ACL来保护你的应用后，移除指针权限是很简单的。

_查询权限验证\(需要parse-server &gt;= 2.3.0\)_

从2.3.0版本开始，parse-server引进了心的表级权限\(CLP\)requiresAuthentication，CLP会阻止任何未经验证的用户执行CLP保护的操作。

比如，你想允许经过验证的用户从你的云端find和get Announcement，你需要设置此CLP：

```
// POST http://my-parse-server.com/schemas/Announcement
// Set the X-Parse-Application-Id and X-Parse-Master-Key header
// body:
{
  classLevelPermissions:
  {
    "find": {
      "requiresAuthentication": true,
      "role:admin": true
    },
    "get": {
      "requiresAuthentication": true,
      "role:admin": true
    },
    "create": { "role:admin": true },
    "update": { "role:admin": true },
    "delete": { "role:admin": true }
  }
}
```

效果：

* 未经验证的用户将不能做任何事
* 经验证的用户将可以读取这个class表的所有对象
* 属于admin角色的用户将可以执行所有操作

提醒：如果你允许任何人登录你的应用，任何客户端仍然可以查找表中对象，以上将形同虚设。

_CLP和ACL的相互影响_

表级权限\(CLP，Class Level Permmissions\)和访问控制列表\(ACL，Access Control List\)都是安全防护的好工具，但它们并不总是如你期望的那样交互。它们实际上代表两种不同安全层，每个请求必须通过每一层后才能返回正确的信息或做预期的修改。下图展示了这些层，一个在lass表层，一个在对象层，请求必须经过这两层验证。请注意，尽管指针权限和ACL很相似，但是指针权限是表级权限的一种，所以请求为了经过CLP验证必须先经过指针权限验证。![](/assets/clp_vs_acl_diagram.png)就像你看到的，当你同时使用了CLP和ACL，是否授权用户发出请求就会变得复杂。让我们通过一个例子来更好的理解CLP和ACL可以如何交互。假设我们有一个Photo表，并且有一个photoObject对象，我们的应用中有两个用户，user1和user2，现在我们在Photo表上设置Get CLP，禁止公众Get的请求，允许user1的Get请求，然后再再photoObject上设置ACL，只允许user2的Get请求。

你可能期望这会使user1和user2都能Get PhotoObject，但是因为CLP层和ACL层同时生效，所以user1和user2将都不能请求成功。user1的Get请求会通过CLP层，但是无法通过ACL层，同样的，user2可以通过ACL层，但是不能通过CLP层。

现在我们在来看一个指针权限的例子。假设我们有一个Post表，里面有一个myPost对象，在我们的应用中有两个用户，poster和viewer，现在我们要增加一个指针权限使Creator字段中的任何人都可以读写这个对象，并且对于myPost对象，poster是这个字段中的用户，我们还在这个对象上增加了ACL，使view具备只读权限。你可能期望这会允许poster可以读取和修改myPost，但是viewer将会被指针权限拒绝，poster将会被ACL拒绝，所以，两个用户都无法访问这个对象。

介于CLP、ACL和指针权限间的复杂影响，我们建议你同时使用时要小心对待。使用CLP禁用某个类型的请求，然后其他请求使用指针权限或者ACL，这经常被用到。举个例子，你可能想禁用Photo表中的Delete权限，但另一方面又增加一个指针权限以便创建者可以编辑它，只是不允许删除它。因为指针权限和ACL之间的相互影响非常复杂，我们一般推荐你只用这两种安全机制中的一种。

