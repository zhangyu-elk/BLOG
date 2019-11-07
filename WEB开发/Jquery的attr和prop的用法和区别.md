# Jquery中的`attr`和`prop`的区别
> 测试唤醒: Google

## `attr`和`prop`的作用和区别

&emsp;&emsp;`attr`和`prop`都是Jquery对象用于设置或者获取DOM元素的函数, 比如操作`disabled`控制按钮是否可用; 最近再操作`checkbox`时发现, 使用`attr`无法生效只能使用后者. 依赖于getAttribute()和setAttribute()两个函数.

## `attr`

> `attribute`指明的是HTML文档节点属性;

```
<input type="submit" value="百度一下" id="su" class="btn self-btn bg s_btn">
```
这里的`type`/`value`/`id`等, 均是DOM节点的属性

**使用`attr`获取属性**
```
var b = $("#su")

b.attr("type")
"submit"
b.attr("value")
"百度一下"
b.attr("id")
"su"
b.attr("class")
"btn self-btn bg s_btn"
b.attr("className")
undefined
```

**注意**:不同的Jquery版本中的这些值, 可能返回的类型都会不同, 可能返回`true/false`也可能返回`checked/selected`等. 笔者没详细测试, 但是如果遇到问题最后自行测试一下返回结果比较好.

## `prop`
> `prop`可以获取的是DOM节点对应的JS对象的属性, 我们可以使用诸如`document.getElementById("su")`获取一个JS对象, DOM节点上的属性再这个对象上未必有, 这个对象上的属性也未必在DOM节点上显示的说明.

```
var a = document.getElementById("su")

a.className
"btn self-btn bg s_btn"

$(a).prop("className")
"btn self-btn bg s_btn"
```
这里的`className`就是JS对象的属性, 在DOM节点上对应的是`class`(没有`className`这个东西)

**注意**: 对于之前说明的如`checked`实际上是与JQuery的版本有关, 不过建议使用`prop`; 不过有兴趣可以测试一下发现, 使用`attr`或者直接操作`JS`对象的`attr`属性, 并不会影响DOM节点的代码(可以猜到`checked`应当是与JS对象关联的); 而修改`disabled`属性会发现HTML标签中显示的添加了该属性(JS的属性和DOM节点的属性是一致的).



## 扩展`data`
> HTML标签可以使用`data-key='value'`的方式自定义数据
```
$("#test").data("key")
```
可以获取该自定义的属性值