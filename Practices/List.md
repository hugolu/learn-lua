# List

Lua 本身沒有實作 List，使用上不方便，網路上找到有人實作出來但又不全然是我想要的，拿來練習修改一下。

程式碼來源：[BlackBulletIV/example.lua](https://gist.github.com/BlackBulletIV/4084042)

**list.lau**
```lua
list = {}
list.__index = list

setmetatable(list, { __call = function(_, ...)
  local t = setmetatable({ length = 0 }, list)
  for _, v in ipairs{...} do t:push(v) end
  return t
end })

local function find(self, value)
  if not self.first then return end
  local t = self.first
  while t do
    if t.value == value then break end
    t = t._next
  end
  return t
end

function list:push(value)
  t = {}
  t.value = value
  if self.last then
    self.last._next = t
    t._prev = self.last
    self.last = t
  else
    -- this is the first node
    self.first = t
    self.last = t
  end

  self.length = self.length + 1
end

function list:unshift(value)
  t = {}
  t.value = value
  if self.first then
    self.first._prev = t
    t._next = self.first
    self.first = t
  else
    self.first = t
    self.last = t
  end

  self.length = self.length + 1
end

function list:insert(value, _after)
  after = find(self, _after)
  t = {}
  t.value = value

  if after then
    if after._next then
      after._next._prev = t
      t._next = after._next
    else
      self.last = t
    end

    t._prev = after
    after._next = t
    self.length = self.length + 1
  elseif not self.first then
    -- this is the first node
    self.first = t
    self.last = t
  end
end

function list:pop()
  if not self.last then return end
  local ret = self.last

  if ret._prev then
    ret._prev._next = nil
    self.last = ret._prev
    ret._prev = nil
  else
    -- this was the only node
    self.first = nil
    self.last = nil
  end

  self.length = self.length - 1
  return ret.value
end

function list:shift()
  if not self.first then return end
  local ret = self.first

  if ret._next then
    ret._next._prev = nil
    self.first = ret._next
    ret._next = nil
  else
    self.first = nil
    self.last = nil
  end

  self.length = self.length - 1
  return ret.value
end

function list:remove(value)
  t = find(self, value)
  if not t then return end

  if t._next then
    if t._prev then
      t._next._prev = t._prev
      t._prev._next = t._next
    else
      -- this was the first node
      t._next._prev = nil
      self._first = t._next
    end
  elseif t._prev then
    -- this was the last node
    t._prev._next = nil
    self._last = t._prev
  else
    -- this was the only node
    self._first = nil
    self._last = nil
  end

  t._next = nil
  t._prev = nil
  self.length = self.length - 1
end

local function iterate(self, current)
  if not current then
    current = self.first
  elseif current then
    current = current._next
  end

  return current
end

function list:iterate()
  return iterate, self, nil
end
```

**test.lua**
```
require("list")

function printf(s,...)
  return io.write(s:format(...))
end

function puts(str)
  io.write(str)
end

function show(msg, list)
  puts(msg)
  for v in list:iterate() do
    printf("%d ", v.value)
  end
  print("")
end

local a = 3
local b = 4
local l = list(2, a, b, 5)
show("init: ", l)
printf("length: %d\n", l.length)

l:pop()
show("pop: ", l)

l:shift()
show("shift: ", l)

l:push(6)
show("push: ", l)

l:unshift(7)
show("unshift: ", l)

l:remove(a)
show("remove: ", l)

l:insert(8, b)
show("insert: ", l)
```

執行結果:
```
$ lua test.lua
init: 2 3 4 5
length: 4
pop: 2 3 4
shift: 3 4
push: 3 4 6
unshift: 7 3 4 6
remove: 7 4 6
insert: 7 4 8 6
```