## C和lua中的闭包

> 在看openwrt的uci的代码时看到对lua5.1实现的luaL_setfuncs(注册一个函数到lua中)函数, 学习了一下闭包

### luaL_register 和 luaL_setfuncs函数

> lua_register函数是在5.1中的函数, 在5.2中被废弃
> [抵制module原因](https://moonbingbing.gitbooks.io/openresty-best-practices/lua/not_use_module.html)

* luaL_register会先创建一个全局table在把函数注册到这个table中, 这大概与上述module的原因是类似的吧
* 下面是uci中实现的luaL_setfuncs代码, 今天主要研究这部分的代码(关于闭包)
```
static void luaL_setfuncs (lua_State *L, const luaL_Reg *l, int nup)
{
	luaL_checkstack(L, nup + 1, "too many upvalues");
	/* 调整栈的大小 */
	for (; l->name != NULL; l++)
	{  
		int i;
		lua_pushstring(L, l->name);
		/*设置函数名字*/
		for (i = 0; i < nup; i++)
		{			
			lua_pushvalue(L, -(nup + 1));
		}
		lua_pushcclosure(L, l->func, nup);
		/*生成闭包*/
		lua_settable(L, -(nup + 3));
		/*将闭包放入table中*/
	}
	lua_pop(L, nup);  
}
```


### 闭包
> 在这里闭包是一个双向的概念, 既对C又对lua
> [例子](https://www.cnblogs.com/biyeqingfeng/p/4990101.html)
> 环境: lua5.1
请注意: **上述的例子是存在问题的**
下面就以上面的例子代码进行说明: 

#### upvalue
*  不是函数的局部变量,即在函数中没有用local修饰
*  不是全局变量,即他在函数外部还是要用local修饰

#### 对于C来说
* 生成
```
lua_pushinteger(L,10);
lua_pushinteger(L,11);
lua_pushcclosure(L,mytest,2);
lua_setglobal(L,"upvalue_test");
```
**lua_pushinteger**函数将两个upvalue放入堆栈中
**lua_pushcclosure**放入函数生成闭包, mytest是函数, 2表示upvalue的个数, 闭包相关2个upvalue会被弹出

* 使用(被lua调用)
```
lua_tonumber(L, lua_upvalueindex(1));
lua_pushinteger(L, upv);
lua_replace(L, lua_upvalueindex(1));
```
**lua_upvalueindex**获取第一个upvalue的索引(我进行过打印, 其值为10003, 感觉没有研究的必要)
**lua_replace**可以用来修改这个upvalue

#### 对于lua来说(C中操作lua)
> lua中的闭包想必更多人用过吧
```
--lua
local local_upvalue = 100
 
function l_counter()
   return function ()
      local_upvalue = local_upvalue + 1
      return local_upvalue
   end
end
```
```
//获取fclosure上值的名称(临时值, 不带env)
lua_getglobal(L, "l_counter");
const char *name = lua_getupvalue(L, -1, 1);
printf("%s\n", name);
 
//设置fclosure上值
lua_getglobal(L, "l_counter");
lua_pushinteger(L,1000);
name = lua_setupvalue(L, -2, 1);
printf("%s\n", name);
```

**name = lua_getupvalue(L, -1, 1)**返回栈中坐标-1的函数的第一个upvalue, 放入栈顶
**lua_setupvalue(L, -2, 1)** 将栈顶的参数设置为坐标为-2的函数的第1个upvalue
对于upvalue来说, 按照声明的顺序即为索引

> ps: 返回值是变量的名字, 如果没有(索引超出), 则返回NULL

#### 全局upvalue
> 我认为这个概念是错误的
```
--lua
function g_counter()
   return function ()
      gloval_upvalue = gloval_upvalue + 1
      return gloval_upvalue,gloval_upvalue1
   end
end
```
```
//获取fclosure的上值（带env）
lua_getglobal(L, "g_counter");
lua_getupvalue(L, -1, 1);
   
//通过settable重新设置env中对应的值
lua_pushstring(L, "gloval_upvalue");
lua_pushinteger(L,10000);
lua_settable(L,-3);
```
***我查了一下网上说明, 如果使用了全局变量第一个upvalue必然是全局table: _ENV***

***但是***
* 1 从结果来看, 即尝试获取`g_counter`的上值, 返回值是NULL

* 2 后续设置`gloval_upvalue`时报错: `PANIC: unprotected error in call to Lua API (attempt to index a function value)`

* 3 对于全局变量来说应该不存在upvalue这个概念

* 当执行了`luaL_dofile(L, "./luatest.lua")`这一句后, 堆栈中已经获取了全局的环境
```
lua_pushinteger(L,20000);
lua_setglobal(L, "gloval_upvalue");
```
通过了这种方式就已经可以了

> 以上是我在我自己的环境中实验得到的
> 感觉离开学校后只能从网上找资料, 很多时候很难获得很权威的结果, 只能进行自己的实验, 但是又很难深入, 甚至还可能走入误区



