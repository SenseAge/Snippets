## Requement: 
为了使得esp8266在上电后快速进入工作状态，用户没有延迟卡顿感，需要尽量缩短NodeMCU进入工作状态时间

#### Way 1: 缩短NodeMCU Lua固件解释器启动时间
这一点，本团队现有技术实力无法做到

#### Way 2: 取消掉NodeMCU启动时打印的固件信息
1. 固件信息是在固件构建中，以参数宏的形式添加，找到这一点的途径是在使用Docker构建固件的时候发现固件信息中会带有Docker字样
```
NodeMCU 2.2.0.0 built with Docker provided by frightanic.com
	branch: master
	commit: 4095c408e6a8cc9718cb06007b408d0aad15d9cd
	SSL: true
	Build type: float
	LFS: disabled
	modules: file,gpio,http,net,node,pwm,sjson,
tmr,uart,wifi
 build created on 2018-11-15 05:50
 powered by Lua 5.1.4 on SDK 2.2.1(6ab97e9)
 ```
 因此通过这些信息是在通过[Docker构建脚本(https://github.com/SenseAge/docker-nodemcu-build/blob/master/build-esp8266)](https://github.com/SenseAge/docker-nodemcu-build/blob/master/build-esp8266)中找到
所有用户配置信息都在[app/include(https://github.com/nodemcu/nodemcu-firmware/tree/11592951b90707cdcb6d751876170bf4da82850d/app/include)](https://github.com/nodemcu/nodemcu-firmware/tree/11592951b90707cdcb6d751876170bf4da82850d/app/include)其中模块信息是使用脚本
目录下。模块信息是通过脚本检索user_modules.h打开的宏。
2. 在[app/lua/lua.c(https://github.com/nodemcu/nodemcu-firmware/blob/11592951b90707cdcb6d751876170bf4da82850d/app/lua/lua.c)](https://github.com/nodemcu/nodemcu-firmware/blob/11592951b90707cdcb6d751876170bf4da82850d/app/lua/lua.c)文件中，
函数
```C
static void print_version (lua_State *L) {
  lua_pushliteral (L, "\n" NODE_VERSION " build " BUILD_DATE " powered by " LUA_RELEASE " on SDK ");
  lua_pushstring (L, SDK_VERSION);
  lua_concat (L, 2);
  const char *msg = lua_tostring (L, -1);
  l_message (NULL, msg);
  lua_pop (L, 1);
}
```
该函数在同一文件中`static int pmain (lua_State *L)`函数中被调用。

因此去掉版本信息的方式就是修改print_version函数内容。[lua_State结构体](https://www.cnblogs.com/cnxkey/articles/4235127.html)是脚本相关的类型，
在不了解Lua解释器的情况下，修改本函数要谨慎，

电脑没电了，明年再


