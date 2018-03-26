# 开始

Parse服务的后端搭建很简单，在本书的[GitHub](https://github.com/jaweii/Parse-JavaScript-translation)仓库中，有译者上传的配置好的ParseServer项目，下载下来安装对应的依赖后运行即可。

然后，在前端使用Parse的SDK即可与后端完成交互。

## 安装Parse服务

你可以照着[ParseServer文档](http://docs.parseplatform.org/parse-server/guide/) 自己搭建Parse服务，也可以直接下载[配置好的Parse服务项目](https://github.com/jaweii/Parse-JavaScript-translation)，大概长这样：

![](/assets/1.png)

这个项目是基于官方的例子，集成了仪表盘和文件管理功能。

下载完这个项目后，安装依赖。你可以将目录移到合适的位置，因为它将作为我们的后端服务。

然后你还需要 [安装MongoDB](https://www.mongodb.com/download-center#community)，安装完成后使用`mongod -dbpath "你想要保存数据库的路径"` 命令运行数据库服务，然后运行`mongo`命令测试能不能连接上：

```
$ mongo
MongoDB shell version: 3.2.9
connecting to: test
> ^C
bye
```

如果连接成功，在项目目录运行 `npm  run start`即可开启Parse后端服务。

```
$ npm run start

> parse-server-example@1.4.0 start C:\Users\Administrator\Desktop\Parse\parse-server-example-master
> cross-env APP_ID='myAppId' CLIENT_KEY='123456' MASTER_KEY='123456' PORT=2018 SERVER_URL=http://localhost:2018/parse node index.js

DATABASE_URI not specified, falling back to localhost.
parse-server-example running on port 2018.
```

你可以通过访问 [http://localhost:2018/test](http://localhost:2018/test) 来测试是否运行成功。

如果运行成功，你可以访问[http://localhost:2018/dashboard](http://localhost:2018/dashboard) 进入仪表盘，进行可视化管理。

其中， [http://localhost:2018/parse](http://localhost:2018/parse) 就是我们的后端服务地址，在初始化SDK时我们将会用到。你也可以通过修改index.js的代码来修改服务地址和其他服务信息，也可以在package.json中修改环境变量来修改服务信息。

## 集成Parse SDK

集成Parse SDK到你的JavaScript项目中，最简单的方法是通过[npm](https://npmjs.org/parse)安装。

但如果你想使用Parse的预编译文件，你可以从[npm cdn](https://npmcdn.com/)获得。

同时，你可以从[https://npmcdn.com/parse/dist/parse.js](https://npmcdn.com/parse/dist/parse.js) 获得开发版本，从[https://npmcdn.com/parse/dist/parse.min.js ](https://npmcdn.com/parse/dist/parse.min.js)获得压缩后的生产版本。

JavaScript的生态系统非常宽泛，它有着为数众多的框架和运行环境。为了适配各种情况，Parse的npm模块包含了为Node.js和React Native量身定制的SDK版本。最适合的才是最好的，所以，使用合适的npm包，可以确保诸如本地存储、用户会话、HTTP请求之类的项目使用合适的依赖。

在基于浏览器的应用中使用npm包：

```js
var Parse = require('parse');
```

在后端应用或Node.js命令行工具中 ：

```js
// node.js
var Parse = require('parse/node');
```

在React Native应用中 ：

```js
// React Native
var Parse = require('parse/react-native');
```

接下来，用JavaScript初始化你的Parse-Sever，记得替换为你的应用信息：

```js
Parse.initialize("你的appId","javascript key","master key");//参数2、3选填。
Parse.serverURL = '你的后端服务地址'
```

SDK 基于Backbone.js框架，它提供了灵活的API，允许你配置你喜欢的JS库。

目前SDK支持Firefox 23+，Chrome 17+，Safari 5+和IE 10。 IE 9仅支持使用HTTPS托管的应用程序。

## 例子

在此例子中，假设你已经搭建好ParseServer。

现在，我们使用Vue来做一个Todo List。

1、使用脚手架命令`vue init webpack todolist`创建一个vue项目。

2、安装Parse、UI依赖 ，集成到main.js。

```js
// main.js
import Vue from 'vue'
import App from './App'
import router from './router'
import muse from 'muse-uI'
import 'muse-ui/dist/muse-ui.css'

Vue.use(muse)

Vue.config.productionTip = false

/* eslint-disable no-new */
new Vue({
    el: '#app',
    router,
    components: { App },
    template: '<App/>'
})
```

3、编写页面。

```js
//app.vue
<template>
    <div id="app">
        <mu-appbar>
            <mu-text-field type="text" hintText="Todo List" v-model="value"/>
            <mu-icon-button icon="add" slot="right" @click="addItem"/>
        </mu-appbar>
        <mu-list>
            <mu-list-item v-for="item in list" :title="item.get('title')" :describeText="item.get('finish')?'完成':'未完成'">
                <mu-icon-button icon="check" slot="left" @click="finishItem(item)" :disabled="item.get('finish')"/>
                <mu-icon-button icon="delete" slot="right" @click="deleteItem(item)"/>
            </mu-list-item>
        </mu-list>
        <div style="text-align:center;" v-if="!list.length">暂无任务</div>
        </section>
    </div>
</template>
<script>
//初始化SDK
import parse from 'parse'
parse.serverURL = 'http://localhost:2018/parse'
parse.initialize('myAppId', '123456')//传入APPID和JavaScript key即可

export default {
    name: 'App',
    data() {
        return {
            value: '',
            list: []
        }
    },
    mounted() {
        const query = new parse.Query('Todo')
        query.find().then(list => {
            this.list = list
        })
    },
    methods: {
        addItem() {
            if (!this.value)
                return
            const todo = new parse.Object('Todo')
            todo.set('title', this.value)
            todo.set('finish', false)
            todo.save().then(todo => {
                this.list.push(todo)
                this.value = ''
            }).catch(console.error)
        },
        finishItem(todo) {
            todo.set('finish', true).save().then(todo => {
                this.$forceUpdate()
            }).catch(console.error)
        },
        deleteItem(todo) {
            todo.destroy().then(todo => {
                this.list = this.list.filter(item => item !== todo)
            }).catch(console.error)
        }
    }
}
</script>

<style>
@import "http://cdn.bootcss.com/material-design-icons/3.0.1/iconfont/material-icons.css";
#app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    color: #2c3e50;
    max-width: 400px;
    left: 0;
    right: 0;
    margin: auto;
}
</style>
```

以上，是手写的所有代码。按惯例，上GIF：

![](/assets/GIF.gif)

图中的数据都是同步保存到后端数据库的，我们从仪表盘的数据浏览器可以看到：

![](/assets/dashboard.png)

你看，一个任务清单应用分分钟就完成了。

你可能对例子中SDK的用法不是很了解，如果你用过国内的leancloud、bmob等bass平台，你会发现他们超级相似；没有用过也没关系，本章之后的指南基本上都是讲SDK的使用。

另外，例子会上传到仓库中，有需要可以下载。

接下来，开始你的Parse之旅吧，有问题可以点击左上角“New Issue”发起讨论。

