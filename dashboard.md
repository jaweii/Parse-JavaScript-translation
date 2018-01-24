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

然后你就可以使用`parse-dashboard --config parse-dashboard-config.json `命令来启动仪表盘了。

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



