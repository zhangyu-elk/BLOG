# HTML透明度样式filter和opacity

## `opacity`属性

&emsp;&emsp;`opacity`是CSS3的属性,  能够元素设置不透明度, 默认值为1(完全不透明)
```
div
{
	opacity:0.5;
	filter:Alpha(opacity=50);
}
```

**注意**: 除IE8以前的所有浏览器均支持该特性, 不过可以通过`filter:Alpha(opacity=50)`来设置.
