# 坐标

Parse允许你在对象中关联真实世界的经纬坐标，使用`Parse.GeoPoint`，可以查询这个对象和参考位置的距离。这让你可以超级简单的实现基于坐标的功能，比如附近的人、最近的KTV。

---

## Parse.GeoPoint

要关联一个坐标到对象，你首先需要创建一个`Parse.GeoPoint`。假设你要创建一个纬度40.00、经度-30.00的坐标：

```js
var point = new Parse.GeoPoint({latitude: 40.0, longitude: -30.0});
```

这个位置将会作为一个普通的字段存储在对象中。

```js
placeObject.set("location", point);
```

---

## 坐标查询

现在假设，你有一大堆具有位置坐标的对象，要找出最靠近某个点的对象，可以通过给`Parse.Query`增加一个`near`条件，就能得到结果：

```js
// 用户位置
var userGeoPoint = userObject.get("location");
// 创建一个位置查询
var query = new Parse.Query(PlaceObject);
// 附近的用户，location为字段名
query.near("location", userGeoPoint);
// 限制返回数量
query.limit(10);
// 查询结果
query.find({
  success: function(placesObjects) {
  }
});
```

注意，查询结果`placesObjects`是根据距离从近到远排序的，除非你使用`ascending()`或者`descending()`设置了其他排序规则。

如果要根据距离过滤查询结果，可以使用`withinMiles`（英米内），`withinKilometers`（千米内）和`withinReadians`（弧度内）。

你可以查询特定区域内的对象。要查询距离范围内的对象，给`Parse.Query`增加`withinGeoBox`条件即可：

```js
var southwestOfSF = new Parse.GeoPoint(37.708813, -122.526398);
var northeastOfSF = new Parse.GeoPoint(37.822802, -122.373962);

var query = new Parse.Query(PizzaPlaceObject);
query.withinGeoBox("location", southwestOfSF, northeastOfSF);
query.find({
  success: function(pizzaPlacesInSF) {
    ...
  }
});
```

#### 

## 注意

有几个需要注意的地方：

1. 每一个`Parse.Object`类应当只有一个字段是`Parse.GeoPoint`对象；
2. `near`条件限制同时也会限制搜索结果在100英里内；
3. 坐标不能大于或等于最终范围，纬度应该在-90.00到90.00间，经度应在-180.00到180.00间。超过将造成error。



