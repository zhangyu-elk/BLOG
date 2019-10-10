# Form中的button点击时默认提交表单
> 遇到一个问题, 当点击`Form`中的`button`时也会提交表单, 做个记录

## 原因
&emsp;&emsp;对于`Form`中的`button`来说, 其有`type`属性, 存在`button`,`submit`,`reset`三种可选属性, `IE`默认是`button`, 而其他浏览器默认为`submit`, 这就是原因

**结论:**
&emsp;&emsp;对于Form中使用`button`情况, 请显示的执行其`type`
```
<button type="button" id="test">Test</button>
```