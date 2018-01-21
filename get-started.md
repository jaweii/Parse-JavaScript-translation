# 开始

Parse服务的后端搭建很简单，在本书的GitHub仓库中，有我上传的配置好的ParseServer项目，下载下来安装对应的依赖后运行即可。

然后在前端使用Parse的SDK即可与后端完成交互。

## 安装Parse服务

你可以照着[ParseServer文档](http://docs.parseplatform.org/parse-server/guide/) 自己搭建Parse服务。

也可以直接下载[配置好的Parse服务项目](https://github.com/jaweii/Parse-JavaScript-translation)，大概长这样：

![](/assets/1.png)

这个项目是基于官方的例子，集成了仪表盘和文件管理功能。

下载完这个项目后，安装依赖。你可以将目录移到合适的位置，因为它将作为我们的后端服务。

然后你还需要 [安装MongoDB](https://www.mongodb.com/download-center#community)，安装完成后使用`mongod -dbpath "你想要保存数据库的路径"` 运行数据库服务\(建议将数据库保存在项目目录\)，然后中 运行`mongo`命令测试能不能连接上：

```
$ mongo
MongoDB shell version: 3.2.9
connecting to: test
> ^C
bye
```

最后在项目目录运行 `npm  run start`即可开启Parse后端服务。

```
$ npm run start

> parse-server-example@1.4.0 start C:\Users\Administrator\Desktop\Parse\parse-server-example-master
> cross-env APP_ID='myAppId' CLIENT_KEY='123456' MASTER_KEY='123456' PORT=2018 SERVER_URL=http://localhost:2018/parse node index.js

DATABASE_URI not specified, falling back to localhost.
parse-server-example running on port 2018.
```

你可以通过访问 [http://localhost:2018/test](http://localhost:2018/test) 来测试是否运行成功。

如果运行成功，你可以访问[http://localhost:2018/dashboard](http://localhost:2018/dashboard) 进入仪表盘，进行可视化管理。

其中， [http://localhost:2018/parse](http://localhost:2018/parse) 就是我们的后端服务地址，在初始化SDK时我们将会用到。你也可 以通过修改index.js的代码来修改服务地址和其他服务信息，也可以在package.json中修改环境变量来修改服务信息。

# 集成Parse SDK

集成Parse SDK到你的JavaScript项目中，最简单的方法是通过[npm](https://npmjs.org/parse)安装。

但如果你想使用Parse的预编译文件，你可以从[npm cdn](https://npmcdn.com/)获得。

同时，你可以从[https://npmcdn.com/parse/dist/parse.js](https://npmcdn.com/parse/dist/parse.js) 获得开发版本，从[https://npmcdn.com/parse/dist/parse.min.js ](https://npmcdn.com/parse/dist/parse.min.js)获得压缩后的生产版本。

JavaScript的生态系统非常宽泛，它有着为数众多的框架和运行环境。为了适配各种情况，Parse的npm模块包含了为Node.js和React Native量身定制的SDK版本。最适合的才是最好的，所以，使用合适的npm包，可以确保诸如本地存储、用户会话、HTTP请求之类的项目使用合适的依赖。

在基于浏览器的应用中使用npm包，直接引入：

```js
var Parse = require('parse');
```

在后端应用或Node.js命令行工具中，引入`'parse/node'` ：

```js
// node.js
var Parse = require('parse/node');
```

在React Native应用中，引入`'parse/react-native'` ：

```js
// React Native
var Parse = require('parse/react-native');
```

接下来，用JavaScript初始化你的Parse-Sever，记得替换为你的应用信息：

```js
Parse.initialize("你的appId");
Parse.serverURL = '你的后端服务地址'
```

此JavaScript SDK 最初是基于广受欢迎的Backbone.js框架，它提供了灵活的API，允许你配置你喜欢的JS库。

我们的目标是最小化配置，以便你快速开始构建你的JavaScript和HTML5应用。

目前SDK支持Firefox 23+，Chrome 17+，Safari 5+和IE 10。 IE 9仅支持使用HTTPS托管的应用程序。

## 例子

在此例子中，假设你已经搭建好ParseServer。

我们使用Vue来做一个Todo List。

