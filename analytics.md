# 分析

> 译者注：此功能在Parse未关闭服务前，在Parse.com上可用；现在仪表盘上已经移除此功能，并建议用户使用第三方分析服务，所以本章可跳过。

Parse提供了一些钩子来帮助你分析你应用的一些数据。你知道，了解你的app在做什么，和谁做的，什么时候做的，做的频率，做了多久，嗯，是很重要的。

虽然本节会介绍不同的方法，来最大化利用Parse的分析服务，但开发者也可以使用Parse仪表盘上已经可以使用的指标来分析。

在Parse仪表盘，不需要执行任何客户端逻辑，你就可以实时看到你应用的API请求统计图表（有设备类型、Parse类名、REST请求），并且可以筛选你感兴趣的数据。

---

## 常用分析

`Parse.Analytics`允许你跟踪自定义事件，只需要使用一些string类型的键值即可。这些自定义的指标将可以在你应用的仪表盘上看到。

假设你的应用提供了公寓列表搜索的功能，并且你想跟踪这个功能的使用情况。下面我们来跟踪三个维度的搜索数据：

```js
var dimensions = {
  // 定义价格范围
  priceRange: '1000-1500',
  // 用户是否使用了筛选功能
  source: 'craigslist',
  // 这次搜索发生在工作日还是周末
  dayType: 'weekday'
};
// 发送搜索事件及数据到Parse后端
Parse.Analytics.track('search', dimensions);
```

`Parse.Analytics`甚至可以用于异常跟踪，只需要使用下面的代码，就可以为你的应用记录错误频率、错误信息。

```
var codeString = '' + error.code;
Parse.Analytics.track('error', { code: codeString });
```

注意目前Parse每次调用`Parse.Analytics.track()`存储指标，最多只有前8个维度数据会被保存。

