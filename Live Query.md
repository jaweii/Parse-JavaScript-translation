# 实时请求

## 标准API {#standard-api}

就像我们在[LiveQuery protocol](https://github.com/parse-community/parse-server/wiki/Parse-LiveQuery-Protocol-Specification)里面说到的，我们保留了一个WebSocket连接来与Parse实时请求服务通信。当实时请求在服务端使用时，使用的是[`ws`](https://www.npmjs.com/package/ws)包，当在浏览器端使用时，使用的是window.WebSocket。我们考虑到在大多数使用场景下，WebSocket不是必需的，所以我们提供了简单的API让你可以专注于你的业务逻辑。

提示：实时请求只在[JS SDK ~1.8](https://www.gitbook.com/book/jaweii/parse/edit#)版本以上支持。

## 创建订阅 {#create-a-subscription}

```js
let query = new Parse.Query('Game');
let subscription = query.subscribe();
```

你创建的`subscription`实际上是一个事件触发器，关于事件触发器的更多信息，[查看这里](https://www.gitbook.com/book/jaweii/parse/edit#)。你将会通过`subscription`拿到实时请求事件，我们会为你打开WebSocket连接到实时请求服务。

## 事件处理 {#event-handling}

我们定义了一些事件类型，你可以通过`subscription`对象拿到：

_OPEN 事件_

```js
subscription.on('open', () => {
 console.log('subscription opened');
});
```

当你调用`query.subscribe()`，我们发送了一个订阅请求到实时请求服务器，当我们从实时请求服务器的得到确认消息，就会触发open事件。

当客户端关闭WebSocket连接后，我们会自动重连到实时请求服务器，如果重连成功，也会触发open事件。



