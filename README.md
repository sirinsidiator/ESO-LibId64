<!--
SPDX-FileCopyrightText: 2023 sirinsidiator

SPDX-License-Identifier: Artistic-2.0
-->

LibId64 is a wrapper around the id64 (=uint64) datatype in Elder Scrolls Online.
It aims to make working with id64s as convenient as possible, while hiding away all the computer science, undocumented Lua features and dark (meta table) magic.

Simply pass an id64 or a string containing a number to the function `id64()` and it will hand you the wrapper object. Passing `nil` or an already wrapped id64 will return them unchanged. Other value types will throw an error.
There are also two additional functions `id64.isSafeNumber()` and `id64.fromNumber()` which can be used to convert certain Lua numbers. Check the reference section below for more details.

```lua
local value1 = id64(GetNextMailId())
local value2 = id64("1234567890")
local value3 = id64.fromNumber(1)
local value4 = id64(value1)
```

The wrapper has three properties `id64`, `string` and `number`, which return the underlying id64 and string representation for use with APIs and the saved variables and optionally the numeric presentation if the id64 is in the range that can be represented by Lua numbers.

```lua
GetMailItemInfo(value1.id64)
mySaveData.someValue = value2.string -- or tostring(value2)
mySaveData.anotherValue = value3.number
```

But that's not all - you can use the wrapper to do math operations with other id64s!
It's limited to additions and subtractions only and numbers starting from `9007199254740992` (2<sup>53</sup>) will use string based arithmetic, which will have abysmal performance.

```lua
local result1 = 1 + value2
local result2 = value2 + value3
local result3 = value4 - 1337
```

It also supports logic operators, allowing you to do comparisons between two id64 objects with regular Lua syntax instead of having to call the API functions yourself.

```lua
d(value1 == value4) -- true
if value1 > result1 then d("checks out") end
```

Last but not least you can concatenate the wrapper to a string!
```lua
d("my result: " .. result2)
```

The wrapper objects are immutable and only exist once per value, which makes them somewhat memory efficient. You should still try to avoid keeping every single id64 wrapped all the time and instead only wrap them when you actually work with them, so the garbage collector can free up the memory.


## Reference:

**id64()**
```
id64obj|nil = id64(string|id64|id64obj|nil)
```

Creates an id64 wrapper from a string containing a number or an id64. `nil` and already wrapped id64s are returned as is. Other value types are not supported and will throw an error.

**id64.fromNumber()**
```
id64obj = id64.fromNumber(luaint53)
```

Creates an id64 wrapper from integers between `-9007199254740992` (-2<sup>53</sup>) and `9007199254740992` (2<sup>53</sup>). Other integer values cannot be [safely represented](https://en.wikipedia.org/wiki/Double-precision_floating-point_format#Precision_limitations_on_integer_values) as a Lua number and will throw an error.

**id64.isSafeNumber()**
```
bool = id64.isSafeNumber(number)
```

Returns true if the passed number is between `-9007199254740992` (-2<sup>53</sup>) and `9007199254740992` (2<sup>53</sup>). 

**id64.isInstance()**
```
bool = id64.isInstance(any)
```

Returns `true` if the passed value is an id64 wrapper object.

**Properties**

| name       | type           | comment                                                                                         |
|------------|----------------|-------------------------------------------------------------------------------------------------|
| string     | `string`       | The string representing the id64 value. Mainly used for storing them in saved variables.        |
| id64       | `id64`         | The actual id64 value for use with API functions.                                               |
| number     | `luaint53|nil` | Either a Lua number or nil, if the id64 is outside of the range that can be safely represented. |

**Operators**

| type       | supported                        |
|------------|----------------------------------|
| arithmetic | `+`, `-`                         |
| logic      | `==`, `~=`, `<`, `>`, `<=`, `>=` |
| string     | `..`                             |
