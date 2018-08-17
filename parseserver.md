# ParseServer配置指南

> 番外部分不属于原版指南的一部分，由译者译制添加，以提供完整的体验。
>
> ParseServer GitHub地址：[https://github.com/parse-community/Parse-Server](https://github.com/parse-community/Parse-Server)

“开始”“一章有讲到，在本书GitHub有提供一个ParseServer脚手架项目，就是基于ParseServer + Express编写的，在本章会介绍更多关于ParseServer的配置，以满足开发者更多的需求。

比如，在“用户”一章有讲到，通过邮箱注册用户后，系统会自动发送一封邮件到指定邮箱，然后用户通过邮箱中的链接激活账号，但这是在使用Parse服务商的服务时才会自动发送邮件，自己搭建服务的话，这封邮件谁来发送呢？

这就要开发者自己来配置了。

### Parse Server + Express

你可以创建一个Parse实例后，挂载到Express站点上，就像刚才本书提供的ParseServer脚手架里面做的一样：

```js
var express = require('express');
var ParseServer = require('parse-server').ParseServer;
var app = express();

var api = new ParseServer({
  databaseURI: 'mongodb://localhost:27017/dev', // 你的Mongo数据库地址
  cloud: '/home/myApp/cloud/main.js',           // 云代码文件的路径，绝对路径
  appId: 'myAppId',                             // appId  
  masterKey: 'myMasterKey',                     // masterKey
  fileKey: 'optionalFileKey',                   // fileKey
  serverURL: 'http://localhost:1337/parse'      // 设置Parse服务地址
});

// 在/parse URL 前缀上提供Parse服务
app.use('/parse', api);

app.listen(1337, function() {
  console.log('parse-server-example running on port 1337.');
});
```

要查看所有可用的ParseServer 参数选项，可以运行`parse-server --help`查看。

## 配置

ParseServer可以通过下面的选项来配置，你可以在创建ParseServer实例的时候传入它们，也可以配置在JSON文件中用命令行加载`parse-server path/to/configuration.json`。

### 基本选项

* `appId`**\(必需\) **- 应用id。
* `masterKey`**\(必需\) **- 用来无视所有ACL/CLP权限，你应该妥善保管它。
* `databaseURI`**\(必需\) **- 你的数据库地址, i.e.`mongodb://user:pass@host.com/dbname`。如果你的密码有特殊字符，确保它 [URL encode 编码。](https://www.gitbook.com/book/jaweii/parse/edit#)
* `port`- 端口，默认1337。
* `serverURL`- ParseServer的URL，当云代码发送请求到ParseServer时会发送这个地址。
* `cloud`- 云代码文件的路径地址，是绝对路径。
* `push`- 配置APNS和GCM推送.参考[Push Notifications quick start](http://docs.parseplatform.org/parse-server/guide/#push-notifications_push-notifications-quick-start)。

### 客户端秘钥选项

下面密钥不是必需选项，如果设置了对应的选项，在对应的SDK初始化中需要传入你设置的秘钥。

* `clientKey`
* `javascriptKey`
* `restAPIKey`
* `dotNetKey`

### 高级选项

* `allowClientClassCreation`- 设置为false后禁止客户端创建Class表，默认为true。
* `enableAnonymousUsers`- 设置为false后禁用用户匿名功能，默认为true。
* `auth`- 用来配置支持[3rd party authentication](http://docs.parseplatform.org/parse-server/guide/#oauth-and-3rd-party-authentication)。
* `facebookAppIds`- Facebook应用Id数组。
* `mountPath`- 指定服务挂载的路由. 默认为`/parse`
* `filesAdapter`- 默认的文件管理可以通过适配器修改，参考[`FilesAdapter.js`](https://www.gitbook.com/book/jaweii/parse/edit#)。
* `maxUploadSize`- 最大上传文件限制，默认20M。
* `loggerAdapter`- 参考[`LoggerAdapter.js`](https://www.gitbook.com/book/jaweii/parse/edit#)。
* `logLevel`- 设置你想要的日志记录级别，默认为`info`，查看[Winston logging levels](https://github.com/winstonjs/winston#logging-levels) 支持哪些值。
* `sessionLength`- 设置session有效期，单位秒，默认31536000 seconds \(1 年\)。
* `maxLimit`- 限制查询的最大返回数量，默认不限制。
* `revokeSessionOnPasswordReset`- 当用户密码被重置，作废之前的session。
* `accountLockout`- 当用户尝试修改其他用户的信息时，锁定用户账号。
* `passwordPolicy`- 密码策略。
* `customPages`- 与邮箱验证链接、密码重置链接、面向用户的页面地址哈希，可用值有：`parseFrameURL`,`invalidLink`,`choosePassword`,`passwordResetSucces`,`verifyEmailSuccess`.
* `middleware`- \(CLI only\), 一个模块名，功能是express的中间件。 这个选项用来注入一个监听的中间件.
* `masterKeyIps`- 一个IP数组，masterKey的使用将被限定在这个数组范围内，默认为空，不限制。如果使用了这个选项，确保使用云代码时，你的IP包含在内。
* `readOnlyMasterKey`- 类似于masterKey，但是只有读权限，没有写权限。

## 邮箱验证和密码重置

使用email适配器，可以验证用户的邮箱地址和允许密码重置。下面以maigun为例，Mailgun适配器提供了发送邮件的功能，要使用这个，你需要注册Mailgun，并添加下面代码到你的项目：

```js
var server = ParseServer({
  ...otherOptions,
  // 启用邮箱验证
  verifyUserEmails: true,

  // 如果 `verifyUserEmails` 为 `true` 并且
  //     如果 `emailVerifyTokenValidityDuration` 为 `undefined` 那么
  //        email verify token 永不过期
  //     否则
  //        email verify token 在`emailVerifyTokenValidityDuration`之后过期
  //
  // `emailVerifyTokenValidityDuration` 默认为 `undefined`
  //
  // 2小时候过期 (= 2 * 60 * 60 == 7200 seconds)
  emailVerifyTokenValidityDuration: 2 * 60 * 60, //单位秒 (2 hours = 7200 seconds)

  // 设置为false，允许用户未验证也能登录
  // 设置为true，则必须验证才能登录
  preventLoginWithUnverifiedEmail: false, // 默认 false

  // 你的应用的URL
  // 将会出现在验证邮箱和密码重置的链接中
  // 就像设置serverURL一样
  publicServerURL: 'https://example.com/parse',
  // 你的应用名，将作为邮件主题
  appName: 'Parse App',
  // email适配器
  emailAdapter: {
    module: '@parse/simple-mailgun-adapter',
    options: {
      // 设置发件地址
      fromAddress: 'parse@example.com',
      // 你的mailgun域名
      domain: 'example.com',
      // 你的mailgun apiKey
      apiKey: 'key-mykey',
    }
  },

  // 账户停用策略 (可选) - 默认为undefined
  /*
  如果设置了帐户锁定策略，并且登录尝试次数超过了“阈值”次数，
  则“login”api调用将返回错误代码“Parse.Error.OBJECT_NOT_FOUND”，
  并显示错误消息“您的帐户由于多次登录失败而被锁定尝试。 请在<时间>分钟后再试一次。 
  在没有登录尝试的“持续时间”分钟之后，应用程序将允许用户再次尝试登录。
  */
  accountLockout: {
    duration: 5, // 账户锁定等待时长，等位分钟，设置为1-100000.
    threshold: 3, // 设置登录错误次数阈值，1-1000
  },
  // 可选的密码策略
  passwordPolicy: {
    //两个可选设置来执行强密码。 可以指定一个或两个。
    //如果两者都被指定，则两个检查都必须通过才能接受密码
    // 1.表示要执行的模式的RegExp对象或正则表达式字符串
    validatorPattern: /^(?=.*[a-z])(?=.*[A-Z])(?=.*[0-9])(?=.{8,})/, // 最少8为，大小写和数字组成
    // 2. 在回调中使用验证方法验证
    validatorCallback: (password) => { return validatePassword(password) }, 
    doNotAllowUsername: true, // 是否允许账号密码相同
    maxPasswordAge: 90, // 设置密码过期天数
    maxPasswordHistory: 5, // 禁用前n个密码再次使用
     //设置密码重置链接超时时间
    resetTokenValidityDuration: 24*60*60, //24小时
  }
});
```

你还可以安装来自社区的其他邮件适配器：

* [parse-server-postmark-adapter](https://www.npmjs.com/package/parse-server-postmark-adapter)
* [parse-server-sendgrid-adapter](https://www.npmjs.com/package/parse-server-sendgrid-adapter)
* [parse-server-mandrill-adapter](https://www.npmjs.com/package/parse-server-mandrill-adapter)
* [parse-server-simple-ses-adapter](https://www.npmjs.com/package/parse-server-simple-ses-adapter)
* [parse-server-mailgun-adapter-template](https://www.npmjs.com/package/parse-server-mailgun-adapter-template)
* [parse-server-sendinblue-adapter](https://www.npmjs.com/package/parse-server-sendinblue-adapter)
* [parse-server-mailjet-adapter](https://www.npmjs.com/package/parse-server-mailjet-adapter)
* [simple-parse-smtp-adapter](https://www.npmjs.com/package/simple-parse-smtp-adapter)
* [parse-server-generic-email-adapter](https://www.npmjs.com/package/parse-server-generic-email-adapter)

在本书提供的ParseServer脚手架中已经配置好了邮件服务，使用的是译者专门编写的适配器parse-zh-email-adapter，你只需要换上你的SMTP账号就行了。

但是还有些地方要你自己修改，比如邮箱验证成功的页面：

![](/assets/email.png)

英文界面，对用户显然不够友好，如果要修改成理想的界面，编辑目录`./node_modules/parse-server/public_html/`下的模板即可。

## 日志

ParseServer日志默认会：

* 在控制台打印。
* 将每天的日志保存到新文件中。

日志在Parse仪表盘中也可以查看。

### 记录每个请求

在运ParseServer服务时，将环境变量`VERBOSE`设置为'1'即可。比如：

`VERBOSE='1' parse-server --appId APPLICATION_ID --masterKey MASTER_KEY`

### 设置日志目录

将环境变量`PARSE_SERVER_LOGS_FOLDER`设置为新的路径即可：

`PARSE_SERVER_LOGS_FOLDER='<path-to-logs-folder>' parse-server --appId APPLICATION_ID --masterKey MASTER_KEY`

### 设置日志级别

通过环境变量`logLevel`设置：

`parse-server --appId APPLICATION_ID --masterKey MASTER_KEY --logLevel LOG_LEVEL`

