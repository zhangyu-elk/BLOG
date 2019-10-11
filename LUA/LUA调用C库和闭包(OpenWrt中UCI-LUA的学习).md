## LUA调用C库和闭包(OpenWrt中UCI-LUA的学习)
> 不是为了学习UCI, 而是学习UCI中C代码和LUA的交互使用, 可以不用了解UCI具体是干什么的

### 说明
&emsp;&emsp;介绍一下再LUA中以面对对象的方式调用C代码, 以及LUA-C中的闭包概念; LUA调用C代码是C编译为.so动态库, 注册函数供LUA调用.


### 注册函数
&emsp;&emsp;LUA5.1中提供了`module`的方式来定义模块(LUA中), 不过应当别抵制[(抵制module原因)](https://moonbingbing.gitbooks.io/openresty-best-practices/lua/not_use_module.html); 这里是LUA代码被LUA代码调用. 
&emsp;&emsp;对于C代码来说, 主要是`luaL_register`和`luaL_setfuncs`函数, 前者已经被废弃, 因为会产生一个全局变量, 后者是往一个表中注册函数.

### UCI介绍
> 不是为了介绍UCI, 仅仅是介绍一些UCI怎么调用这些C中的函数
```
local uci = require "uci".cursor()
local test = uci:get("system","main","test")
```
`cursor`,`get`都是C代码中提供的函数, 我们可以看到是以一种类似于面对对象的方式直接调用C中的函数的

### lua中的面对对象
> lua实质是不存在面对对象的说法的, 实质以metatable的方式为table(或userdata)提供面对对象的方法的(:方式调用)

&emsp;&emsp;当对对象操作的成员不存在时, 就会去元表中查询,  `__newindex` 元方法用来对表更新，`__index`则用来对表访问(这两个成员可以是函数或者是表), 这里不做特别详细介绍; 如果我们要实现LUA中以面对对象的方式调用C函数, 也是要创建类似环境. C中的结构体在LUA中就是一个`userdata`


### C库注册函数

#### 函数介绍
`int   luaL_newmetatable (lua_State *L, const char *tname);`
和`lua_newtable`类似, 也是用来生成一个指定名字的表, 但是只有前者能够作为`userdata`的`metatable`

`void lua_setfield (lua_State *L, int index, const char *k);`
将堆栈`L`中的第`index`值, 设置到栈顶的`table`的`k`成员

`void *lua_newuserdata (lua_State *L, size_t size);`
根据指定的大小, 生成一块地址

`void  luaL_getmetatable (lua_State *L, const char *tname);`
获取指定名字的`metatable`

`void lua_settable (lua_State *L, int index);`
t[k] = v, `t`即为有效索引为`index`的值, 栈顶作为`v`, 栈顶的下一个作为`index`, 最后弹出栈

#### 思路
&emsp;&emsp; 为C对象添加一个壳，lua 的 userdata 中仅仅保存 C 对象指针。然后给 userdata 设置 gc 元方法，在被回收时，正确调用 C 对象的销毁函数。 所有的函数均放在改`userdata`的元表中, 调用时去元表中查找(可能还有其他方式, 但我感觉这已经挺好用的了)

#### 生成对象
```
//1045
int
luaopen_uci(lua_State *L)
{
	/* create metatable */
	luaL_newmetatable(L, METANAME);

	/* metatable.__index = metatable */
	lua_pushvalue(L, -1);
	lua_setfield(L, -2, "__index");

	/* fill metatable */
	luaL_setfuncs(L, uci, 0);
	lua_pop(L, 1);

	/* create module */
	lua_newtable(L);
	lua_pushvalue(L, -1);
	luaL_setfuncs(L, uci, 0);
	lua_setglobal(L, MODNAME);

	return 1;
}

//986
static int
uci_lua_cursor(lua_State *L)
{
	struct uci_context **u;
	int argc = lua_gettop(L);

	u = lua_newuserdata(L, sizeof(struct uci_context *));
	luaL_getmetatable(L, METANAME);
	lua_setmetatable(L, -2);
	......
```
以上包括初始化环境, 以及生成一个对象的代码, 主要包括如下几步
* 1 create metatable 生成一个元表
* 2 metatable.\__index = metatable 将这个元表的元表指向自身, 这样`userdata`寻找方法时就会在这里寻找 
* 3 fill metatable 使用`luaL_setfuncs`将函数注册到该表中
* 4 create module 对于这一段, 个人认为对于我们面对对象的调用方式可能用处不大了
* 5 lua_newuserdata 创建一个指针大小的空间, 用于存放C对象的指针, 我们可以在返回值`u`中赋值存放结构体指针
* 6 lua_setmetatable 将我们之前创建的元表设置为该结构体的元表
* 7 最后我们创建的该`userdata`就可以以面对对象的方式调用我们最开始注册的那些函数了

***注意:***`{ "__gc", uci_lua_gc },`该函数在LUA回收该资源时调用的, 如果有动态创建资源那么必须在这里回收(LUA会自动回收资源)

ps: ***调用函数时可以用`luaL_checkudata(L, 1, METANAME)`进行校验是否是正确的userdata***, 调用函数时(比如之前的`get`)第一个参数就是该`userdata`

### 闭包
> 在看openwrt的uci的代码时看到为兼容lua5.1实现的luaL_setfuncs(注册一个函数到lua中)函数, 涉及到了闭包, 所以学习一下
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
`nup`就是`l`中所有函数的闭包个数, 这些`upvalues`都必须在调用改函数之前`push`到栈上


**闭包学习**
> 在这里闭包是一个双向的概念, 既对C又对lua
> [例子](https://www.cnblogs.com/biyeqingfeng/p/4990101.html)
> 环境: lua5.1

**贴上代码(注意:代码有问题):**
```
//C代码

#include <stdio.h>
#include <string.h>
#include <lua.h>
#include <lauxlib.h>
#include <lualib.h>
#include <dlfcn.h>
#include <math.h>
 
static int mytest(lua_State *L) {
  //获取上值
  int upv = (int)lua_tonumber(L, lua_upvalueindex(1));
  printf("%d\n", upv);
  upv += 5;
  lua_pushinteger(L, upv);
  lua_replace(L, lua_upvalueindex(1));
 
  //获取一般参数
  printf("%d\n", (int)lua_tonumber(L,1));
 
  return 0;
}
 
int main(void) {
  lua_State *L = luaL_newstate();
  luaL_openlibs(L);
 
  //设置Cclosure函数的上值
  lua_pushinteger(L,10);
  lua_pushinteger(L,11);
  lua_pushcclosure(L,mytest,2);
  lua_setglobal(L,"upvalue_test");
  luaL_dofile(L, "./luatest.lua");
 
  //获取fclosure上值的名称(临时值, 不带env)
  lua_getglobal(L, "l_counter");
  const char *name = lua_getupvalue(L, -1, 1);
  printf("%s\n", name);
 
  //设置fclosure上值
  lua_getglobal(L, "l_counter");
  lua_pushinteger(L,1000);
  name = lua_setupvalue(L, -2, 1);
  printf("%s\n", name);
 
  lua_getglobal(L,"ltest");
  lua_pcall(L,0,0,0);
  lua_getglobal(L,"ltest");
  lua_pcall(L,0,0,0);
 
  //获取fclosure的上值（带env）
  lua_getglobal(L, "g_counter");
  lua_getupvalue(L, -1, 1);
   
  //通过settable重新设置env中对应的值
  lua_pushstring(L, "gloval_upvalue");
  lua_pushinteger(L,10000);
  lua_settable(L,-3);
   
  lua_pushstring(L, "gloval_upvalue1");
  lua_pushinteger(L,20000);
  lua_settable(L,-3);
   
  lua_getglobal(L,"gtest");
  lua_pcall(L,0,0,0);
  lua_close(L);
  return 0;
}

//LUA代码
gloval_upvalue = 10
gloval_upvalue1 = 20
local local_upvalue = 100
 
function l_counter()
   return function ()
      local_upvalue = local_upvalue + 1
      return local_upvalue
   end
end
 
function g_counter()
   return function ()
      gloval_upvalue = gloval_upvalue + 1
      return gloval_upvalue,gloval_upvalue1
   end
end
 
g_testf = g_counter()
l_testf = l_counter()
 
function gtest()
  print(g_testf())
end
 
 
function ltest()
  print(l_testf())
end
 
upvalue_test(1,2,3)
upvalue_test(4,5,6)
```
接下来针对代码说明

#### upvalue概念
*  不是函数的局部变量,即在函数中没有用local修饰
*  不是全局变量,即他在函数外部还是要用local修饰

#### 对于C方面来说, 
```
lua_pushinteger(L,10);
lua_pushinteger(L,11);
lua_pushcclosure(L,mytest,2);
lua_setglobal(L,"upvalue_test");

static int mytest(lua_State *L) {
  //获取上值
  int upv = (int)lua_tonumber(L, lua_upvalueindex(1));
  printf("%d\n", upv);
  upv += 5;
  lua_pushinteger(L, upv);
  lua_replace(L, lua_upvalueindex(1));
 
  //获取一般参数
  printf("%d\n", (int)lua_tonumber(L,1));
 
  return 0;
}

upvalue_test(1,2,3)
upvalue_test(4,5,6)
```
**lua_pushinteger**函数将两个upvalue放入堆栈中
**lua_pushcclosure**放入函数生成闭包, mytest是函数, 2表示upvalue的个数, 闭包相关2个upvalue会被弹出
这就生成了一个闭包, 这个闭包被设置为全局变量, 在lua中可以直接调用

**使用(被lua调用)**
```
lua_tonumber(L, lua_upvalueindex(1));
lua_pushinteger(L, upv);
lua_replace(L, lua_upvalueindex(1));
```
**lua_upvalueindex**获取第一个upvalue的索引(我进行过打印, 其值为10003, 感觉没有研究的必要)
**lua_replace**可以用来修改这个upvalue
`lua_tonumber(L, lua_upvalueindex(1));`: 获取第一个`upvalue`的值
`lua_replace(L, lua_upvalueindex(1))`: 修改第一个`upvalue`的值

很简单的使用方法

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
这里就是一个简单的闭包, local_upvalue就是函数的闭包

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
这里就是在C语言中获取和设置该函数的`upvalue`
**name = lua_getupvalue(L, -1, 1)**: 返回栈中坐标-1的函数的第一个upvalue的名字, 将值放入栈顶
**lua_setupvalue(L, -2, 1)**: 将栈顶的参数设置为为坐标为-2的函数的第1个upvalue
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
这是作者的说法:
`如果你的fclosure中没有使用全局变量，那么其上值索引应该是按照local变量的声明顺序来的，但是一旦使用了全局变量，第一个上值必定是_ENV环境，也就是说你可以通过重新设置_ENV环境，修改全局的上值，这也是修改环境变量的一种方法。`

在LUA5.1的测试中
```
//获取fclosure的上值（带env）
lua_getglobal(L, "g_counter");
lua_getupvalue(L, -1, 1);
   
//通过settable重新设置env中对应的值
lua_pushstring(L, "gloval_upvalue");
lua_pushinteger(L,10000);
lua_settable(L,-3);
```

***但是***
* 1 从结果来看, 即尝试获取`g_counter`的上值, 返回值是NULL

* 2 后续设置`gloval_upvalue`时报错: `PANIC: unprotected error in call to Lua API (attempt to index a function value)`

* 3 对于全局变量来说应该不存在upvalue这个概念

* 当执行了`luaL_dofile(L, "./luatest.lua")`这一句后, 堆栈中已经获取了全局的环境
```
lua_pushinteger(L,20000);
lua_setglobal(L, "gloval_upvalue");
```
通过了这种方式就已经可以了设置该变量了

> 以上是我在我自己的环境中实验得到的, 可能因为环境原因导致的结果不一致
> 感觉离开学校后只能从网上找资料, 很多时候很难获得很权威的结果, 只能进行自己的实验, 但是又很难深入, 甚至还可能走入误区



