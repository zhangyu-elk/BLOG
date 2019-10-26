# CSS3中transition的触发方式
> 最近在工作中需要用到`transition`, 但是发现其触发方式很复杂

## `transition`
&emsp;&emsp;`transition`是CSS3最简单的动画, 当元素的属性发生改变能够以渐变的方式呈现出来; 如下代码是`w3c`上的一个示例, 加上了`transition`的结果就是在`hover`时, 长度会逐步增加到300px.

```
<!DOCTYPE html>
<html>
<head>
<style> 
div
{
width:100px;
height:100px;
background:blue;
transition:width 2s;
-moz-transition:width 2s; /* Firefox 4 */
-webkit-transition:width 2s; /* Safari and Chrome */
-o-transition:width 2s; /* Opera */
}

div:hover
{
width:300px;
}
</style>
</head>
<body>

<div></div>

<p>请把鼠标指针移动到蓝色的 div 元素上，就可以看到过渡效果。</p>

<p><b>注释：</b>本例在 Internet Explorer 中无效。</p>

</body>
</html>
```

## `transition`的触发方式
&emsp;&emsp;**第一种方式如上**, 使用伪类的方式触发, 包括`hover`,`focus`,`checked`等方式; 但是实际使用当中我们更多的是使用JS或者Jquery直接修改属性, 但是工作中发现这样不行.

&emsp;&emsp;**JS触发**: 如果使用JS或者Jquery直接修改CSS属性并不会触发渐变, JS的触发方式是`toggleClass`; 我的理解是必须元素发生什么改变使得它有了一些不同从而获取到一些新的属性, 对于伪类触发是这样, 对于JS触发方式应当是它的`class`发生改变以至于能够得到新的样式.

&emsp;&emsp;**我使用的方式**:对于`p`先有一个样式, 然后同时有一个`div.newclass p`的样式; 我们通过给`div`添加`newclass`的类使得`p`发生改变获取到该类, 能够触发`transition`的渐变.

```
<div><p></p></div>
```