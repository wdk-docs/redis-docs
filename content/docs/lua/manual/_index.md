---
title: "参考手册"
linkTitle: ""
weight: 1
---

The reference manual is the official definition of the Lua language.
For a complete introduction to Lua programming, see the book Programming in Lua.

start · contents · index · other versions
Copyright © 2020–2021 Lua.org, PUC-Rio. Freely available under the terms of the Lua license.

## Contents

1. – Introduction
2. – Basic Concepts
   1. – Values and Types
   2. – Environments and the Global Environment
   3. – Error Handling
   4. – Metatables and Metamethods
   5. – Garbage Collection
      1. – Incremental Garbage Collection
      2. – Generational Garbage Collection
      3. – Garbage-Collection Metamethods
      4. – Weak Tables
   6. – Coroutines
3. – The Language
   1. – Lexical Conventions
   2. – Variables
   3. – Statements
      1. – Blocks
      2. – Chunks
      3. – Assignment
      4. – Control Structures
      5. – For Statement
      6. – Function Calls as Statements
      7. – Local Declarations
      8. – To-be-closed Variables
   4. – Expressions
      1. – Arithmetic Operators
      2. – Bitwise Operators
      3. – Coercions and Conversions
      4. – Relational Operators
      5. – Logical Operators
      6. – Concatenation
      7. – The Length Operator
      8. – Precedence
      9. – Table Constructors
      10. – Function Calls
      11. – Function Definitions
   5. – Visibility Rules
4. – The Application Program Interface
   1. – The Stack
      1. – Stack Size
      2. – Valid and Acceptable Indices
      3. – Pointers to strings
   2. – C Closures
   3. – Registry
   4. – Error Handling in C
      1. – Status Codes
   5. – Handling Yields in C
   6. – Functions and Types
   7. – The Debug Interface
5. – The Auxiliary Library
   5.1 – Functions and Types
6. – The Standard Libraries
   1. – Basic Functions
   2. – Coroutine Manipulation
   3. – Modules
   4. – String Manipulation
      1. – Patterns
      2. – Format Strings for Pack and Unpack
   5. – UTF-8 Support
   6. – Table Manipulation
   7. – Mathematical Functions
   8. – Input and Output Facilities
   9. – Operating System Facilities
   10. – The Debug Library
7. – Lua Standalone
8. – Incompatibilities with the Previous Version
   1. – Incompatibilities in the Language
   2. – Incompatibilities in the Libraries
   3. – Incompatibilities in the API
9. – The Complete Syntax of Lua

## Index

### Lua functions

basic
\_G
\_VERSION
assert
collectgarbage
dofile
error
getmetatable
ipairs
load
loadfile
next
pairs
pcall
print
rawequal
rawget
rawlen
rawset
require
select
setmetatable
tonumber
tostring
type
warn
xpcall
coroutine
coroutine.close
coroutine.create
coroutine.isyieldable
coroutine.resume
coroutine.running
coroutine.status
coroutine.wrap
coroutine.yield
debug
debug.debug
debug.gethook
debug.getinfo
debug.getlocal
debug.getmetatable
debug.getregistry
debug.getupvalue
debug.getuservalue
debug.sethook
debug.setlocal
debug.setmetatable
debug.setupvalue
debug.setuservalue
debug.traceback
debug.upvalueid
debug.upvaluejoin
io
io.close
io.flush
io.input
io.lines
io.open
io.output
io.popen
io.read
io.stderr
io.stdin
io.stdout
io.tmpfile
io.type
io.write
file:close
file:flush
file:lines
file:read
file:seek
file:setvbuf
file:write

math
math.abs
math.acos
math.asin
math.atan
math.ceil
math.cos
math.deg
math.exp
math.floor
math.fmod
math.huge
math.log
math.max
math.maxinteger
math.min
math.mininteger
math.modf
math.pi
math.rad
math.random
math.randomseed
math.sin
math.sqrt
math.tan
math.tointeger
math.type
math.ult
os
os.clock
os.date
os.difftime
os.execute
os.exit
os.getenv
os.remove
os.rename
os.setlocale
os.time
os.tmpname
package
package.config
package.cpath
package.loaded
package.loadlib
package.path
package.preload
package.searchers
package.searchpath
string
string.byte
string.char
string.dump
string.find
string.format
string.gmatch
string.gsub
string.len
string.lower
string.match
string.pack
string.packsize
string.rep
string.reverse
string.sub
string.unpack
string.upper
table
table.concat
table.insert
table.move
table.pack
table.remove
table.sort
table.unpack
utf8
utf8.char
utf8.charpattern
utf8.codepoint
utf8.codes
utf8.len
utf8.offset

### metamethods

**add
**band
**bnot
**bor
**bxor
**call
**close
**concat
**div
**eq
**gc
**idiv
**index
**le
**len
**lt
**metatable
**mod
**mode
**mul
**name
**newindex
**pairs
**pow
**shl
**shr
**sub
**tostring
\_\_unm

### environment variables

LUA_CPATH
LUA_CPATH_5_4
LUA_INIT
LUA_INIT_5_4
LUA_PATH
LUA_PATH_5_4

### C API

lua_Alloc
lua_CFunction
lua_Debug
lua_Hook
lua_Integer
lua_KContext
lua_KFunction
lua_Number
lua_Reader
lua_State
lua_Unsigned
lua_WarnFunction
lua_Writer
lua_absindex
lua_arith
lua_atpanic
lua_call
lua_callk
lua_checkstack
lua_close
lua_closeslot
lua_compare
lua_concat
lua_copy
lua_createtable
lua_dump
lua_error
lua_gc
lua_getallocf
lua_getextraspace
lua_getfield
lua_getglobal
lua_gethook
lua_gethookcount
lua_gethookmask
lua_geti
lua_getinfo
lua_getiuservalue
lua_getlocal
lua_getmetatable
lua_getstack
lua_gettable
lua_gettop
lua_getupvalue
lua_insert
lua_isboolean
lua_iscfunction
lua_isfunction
lua_isinteger
lua_islightuserdata
lua_isnil
lua_isnone
lua_isnoneornil
lua_isnumber
lua_isstring
lua_istable
lua_isthread
lua_isuserdata
lua_isyieldable
lua_len
lua_load
lua_newstate
lua_newtable
lua_newthread
lua_newuserdatauv
lua_next
lua_numbertointeger
lua_pcall
lua_pcallk
lua_pop
lua_pushboolean
lua_pushcclosure
lua_pushcfunction
lua_pushfstring
lua_pushglobaltable
lua_pushinteger
lua_pushlightuserdata
lua_pushliteral
lua_pushlstring
lua_pushnil
lua_pushnumber
lua_pushstring
lua_pushthread
lua_pushvalue
lua_pushvfstring
lua_rawequal
lua_rawget
lua_rawgeti
lua_rawgetp
lua_rawlen
lua_rawset
lua_rawseti
lua_rawsetp
lua_register
lua_remove
lua_replace
lua_resetthread
lua_resume
lua_rotate
lua_setallocf
lua_setfield
lua_setglobal
lua_sethook
lua_seti
lua_setiuservalue
lua_setlocal
lua_setmetatable
lua_settable
lua_settop
lua_setupvalue
lua_setwarnf
lua_status
lua_stringtonumber
lua_toboolean
lua_tocfunction
lua_toclose
lua_tointeger
lua_tointegerx
lua_tolstring
lua_tonumber
lua_tonumberx
lua_topointer
lua_tostring
lua_tothread
lua_touserdata
lua_type
lua_typename
lua_upvalueid
lua_upvalueindex
lua_upvaluejoin
lua_version
lua_warning
lua_xmove
lua_yield
lua_yieldk

### auxiliary library

luaL_Buffer
luaL_Reg
luaL_Stream
luaL_addchar
luaL_addgsub
luaL_addlstring
luaL_addsize
luaL_addstring
luaL_addvalue
luaL_argcheck
luaL_argerror
luaL_argexpected
luaL_buffaddr
luaL_buffinit
luaL_buffinitsize
luaL_bufflen
luaL_buffsub
luaL_callmeta
luaL_checkany
luaL_checkinteger
luaL_checklstring
luaL_checknumber
luaL_checkoption
luaL_checkstack
luaL_checkstring
luaL_checktype
luaL_checkudata
luaL_checkversion
luaL_dofile
luaL_dostring
luaL_error
luaL_execresult
luaL_fileresult
luaL_getmetafield
luaL_getmetatable
luaL_getsubtable
luaL_gsub
luaL_len
luaL_loadbuffer
luaL_loadbufferx
luaL_loadfile
luaL_loadfilex
luaL_loadstring
luaL_newlib
luaL_newlibtable
luaL_newmetatable
luaL_newstate
luaL_openlibs
luaL_opt
luaL_optinteger
luaL_optlstring
luaL_optnumber
luaL_optstring
luaL_prepbuffer
luaL_prepbuffsize
luaL_pushfail
luaL_pushresult
luaL_pushresultsize
luaL_ref
luaL_requiref
luaL_setfuncs
luaL_setmetatable
luaL_testudata
luaL_tolstring
luaL_traceback
luaL_typeerror
luaL_typename
luaL_unref
luaL_where
standard library
luaopen_base
luaopen_coroutine
luaopen_debug
luaopen_io
luaopen_math
luaopen_os
luaopen_package
luaopen_string
luaopen_table
luaopen_utf8
constants
LUA_ERRERR
LUA_ERRFILE
LUA_ERRMEM
LUA_ERRRUN
LUA_ERRSYNTAX
LUA_HOOKCALL
LUA_HOOKCOUNT
LUA_HOOKLINE
LUA_HOOKRET
LUA_HOOKTAILCALL
LUAL_BUFFERSIZE
LUA_MASKCALL
LUA_MASKCOUNT
LUA_MASKLINE
LUA_MASKRET
LUA_MAXINTEGER
LUA_MININTEGER
LUA_MINSTACK
LUA_MULTRET
LUA_NOREF
LUA_OK
LUA_OPADD
LUA_OPBAND
LUA_OPBNOT
LUA_OPBOR
LUA_OPBXOR
LUA_OPDIV
LUA_OPEQ
LUA_OPIDIV
LUA_OPLE
LUA_OPLT
LUA_OPMOD
LUA_OPMUL
LUA_OPPOW
LUA_OPSHL
LUA_OPSHR
LUA_OPSUB
LUA_OPUNM
LUA_REFNIL
LUA_REGISTRYINDEX
LUA_RIDX_GLOBALS
LUA_RIDX_MAINTHREAD
LUA_TBOOLEAN
LUA_TFUNCTION
LUA_TLIGHTUSERDATA
LUA_TNIL
LUA_TNONE
LUA_TNUMBER
LUA_TSTRING
LUA_TTABLE
LUA_TTHREAD
LUA_TUSERDATA
LUA_USE_APICHECK
LUA_YIELD
