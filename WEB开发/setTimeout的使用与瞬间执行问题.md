# setTimeout的使用与瞬间执行问题
> `setTimeout`传入两个参数, 在执行`ms`数后执行相应的代码, 即延时执行

## 情况
&emsp;&emsp;最近在使用`setTimeout`碰到了一个问题, 发现在传入的方法瞬间就执行了, 并没有延迟执行

## 原因
&emsp;&emsp;我传入的是一串可执行代码, 而不是一个函数方法
**错误代码(并Google执行结果,这里直接在console上执行了)**
```
setTimeout(console.log("Test"), 5000)
Test
```
**正确代码**
```
setTimeout(function(){console.log("Test")}, 5000)
console.log(new Date())
Thu Oct 10 2019 15:20:20 GMT+0800 (中国标准时间)
undefined
console.log(new Date())
Thu Oct 10 2019 15:20:21 GMT+0800 (中国标准时间)
undefined
Test
console.log(new Date())
Thu Oct 10 2019 15:20:23 GMT+0800 (中国标准时间)
undefined
```

## 思考
&emsp;&emsp;在获得了正确的结果后, 我想到了C语言中的传值, 我认为实际中JS应当是这样的
```
function setTimeout(a){}
setTimeout(console.log("Test"), 5000)
```
在调用时进行了类似 C语言的赋值操作 即: `a=console.log("Test")`, 右边的表达式进行了执行
所以应当是**需要编译的代码**或者是一个**函数方法**, 当然如果这串可执行代码返回一个函数方法那么也可以

## 扩展setInterval
```
setInterval(fn,t)
```
函数原型与`setTimeout`一致, 不过是循环`t`秒调用`fn`方法, 后者仅执行一次
***注意***
&emsp;&emsp;如果`fn`的执行时间大于`t`秒, 那么第二次执行会等到第一次执行完在执行, 那么间隔时间不再是`t`了, 需要注意

## 扩展: 取消定时
```
var timed = setTimeout(function(){console.log("Test")}, 5000)
```
标准的函数原型如上, 返回一个数值ID, 唯一标识符, 可以通过它来取消
```
clearTimeout(id)
clearInterval(id)
```

***注意***
&emsp;&emsp;这里不论是设置还是清除定时, 实际应当是`window.setInterval`和`window.clearInterval`