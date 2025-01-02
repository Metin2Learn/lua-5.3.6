# 升级lua-5.3.6

https://www.inforge.net/forum/threads/c-upgrade-da-lua-5-0-a-lua-5-3.457789/

### lauxlib.h

```
/*
** Compatibility macros and functions
*/

LUALIB_API int  lua_dofile (lua_State *L, const char *filename);
LUALIB_API int  lua_dostring (lua_State *L, const char *str);
LUALIB_API int  lua_dobuffer (lua_State *L, const char *buff,size_t sz, const char *n);
```

### lauxlib.c

```
/*
** {======================================================
** compatibility code
** =======================================================
*/


static void callalert (lua_State *L, int status) {
  if (status != 0) {
  lua_getglobal(L, "_ALERT");
  if (lua_isfunction(L, -1)) {
  lua_insert(L, -2);
  lua_call(L, 1, 0);
  }
  else {  /* no _ALERT function; print it on stderr */
  fprintf(stderr, "%s\n", lua_tostring(L, -2));
  lua_pop(L, 2);  /* remove error message and _ALERT */
  }
  }
}


static int aux_do (lua_State *L, int status) {
  if (status == 0) {  /* parse OK? */
  status = lua_pcall(L, 0, LUA_MULTRET, 0);  /* call main */
  }
  callalert(L, status);
  return status;
}


LUALIB_API int lua_dofile (lua_State *L, const char *filename) {
  return aux_do(L, luaL_loadfile(L, filename));
}


LUALIB_API int lua_dobuffer (lua_State *L, const char *buff, size_t size,
  const char *name) {
  return aux_do(L, luaL_loadbuffer(L, buff, size, name));
}


LUALIB_API int lua_dostring (lua_State *L, const char *str) {
  return lua_dobuffer(L, str, strlen(str), str);
}

```


### ltablib.c
添加了 table.getn 函数以保持向后兼容性
```
static int luaB_getn (lua_State *L) {
  lua_pushnumber(L, (lua_Number)aux_getn(L, 1, TAB_R));
  return 1;
}

/* }====================================================== */


static const luaL_Reg tab_funcs[] = {
  {"concat", tconcat},
#if defined(LUA_COMPAT_MAXN)
  {"maxn", maxn},
#endif
  {"getn", luaB_getn}, // MODDED - backward compatibility
  {"insert", tinsert},
  {"pack", pack},
  {"unpack", unpack},
  {"remove", tremove},
  {"move", tmove},
  {"sort", sort},
  {NULL, NULL}
};
```


删除./liblua中的旧文件并上传新的修改文件。

### 替换内容
您需要在所有文件中替换：
“luaL_reg”与“luaL_Reg”
“luaL_getn”与“lua_objlen”
“lua_tonumber”和“lua_tointeger”（questlua.cpp 中的“combine_lua_string”函数除外）
“lua_pushnumber”与“lua_pushinteger”


### questlua.cpp

```
    L = luaL_newstate();

     luaL_openlibs(L); // LUA 5.3
```