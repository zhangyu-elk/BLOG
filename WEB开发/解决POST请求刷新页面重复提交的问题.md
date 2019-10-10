## 刷新页面重复提交
> 最近碰到一个问题, 使用POST提交表单后, 当我们使用`F5`或者浏览器网址栏左侧刷新按钮进行刷新时会造成POST请求重复提交(刷新的时候会再一次提交上一次的数据)

### 重定向方法
	通过返回`302`相应, 重定向到一个新的网址, 之后刷新也是刷新该新的网址
### `window.history.replaceState`
```
if(window.history.replaceState)
{
	window.history.replaceState(null, null, window.location.href)
}
```
`history.replaceState`的作用时把当前的页面的历史记录替换掉, 但不会刷新网页
本来历史记录是之前的POST提交, 替换后变为当前的页面网址
我们只需要在页面载入的时候执行以下这一句即可

### 可以使用AJAX提交
> 我没有用这种方法, 但我认为应该是可以的, 实现起来应该也不会特别困难

### 其他方法
	网上还有介绍一些其他的方法, 但是已经不是简单能说明的了