# Lua
## Tips
### JavaScriptからLuaを使う方法
Fengari使うと可能。
```javascript
	function eval_as_lua_code(code: string): string {
		let L = lauxlib.luaL_newstate();
		lualib.luaL_openlibs(L);
		lauxlib.luaL_loadstring(L, to_luastring(code));
		lua.lua_call(L, 0, -1);
		return lua.lua_tojsstring(L, -1);
	}
```