## jquery 获取\<li>标签的value值

对于普通标签来说，可以使用jquery的`val()`方法获取属性，如：

```html
<input name="username" value="myname"/>
```

```js
var myname = $('input').val();
```

而对于`<li>`标签来说，如果value存放的不是数字，使用`val()`方法是无法获取到真正的值的， 如：

```html
<ul>
    <li value="myname">username</li>
</ul>
```

```js
var zero = $('li').val();	// 获取不到,结果为0
var myname = $('li').attr('value');	// 可以正确获取到myname值
```

**原因：**

HTML的li标签的value属性存在一下规定：规定列表项目为数字，所以它的value只能是数字。

