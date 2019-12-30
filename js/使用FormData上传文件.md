## 使用FormData上传文件

### form标签上传文件

```html
<form action="上传文件的后端api接口" enctype="multipart/form-data">
    <!-- 上传额外的参数 -->
    <input type="text" name="filename"/>
    <!-- 上传的文件 -->
    <input type="file" name="file"/>
    <input type="submit" value="提交"/>
</form>
```

使用常规的form表单方式可以实现文件上传，但是当用户点击提交按钮时，页面也会跟随着跳转，用户体验很差。

### FromData异步上传

`FormData`是js提供的用于手动组装from表单的数据，为form数据的提交提供了更加灵活的方式。

#### 初始化FromData

```html
<form id="myfrom" enctype="multipart/form-data">
    <!-- 上传额外的参数 -->
    <input type="text" name="filename"/>
    <!-- 上传的文件 -->
    <input id="id_file" type="file" name="file"/>
</form>
```

```js
// 获取form表单对象
var formObj = document.getElementById('myform');
// 通过form表单构造FromData对象
var formData = new FormData(formObj);
```

#### 添加属性

```js
// 获取文件
var file = $('#id_file')[0].files[0];
// 文件名
var filename = file.name;
// 直接构建FormData
var formData = new FormData();
// 添加上传的文件
formData.append('file', file);
// 添加其他属性
formData.append('serverId', 'myServerId');
```

### 使用jquery上传

```js
$.ajax({
    url: '后端提供的上传url',
    data: formData,   // 使用上面构建好的FromData对象
    type: 'post',
    cache: false,	// 上传文件不需要缓存
    processData: false,	// 用于对data参数进行序列化，必须为false
    contentType: false,	// 必须为false
    success: function(data) {
        // 处理成功后的回调函数
    },
    error: function(data) {
        // 处理失败后的回调函数
    }
})
```

