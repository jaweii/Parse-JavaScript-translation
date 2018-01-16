# 文件

## 创建一个Parse文件

`Parse.File`使你可以在云端存储文件，以免将大型数据存储在`Parse.Object`中，造成数据冗余。最常用的使用场景就是存储图片了，但是你也可以用它来存储文档、视频、音乐和任何二进制文件（最大支持10M）。

`Parse.File`的使用超级简单，你有两种方法可以创建一个文件。首先是使用base64编码字符串创建：

```js
var base64 = "V29ya2luZyBhdCBQYXJzZSBpcyBncmVhdCE=";
var file = new Parse.File("myfile.txt", { base64: base64 });
```

或者，你可以从字节内容的数组中创建一个文件：

```js
var bytes = [ 0xBE, 0xEF, 0xCA, 0xFE ];
var file = new Parse.File("myfile.txt", bytes);
```

Parse将会根据文件名自动检查文件类型，你也可以使用第三个参数`Content-Type`来指定文件类型：

```js
var file = new Parse.File("myfile.zzz", fileData, "image/png");
```

在H5应用中最常见的是，用html表单上传文件，这在现代化的浏览器中实现超级简单。创建一个input标签，使用户可以从本地设备选择文件上传：

```js
<input type="file" id="profilePhotoFileUpload">
```

然后，在某个点击事件中，获得引用的文件：

```js
var fileUploadControl = $("#profilePhotoFileUpload")[0];
if (fileUploadControl.files.length > 0) {
  var file = fileUploadControl.files[0];
  var name = "photo.jpg";

  var parseFile = new Parse.File(name, file);
}
```

注意在上面例子中，我们的文件名是photo.jpg，两处注意：

* 你不需要担心文件名冲突，因为上传的每个文件都会有一个唯一的id，所以上传n个photo.jpg的文件都不在话下。
* 给你上传的文件指定文件拓展名是很重要的，Parse会根据扩展名来对这个文件做相应的计算、处理。所以，如果你上传的是png图片，请确保你的文件名以.png结尾。

下一步就是存储文件到云端了。和Parse.Object一样，有多种保存文件的方法，你可以选择适合你的方法处理回调和错误：

```js
parseFile.save().then(function() {
  // The file has been saved to Parse.
}, function(error) {
  // The file either could not be read, or could not be saved to Parse.
});
```

最后，你可以用Parse.Object关联Parse.File，就像关联其他对象一样：

```js
var jobApplication = new Parse.Object("JobApplication");
jobApplication.set("applicantName", "Joe Smith");
jobApplication.set("applicantResumeFile", parseFile);
jobApplication.save();
```

---

## 获得文件

获得文件的最佳实践还要视你的应用而定。因为有跨域请求，所以最好先确保跨域能正常工作。典型的应用场景就是在页面上显示上传的图片，下面我们用jQuery在页面上显示上传的图片文件：

```js
var profilePhoto = profile.get("photoFile");
$("profileImg")[0].src = profilePhoto.url();
```

如果你想在云代码中处理文件数据，你可以用我们的http请求库来获取文件：

```js
Parse.Cloud.httpRequest({ url: profilePhoto.url() }).then(function(response) {
  // 文件数据在 response.buffer 中。
});
```

你可以使用[REST API](http://docs.parseplatform.org/rest/guide/#deleting-files)删除对象引用的文件，但是你需要提供master key才有权限删除文件。

如果你的文件没有被任何对象引用，那么不能通过REST API删除文件，你应该到应用设置页面清理未使用的文件。注意，删除后就不能再被访问到了。

