#User defined types #2: Glory and shame of dot notation
*Target audience: beginner to intermediate, applicable version: 1.9.16.16 and newer*

The [first article](https://github.com/petrSchreiber/ArticleSeries_UserDefinedTypes/blob/master/01_introduction/introduction.md) in the series provided some basic motivation for usage of user defined types (UDT): their ability to encapsulate multiple fields and straightforward memory consumption tracking.

The second part in the series will introduce you to dot notation, its benefits and pitfalls.

##Dot notation
Designing the data model with UDT is very straightforward, as the first article suggested. Each field of the UDT can be then accessed via dotted syntax:
```thinbasic
type vector2d
  x as single
  y as single
end type

dim myPoint as vector2d
myPoint.x = 1
myPoint.y = 2
```

The fields of the UDT may be one of the primitive types or even another UDT.
```thinbasic
type line2d
  pointA as vector2d
  pointB as vector2d
end type

dim xAxis as line2d

xAxis.pointA.x = -10
xAxis.pointA.y = 0

xAxis.pointB.x = 10
xAxis.pointB.y = 0
```

Field inheritance
You can even inherit the fields of one UDT in another. This can be done in two ways:
```thinbasic
type vector3d
  vector2d
  z as single
end type

' -- or equivalent

type vector3d extends vector2d
  z as single
end type
```

In both cases, the vector3d will be equipped with x, y and z fields.

The first approach allows specifying even multiple types to inherit fields from.
The second one allows inheriting from just one specified type, whose fields will be always inserted as first in the new type.

##Memory representation
Another interesting property of user defined types is how they are stored in memory. The following example will give you little hint:
```thinbasic
uses "console"

type vector2d
  x as single
  y as single
end type

type vector3d extends vector2d
  z as single
end type

dim v as vector3d
printL "v:   " + varPtr(v)
printL "v.x: " + varPtr(v.x)
printL "v.y: " + varPtr(v.y)
printL "v.z: " + varPtr(v.z)

printL

single x, y, z
printL "x:   " + varPtr(x)
printL "y:   " + varPtr(y)
printL "z:   " + varPtr(z)
waitKey
```
*Download as [gloryAndShameOfDotNotation01.tbasic](https://github.com/petrSchreiber/Article-series-User-defined-types/blob/master/02_gloryAndShameOfDotNotation/code/gloryAndShameOfDotNotation01.tbasic)*

By running the code above you'll be able to make a few interesting observations:

1. the pointer to the first field of UDT equals pointer to the UDT variable itself
2. by default, the fields are tightly packed in memory, one after another
3. the memory order of the fields is the same as the order in the declaration
4. the memory order of separate x, y, z variables is completely random

The third point is very important, if you want to access the fields via pointers. While it is tempting to presume .x is available at +0 offset, .y at +4 and .z at +8, please do not hardcode these.

Never. Ever.

Instead, use *UDT_ElementOffset* function to calculate the offsets dynamically. This will make your code more maintainable in case you would like to reorder the fields in the UDT later, for aesthetical purposes, for example.
```thinbasic
uses "console"

type vector3d
  x as single
  y as single
  z As Single
end type

dim v(3) as vector3d

dword v_pointer
long  i

printL "Bad practice - try changing order of fields in vector3d" in 12
v_pointer = varptr(v(1))
for i = 1 to countof(v)
  poke(single, v_pointer + 0, 1 * i)
  poke(single, v_pointer + 4, 2 * i)
  poke(single, v_pointer + 8, 3 * i)
  
  printL v(i).x, v(i).y, v(i).z
  
  v_pointer += sizeOf(vector3d)
next

printL

printL "Good practice - changing the order in vector3d does not affect result" in 10
dword x_offset  = udt_elementoffset(v(1).x)
dword y_offset  = udt_elementoffset(v(1).y)
dword z_offset  = udt_elementoffset(v(1).z)

printL "x offset: " + x_offset
printL "y offset: " + y_offset
printL "z offset: " + z_offset
printL

v_pointer = varptr(v(1))
for i = 1 to countof(v)
  poke(single, v_pointer + x_offset, 1 * i)
  poke(single, v_pointer + y_offset, 2 * i)
  poke(single, v_pointer + z_offset, 3 * i)
  
  printL v(i).x, v(i).y, v(i).z
  
  v_pointer += sizeOf(vector3d)
next

waitKey
```
*Download as [gloryAndShameOfDotNotation02.tbasic](https://github.com/petrSchreiber/Article-series-User-defined-types/blob/master/02_gloryAndShameOfDotNotation/code/gloryAndShameOfDotNotation02.tbasic)*

You may wonder - why should I care about accessing the fields via pointers at all? Well, there is a reason which might interest you.

While the dotted syntax is very clear to follow, and de-facto standard even in other languages, it comes at performance price.

Each time a dot is found, a calculation takes place to find the referenced field. The effect gets more serious, the more dots you have in your expression.
This is something many data modeling maniacs do not take in count, bringing them to wonder later.

With thinBasic, this can be completely eliminated with usage of pointers (and you can impress girls by living dangerously).
Have a look at this very synthetic benchmark to demonstrate the time difference:
```thinbasic
uses "console"

type vector3d
  x as single
  y as single
  z As Single
end type

dim v(1000000) as vector3d
long  i

hiResTimer_Init
quad t1, t2

print "Classic dotted aproach: "
t1 = hiResTimer_Get
for i = 1 to countOf(v)
  v(i).x = 1 * i
  v(i).y = 2 * i
  v(i).z = 3 * i
next
t2 = hiResTimer_Get
printL format$((t2 - t1)\1000) + " ms" in 12 ' -- If you wonder, this is forced integer division

'
dword v_pointer
dword x_offset  = udt_elementoffset(v(1).x)
dword y_offset  = udt_elementoffset(v(1).y)
dword z_offset  = udt_elementoffset(v(1).z)
'

print "Virtual variable:       "
t1 = hiResTimer_Get

single x at 0
single y at 0
single z at 0

v_pointer = varptr(v(1))
for i = 1 to countOf(v)
  setAt(x, v_pointer + x_offset)
  setAt(y, v_pointer + y_offset)
  setAt(z, v_pointer + z_offset)
  x = 1 * i
  y = 2 * i
  z = 3 * i
  
  v_pointer += sizeOf(vector3d)
next
t2 = hiResTimer_Get
printL format$((t2 - t1)\1000) + " ms" in 14

print "Poking around:          "
t1 = hiResTimer_Get
v_pointer = varptr(v(1))
for i = 1 to countOf(v)
  poke(single, v_pointer + x_offset, 1 * i)
  poke(single, v_pointer + y_offset, 2 * i)
  poke(single, v_pointer + z_offset, 3 * i)
  
  v_pointer += sizeOf(vector3d)
next
t2 = hiResTimer_Get
printL format$((t2 - t1)\1000) + " ms" in 10

waitKey
```
*Download as [gloryAndShameOfDotNotation03.tbasic](https://github.com/petrSchreiber/Article-series-User-defined-types/blob/master/02_gloryAndShameOfDotNotation/code/gloryAndShameOfDotNotation03.tbasic)*

##Conclusion
User defined types provide great flexibility for data modeling, exposed via dot notation, in line with industry expected approach. While this syntactic sugar offers great readability, it has its performance impact.

The article demonstrates possible approach to overcome this limitation for cases when performance is needed. While the approach is pointer based, it can be made safer by using *UDT_ElementOffset* function, ensuring proper field addressing.
