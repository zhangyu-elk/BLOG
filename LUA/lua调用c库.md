## lua以面对对象的方式调用C

> [我把Redis中的hyperloglog算法拆分出来, 并提供了lua的接口](https://github.com/zhangyu012/hyperloglog.git)
> 感觉用的可能性不高, 不过还是记录一下

### lua中的面对对象
> lua实质是不存在面对对象的说法的, 通常是以metatable的方式为table(或userdata)提供面对对象的方法的

当对对象操作的成员不存在时, 就会去元表中查询,  __newindex 元方法用来对表更新，__index则用来对表访问(这两个成员也可以是一个表, 直接访问这个表对应的key)

### 大致实现

#### 初始化
```
luaL_newmetatable(L, METANAME);

/* metatable.__index = metatable */
lua_pushvalue(L, -1);
lua_setfield(L, -2, "__index");

luaL_setfuncs(L, fun, 0);
lua_pop(L, 1);
```
* 1 **luaL_newmetatable**生成一个元表(只有这个才能给userdata使用)
* 2 **lua_setfield**将该表的__index指向自身(之后会把各种函数注册到这个表中)

#### 生成对象
```
A **ptr = NULL;
	
ptr = lua_newuserdata(L, sizeof(A*));
luaL_getmetatable(L, METANAME);
lua_setmetatable(L, -2);
```

**lua_newuserdata** 会根据大小分配一块空间, 并返回其地址
**luaL_getmetatable** 获取我们刚刚制作的那个元表

> 绝大多数是我们会再lua中分配一个指针类型userdata, 然后这个指针指向我们需要控制的地址, 所以我们得到的返回值是一个二次指针, 因为我们往往需要自行控制资源的生成释放, 而且直接再堆栈里分配大量空间感觉也不好

将新的数据的元表设置为刚刚我们生成的那个metatable, 这样就可以以面对对象的方式调用metatable注册的函数了

*ps: 可以用`luaL_checkudata(L, 1, METANAME)`进行校验是否是正确的userdata*

最后调用的时候堆栈中的第一个参数就是这个userdata了


