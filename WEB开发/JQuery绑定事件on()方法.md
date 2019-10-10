# JQuery绑定事件on()方法简单说明
> 最近使用到这个方法, 做一些说明并记录

## 说明
&emsp;&emsp;`on`方法是JQuery上用于绑定事件的函数, 与之类似的有`live`,`bind`等函数, 但是都不建议使用了

## 函数原型
```
.on( events [, selector ] [, data ], handler )
```

**events**: 事件类型, 包括`click`,`keydown`等, 可以传入多个比如"mouseenter mouseleave"这样的方式, 指明多个
**selector**: 可选参数, 一个选择器, 只有符合该选择器的后代元素才能够触发
**data**: 自定义的一个参数, 最终是传递给`handler`函数
**handler**: 函数

## 注意事项
* 1 调用该`on`函数的元素必须已经存在
* 2 后代元素可以不存在, 后续添加的子元素依然可以触发事件
* 3 对于如下第二种写法来说, ①后续添加的元素可以触发; ②采用了事件委托的方式监控, 可以降低开销
```
//1
$( "#tbody tr" ).on( "click", function() {
  console.log( $( this ).text() );
});
//2
$( "#tbody" ).on( "click", "tr", function() {
  console.log( $( this ).text() );
});
```

## `handler`解析
```
function handler(event){}
```
对于该函数, 默认传入一个`event`对象
**event.data**: 即`on`函数的第三个参数
**event.target**: 实际触发的元素, `on`函数可以监控多个, 该属性指示事件发生的最深元素
**event.type**: 指明事件的类型, `events`可以传入多个的

## 扩展`off`和`one`
```
$( "#tbody" ).off( "click")
```
取消绑定的事件
```
$(selector).one(event,data,function)
```
绑定的事件仅执行一次, 并且参数只有三个