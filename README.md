# Luau Cheatsheet

A maybe-helpful summary of [the Luau docs](https://luau-lang.org/)

- [Types](#types)
  * [Modes](#modes)
    + [Strict vs Nonstrict](#strict-vs-nonstrict)
  * [Inference](#inference)
  * [Type annotations](#type-annotations)
    + [Type Set Ops](#type-set-ops)
    + [Functions](#functions)
    + [Tables](#tables)
- [Syntax](#syntax)
  * [Backwards-compatibility](#backwards-compatibility)
  * [New stuff!](#new-stuff-)
  * [Roblox types](#roblox-types)
  * [Added from 5.2](#added-from-52)
  * [Added from 5.3](#added-from-53)
  * [Added from 5.4](#added-from-54)

## Types

### Modes
Modes must be defined at the top of the file.
* `--!nonstrict` (default): Infer `any` for types early.
* `--!strict`: Infers types based on analysis of your script (initalized values, function params & return values, etc).
* `--!nocheck`: Typechecker is not run at all.

#### Strict vs Nonstrict
Strict mode will infer types of variables declared but not immediately defined, non-strict will infer `any`, ie
```luau
local foo = 1 -- Strict: number; nonstrict: number

local bar
bar = 1   -- Strict: number; nonstrict: any

local function f(x) return x end -- Strict: <A>(A) -> A; nonstrict: (any) -> any
```

Strict mode will emit an error for global variables, use local variables instead

### Inference
`any` essentially disables typechecking, literally "anything is ok here!"
`nil` is never inferred since that isn't useful

Types will cascade, ie
```luau
local function greetingsHelper(name: string)
    return "Hello, " .. name
end

local function greetings(name)
    return greetingsHelper(name)
end

print(greetings("Alexander"))          -- ok
print(greetings({name = "Alexander"})) -- not ok
```

### Type annotations
Define a type by annotating a variable or function with the `:` symbol, ie:
```luau
function foo(x: number, y: string): boolean
    local k: string = y:rep(x)
    return k == "a"
end
```
Or create a type alias by using the `type` keyword, ie.
```luau
type Foo = any
type Point = { x: number, y: number }
type Array<T> = { [number]: T }
type Something = typeof(string.gmatch("", "\d"))
```
Type aliases are local to their own file unless the `export` keyword is used, ie
```luau
type Foo = any
export Foo
export type Point = {x: number, y: number}
```
Exported types can be used in another module by prefexing its name with the require alias, ie
```luau
local M = require(Other.Module)

local a: M.Point = {x=5, y=6}

```

Override types with `::`, ie: `local k = (y :: string):rep(x)`

Built-in types are `any`, `nil`, `boolean`, `number`, `string`, `thread`

Optional parameters can be denotated by `?`, ie `number?`

Generic types can be defined with `<T>` ie `type Pair<T> = {first: T, second: T}`

Variadic args can be typed like any other argument, ie `local function f(...: number)`



#### Type Set Ops
Intersection (conforms to both): A & B, ie `((number) -> string) & ((boolean) -> string)`
Union (either-or): A | B, ie `(number | boolean) -> string`

Checking a type inherently refines it, ie

```luau
local stringOrNumber: string | number = "foo"

if type(x) == "string" then
    local onlyString: string = stringOrNumber -- ok
    local onlyNumber: number = stringOrNumber -- not ok
end

local onlyString: string = stringOrNumber -- not ok
local onlyNumber: number = stringOrNumber -- not ok

local maybeString: string? = nil

if maybeString then
    local onlyString: string = maybeString -- ok
end

local stringOrNumber2: string | number = "foo"

assert(type(stringOrNumber2) == "string")

local onlyString: string = stringOrNumber2 -- ok
local onlyNumber: number = stringOrNumber2 -- not ok
```

#### Functions

Function types are defined with `() ->`, ie. `local foo: (number, string) -> boolean`

Functions which return 0 or more than 1 values must be wrapped with `()`, ie:
```luau
local no_returns: (number, string) -> ()
local returns_boolean_and_string: (number, string) -> (boolean, string)
```

A function's variable names may optionally be specified in the type, ie:
```luau
local callback: (errorCode: number, errorText: string) -> ()
```

Generic functions will be defined as `<T>({T}, T) -> ()` but is not available at time of writing.

#### Tables

Tables are `unsealed`, `sealed`, or `generic`.
* `unsealed` tables can have properties added and can have their values changed
* `sealed` tables CANNOT have properties added but can have their values changed
* `generic` tables have some interface defined, but not completely

```luau
local function vec2(x, y)
    local t = {} -- table is unsealed
    t.x = x
    t.y = y
    return t
end

local v2 = vec2(1, 2) -- table exits its original scope and is now sealed
v2.z = 3 -- not ok

local t = {x = 1} -- table is sealed since it had things defined in it
t.y = 2           -- not ok

local function f(t)
    return t.x + t.y -- table is generic since we cannot know that
                     -- there aren't more properties than x and y
end

f({x = 1, y = 2})        -- ok
f({x = 1, y = 2, z = 3}) -- ok
f({x = 1})               -- not ok
```

Table types are specified using the same syntax as creating a table, but with `:`, ie

```luau
local array: { [number] : string }
local arrayShorthand: {string}
local object: { x: number, y: string }
local arrayOfObjects: {[number] : { x: number, y: string}}
```




## Syntax

### Backwards-compatibility
Any valid Lua 5.1 code is valid Luau... (See: [Lua 5.1 Reference Manual](https://www.lua.org/manual/5.1/manual.html#2))

...unless there's sandboxing concerns (See: [Luau Compatibility, Lua 5.1](https://luau-lang.org/compatibility#lua-51))

### New stuff!
* `break` and `continue` keywords exist now
* Compound assignments (`+=`, `-=`, etc) exist now

### Roblox types
All roblox types are known to the typechecker by name.

### Added from 5.2
* yieldable pcall/xpcall
* hex and \z escapes in strings
* arguments for function called through xpcall
* optional base in math.log
* frontier patterns
* %g in patterns
* \0 in patterns
* bit32 library
* string.gsub is stricter about using % on special characters only

### Added from 5.3
* \u escapes in strings
* basic utf-8 support
* functions for packing and unpacking values (string.pack/unpack/packsize)
* new function table.move
* collectgarbage("count") now returns only one result
* coroutine.isyieldable

### Added from 5.4
* new implementation for math.random
* The function print calls __tostring instead of tostring to format its arguments.
