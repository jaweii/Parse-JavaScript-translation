# 仪表盘配置指南

Parse仪表盘是一个单独的应用，用来管理Parse Server 应用。[Parse仪表盘GitHub地址](https://github.com/parse-community/parse-dashboard)。

## 安装

```js
npm install -g parse-dashboard
```

你可以通过在命令行中提供app ID、master key、URL、appName来启动仪表盘：

```
parse-dashboard --appId yourAppId --masterKey yourMasterKey --serverURL "https://example.com/parse" --appName optionalName
```

你还可以通过--host、--port、--mountPath来指定主机、端口和挂载路径，你可以使用任何值作为appName，如果没有指定，默认将会使用appID。

启动完成后，你可以通过[http://localhost:4040](http://localhost:4040/)访问 仪表盘。

![](/assets/dash-shot.png)

## 配置

### 文件

你也可以在命令行中使用配置文件来启动仪表盘，只需要在你的仪表盘项目目录创建一个parse-dashboard-config.json的文件，并符合下面的格式：

```js
{
  "apps": [
    {
      "serverURL": "http://localhost:1337/parse",
      "appId": "myAppId",
      "masterKey": "myMasterKey",
      "appName": "MyApp"
    }
  ]
}
```

然后你就可以使用`parse-dashboard --config parse-dashboard-config.json`命令来启动仪表盘了。

### 环境变量

这只在parse-dashboard命令行中可用。

有两种方法可以让你配置仪表盘的环境变量。

#### 多应用

在`PARSE_DASHBOARD_CONFIG` 中提供完整的JSON配置，它会像配置文件一样被解析。

#### 单应用

你也可以单独的定义每一个配置项：

```
HOST: "0.0.0.0"
PORT: "4040"
MOUNT_PATH: "/"
PARSE_DASHBOARD_TRUST_PROXY: undefined // Or "1" to trust connection info from a proxy's X-Forwarded-* headers
PARSE_DASHBOARD_SERVER_URL: "http://localhost:1337/parse"
PARSE_DASHBOARD_MASTER_KEY: "myMasterKey"
PARSE_DASHBOARD_APP_ID: "myAppId"
PARSE_DASHBOARD_APP_NAME: "MyApp"
PARSE_DASHBOARD_USER_ID: "user1"
PARSE_DASHBOARD_USER_PASSWORD: "pass"
PARSE_DASHBOARD_SSL_KEY: "sslKey"
PARSE_DASHBOARD_SSL_CERT: "sslCert"
PARSE_DASHBOARD_CONFIG: undefined // Only for reference, it must not exist
```

## 管理多个应用

在一个仪表盘中管理多个应用也是可以的，只需要在配置项`apps`数组中增加额外的应用配置即可。

```
{
  "apps": [
    {
      "serverURL": "https://api.parse.com/1", // Hosted on Parse.com
      "appId": "myAppId",
      "masterKey": "myMasterKey",
      "javascriptKey": "myJavascriptKey",
      "restKey": "myRestKey",
      "appName": "My Parse.Com App",
      "production": true
    },
    {
      "serverURL": "http://localhost:1337/parse", // Self-hosted Parse Server
      "appId": "myAppId",
      "masterKey": "myMasterKey",
      "appName": "My Parse Server App"
    }
  ]
}
```

## 配置应用图标

Parse仪表盘可以为每个应用可选的配置图标，以便你在仪表盘列表中快速识别它们。

你需要在配置项中定义`iconsFolder`，并且为每个app配置定义`iconName`，`iconsFolder`的路径相对配置文件的，如果你的Parse仪表盘是安装在全局的，你需要为`iconsFolder`指定完整路径。为了直观理解，下面例子中`icons`的所在位置是和配置文件一样的：

```
{
  "apps": [
    {
      "serverURL": "http://localhost:1337/parse",
      "appId": "myAppId",
      "masterKey": "myMasterKey",
      "appName": "My Parse Server App",
      "iconName": "MyAppIcon.png",
    }
  ],
  "iconsFolder": "icons"
}
```

## 作为Express中间件运行

你可以将Parse仪表盘作为Express的中间件运行：

```js
var express = require('express');
var ParseDashboard = require('parse-dashboard');

var dashboard = new ParseDashboard({
  "apps": [
    {
      "serverURL": "http://localhost:1337/parse",
      "appId": "myAppId",
      "masterKey": "myMasterKey",
      "appName": "MyApp"
    }
  ]
});

var app = express();

// make the Parse Dashboard available at /dashboard
app.use('/dashboard', dashboard);

var httpServer = require('http').createServer(app);
httpServer.listen(4040);
```

如果你想将ParseServer和Parse仪表盘运行在同一个服务上，可以将他们作为express中间件运行：

```js
var express = require('express');
var ParseServer = require('parse-server').ParseServer;
var ParseDashboard = require('parse-dashboard');

var api = new ParseServer({
    // Parse Server settings
});

var options = { allowInsecureHTTP: false };

var dashboard = new ParseDashboard({
    // Parse Dashboard settings
}, options);

var app = express();

// make the Parse Server available at /parse
app.use('/parse', api);

// make the Parse Dashboard available at /dashboard
app.use('/dashboard', dashboard);

var httpServer = require('http').createServer(app);
httpServer.listen(4040);
```

## 部署仪表盘

### 安全

为了安全地部署仪表盘，防止你应用的master key泄露，你需要使用HTTPS或基本的身份验证。

部署后的仪表盘会检查你使用的连接是否安全，如果你在负载均衡或前置代理后部署仪表盘，那么仪表盘将不会检查连接是否安全，这种情况下，你可以使用--strstProxy选项来启动仪表盘，以依赖客户端安全连接的头部X-Forwarded-\* 。这对于Heroku这样的托管服务很有用，这样你可以信任提供的代理头来判断你使用的是HTTP或HTTPS。你也可以在仪表盘作为express中间件使用的时候设置这个选项。

```js
var trustProxy = true;
var dashboard = new ParseDashboard({
  "apps": [
    {
      "serverURL": "http://localhost:1337/parse",
      "appId": "myAppId",
      "masterKey": "myMasterKey",
      "appName": "MyApp"
    }
  ],
  "trustProxy": 1
});
```

### 身份验证

你可以通过添加用户名和密码到配置项中，来为仪表盘添加基本的身份验证：

```js
{
  "apps": [{"...": "..."}],
  "users": [
    {
      "user":"user1",
      "pass":"pass"
    },
    {
      "user":"user2",
      "pass":"pass"
    }
  ],
  "useEncryptedPasswords": true | false
}
```

你可以将密码明文或bcrypt加密存储，如果是使用bcrypt加密存储，你需要配置`useEncryptedPassword`参数为`true`。你可以使用任何bcrypt在线加密工具为你的密码加密，比如：https://www.bcrypt-generator.com。

### 基于用户的访问权限

如果你为仪表盘配置了多应用管理，你可以基于用户来限制应用的管理。像这样：

```js
{
  "apps": [{"...": "..."}],
  "users": [
     {
       "user":"user1",
       "pass":"pass1",
       "apps": [{"appId": "myAppId1"}, {"appId": "myAppId2"}]
     },
     {
       "user":"user2",
       "pass":"pass2",
       "apps": [{"appId": "myAppId1"}]
     }  ]
}
```

这样配置后，user1可以管理myAppId1和myAppId2，user2只能管理myAppId1。

## 使用只读masterKey

你可以通过配置readOnlyMasterKey来限制在仪表盘对对象的写操作，readOnlyMasterKey和masterKey很相似，只不过没有写权限。

### 所用用户使用只读masterKey

在ParseServer中配置如下：

```
{
"masterKey": "YOUR_MASTER_KEY_HERE",
"readOnlyMasterKey": "YOUR_READ_ONLY_MASTER_KEY",
}
```

在仪表盘中配置如下：

```js
var trustProxy = true;
var dashboard = new ParseDashboard({
  "apps": [
    {
      "serverURL": "http://localhost:1337/parse",
      "appId": "myAppId",
      "masterKey": "YOUR_READ_ONLY_MASTER_KEY",
      "appName": "MyApp"
    }
  ],
  "trustProxy": 1
});
```

### 部分用户使用只读masterKey

使用`readOnly`将用户标记为只读用户：

```js
{
  "apps": [
    {
      "appId": "myAppId1",
      "masterKey": "myMasterKey1",
      "readOnlyMasterKey": "myReadOnlyMasterKey1",
      "serverURL": "myURL1",      
      "port": 4040,
      "production": true
    },
    {
      "appId": "myAppId2",
      "masterKey": "myMasterKey2",
      "readOnlyMasterKey": "myReadOnlyMasterKey2",
      "serverURL": "myURL2",      
      "port": 4041,
      "production": true
    }
  ],
  "users": [
    {
      "user":"user1",
      "pass":"pass1",
      "readOnly": true,
      "apps": [{"appId": "myAppId1"}, {"appId": "myAppId2"}]
    },
    {
      "user":"user2",
      "pass":"pass2",
      "apps": [{"appId": "myAppId1"}]
    }
  ]
}
```

user1将只有读操作的权限。

### 指定应用只读

在每个用户的apps配置项中，你可以基于每个应用的给定只读权限：

```js
{
  "apps": [
    {
      "appId": "myAppId1",
      "masterKey": "myMasterKey1",
      "readOnlyMasterKey": "myReadOnlyMasterKey1",
      "serverURL": "myURL",      
      "port": 4040,
      "production": true
    },
    {"...": "..."}
  ],
  "users": [
    {
      "user":"user",
      "pass":"pass",
      "apps": [{"appId": "myAppId", "readOnly": true}, {"appId": "myAppId2"}]
    }
  ]
}
```

user将会myAppId只有读操作的权限。







