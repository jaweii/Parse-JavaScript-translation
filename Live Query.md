# 实时请求

> 译者注：介绍下实时请求的使用场景，比如，某商品的扫码付款页面，在用户付款后，服务端确认付款成功后通知客户端成功付款的事件，客户端收到事件，知道用户付款成功，然后做对应的交互，完成付款。

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

### OPEN 事件

```js
subscription.on('open', () => {
 console.log('subscription opened');
});
```

当你调用`query.subscribe()`，我们发送了一个订阅请求到实时请求服务器，当我们从实时请求服务器的得到确认消息，就会触发open事件。

当客户端关闭WebSocket连接后，我们会自动重连到实时请求服务器，如果重连成功，也会触发open事件。

### CREAT事件

```js
subscription.on('create', (object) => {
  console.log('object created');
});
```

当一个新的Parse对象被创建，并且它满足你订阅的`ParseQuery`时，将会触发create事件。`object`是指创建的对象。

### UPDATE事件

```js
subscription.on('update', (object) => {
  console.log('object updated');
});
```

当一个已经存在的，满足你订阅的`ParseQuery`的`ParseObject`被更新，将会触发update事件。object是指更新后的ParseObject对象。

### ENTER事件

```js
subscription.on('enter', (object) => {
  console.log('object entered');
});
```

当一个已存在的，不满足`ParseQuery`的`ParseObject`变得满足后，就会触发enter事件。object是指变更后的最新对象。

### LEAVE事件

```js
subscription.on('leave', (object) => {
  console.log('object left');
});
```

和enter事件相反，当一个已存在的`ParseObject`对象从满足变成不满足`ParseQuery`后，就会触发leave事件。

### DELETE 事件

```js
subscription.on('delete', (object) => {
  console.log('object deleted');
});
```

当一个已存在，满足`ParseQuery`的`ParseObject`对象被删除后，就会触发delete事件。object对象是指被删除的对象。

### CLOSE 事件

```js
subscription.on('close', () => {
  console.log('subscription closed');
});
```

当客户端断开和LiveQuery服务的WebSocket连接后，就会触发close事件。

## 取消订阅 {#unsubscribe}

```js
subscription.unsubscribe();
```

如果你想停止从`ParseQuery`接收事件，你可以使用`unsubscribe`方法取消订阅。然后，你将不能从`subscription`收到任何事件。

# 关闭

```js
Parse.LiveQuery.close();
```

当你使用完实时请求，你可以调用close方法，WebSocket将会断开和实时请求服务器的链接，取消重连，取消订阅所有的订阅。在此之后，如果你再次调用`query.subscribe()`，将会创建一个新的连接。

## 设置服务地址 {#setup-server-url}

```js
Parse.liveQueryServerURL = 'ws://XXXX'
```

大多数的情况你不需要设置这个，如果你设置了服务地址，我们将会提取出主机地址，并使用ws://hostname作为默认的实时请求服务地址。总之，如果你想定义你自己的实时请求地址，或使用不同的协议，比如wss，你可以通过这个来设置。

## WebSocket 状态 {#websocket-status}

我们暴露出了3个方法来为你监控WebSocket链接的状态。

### OPEN 事件 {#open-event-1}

```js
Parse.LiveQuery.on('open', () => {
  console.log('socket connection established');
});
```

当WebSocket完成了连接到实时请求服务器，将会触发open事件。

### CLOSE 事件 {#close-event-1}

```js
Parse.LiveQuery.on('close', () => {
  console.log('socket connection closed');
});
```

当WebSocket关闭了和实时请求服务器的链接，将会触发close事件。

### ERROR 事件 {#error-event}

```js
Parse.LiveQuery.on('error', (error) => {
  console.log(error);
});
```

当出现某个网络错误，或者实时请求服务器发生错误，将会触发error事件。

## 重连 {#reconnect}

由于实时请求功能依赖WebSocket连接，为了保持WebScoket连接是一直打开的，每当我们发现连接丢失后，就会自动尝试重连，我们会在后台按间隔指数级增加的时间重连。不过，如果WebScoket连接时通过`Parse.LiveQuery.close()`或`client.close()`关闭的，将不会重连。

## SessionToken {#sessiontoken}

当你订阅一个ParseQuery，我们会发送一个当前用户的sessionToken到实时请求服务器。有一点要注意，当你退出当前用户后，sessionToken将失效，你应该取消订阅后重新订阅；否则，你可能会面临安全问题，因为你会收到不应发送给你的事件。













