# lua中return提示'end' expected

## 问题

&emsp;&emsp;最近在使用lua的时候发现, LUA是不能随便返回的, `return`语句的位置是有要求的

## 错误

```
function test( ... )
	-- body
	local a = 100
	return a
	print(a)
end
```

**报错**:`lua: test.lua:6: 'end' expected (to close 'function' at line 2) near 'print'`

## 原因

&emsp;&emsp;从网上查阅的资料可以知道, `return`只能是在代码段的末尾才能够使用, 事实上从上面的报错也猜的出来; 那么我们就给它一个块的结尾位置

```
--1
function test( ... )
	-- body
	local a = 100
	do return a end
	print(a)
end

--2
function test( ... )
	-- body
	local a = 100
	if true then return a end
	print(a)
end
```