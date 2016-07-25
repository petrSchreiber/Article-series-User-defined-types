#User defined types #1: Introduction
*Target audience: beginner, applicable version: 1.9.16.16 and newer*

ThinBASIC is a computer language with roots set in BASIC. The [original](en.wikipedia.org/wiki/Dartmouth_BASIC) BASIC did not ask the user to declare any type for the [variables](www.thinbasic.com/public/products/thinBasic/help/html/variables.htm), it simply stored them as a number using [30 bits](www.dartmouth.edu/basicfifty/commands.html) of precision.

Many of the modern computer languages (Lua, Python, Ruby...) try to mimic this design by hiding the internal variable storage details from the programmer. This approach has the clear advantage of keeping things simple. Number is a number, text is a text. Programmer focuses on the problem instead of implementation. So far so good.

The possible controversy of this approach starts to appear once you realize the program needs to run on physical device whose resources are limited. The mentioned approach of variable complexity hiding also poses two issues as well for people who would start with such a language as with their first programming language:

* the correlation between program memory usage and variables is unclear
* performance characteristics can vary surprisingly, as the language switches the backends and performs the memory reallocations

ThinBASIC takes a different route.

##Numeric data types
With our interpreter, the choice of numeric [data type](www.thinbasic.com/public/products/thinBasic/help/html/numericvariables.htm) is responsibility of the programmer. While the behavior of other solutions could be mimicked by simply using the *Number* data type, which poses the ability to hold the largest range of values thanks to 80bits of precision, once we move from playful stage to work on real world solutions, using specific types can bring great efficiency advantages.

ThinBASIC, like some other programming languages, offers two kinds of numeric types: integer and floating point. Integer types should be the type of choice whenever you use whole numbers. They are further divided to signed and unsigned.

In case you can afford it memory wise and the range of values is enough for your task, the best size/performance ratio in thinBasic is provided by the *Long* data type, which can be also referenced by its alias *Int32*. It is is signed, 32-bit integer. This is the reason why you may see *Long* being used even in cases, where you would expect *DWord* (or *UInt32*) otherwise. For example as counters in *for* loops, which usually go from 1 up.

##String data types
Similar level of fine control is given to *String* data types. While *String* itself will do fine in most of the cases, it has its drawbacks. Did you know, for example, that each time you assign value, append or otherwise alter the length, a memory reallocation is performed?
```thinbasic
uses "console"

string myText

printL "Variable address: | Data address:"
printMemoryInfo(myText)

myText = "Earth"
printMemoryInfo(myText)

myText = "Venus" ' Same length, no change
PrintMemoryInfo(myText)

myText = "Earth and Moon"
printMemoryInfo(myText)
waitkey

function printMemoryInfo(byRef stringVariable as string)
  string stringVariableAddress = hex$(varPtr(stringVariable))
  string stringDataAddress     = hex$(strPtr(stringVariable))
  
  printL lSet$(stringVariableAddress, 20 using " ") +
         lSet$(stringDataAddress, 20 using " ") +
         stringVariable  
end function
```
*Download as [introduction01.tbasic](https://github.com/petrSchreiber/Article-series-User-defined-types/blob/master/01_introduction/code/introduction01.tbasic)*

This is the price you pay for *String* having virtually unlimited length. If this is unacceptable for you, performance wise, you can address the issue via 2 approaches.

The first one is more strict and means you allocate the maximum string size ahead. You can do so during the declaration:
```thinbasic
uses "console"

dim myText as string * 14

printL "Variable address: | Data address: | Content:"
printMemoryInfo(myText)

myText = "Earth"
printMemoryInfo(myText)

myText = "Venus" ' Same length, no change
printMemoryInfo(myText)

myText = "Earth and Moon"
PrintMemoryInfo(myText)
waitkey

function PrintMemoryInfo(byRef stringVariable as string)
  string stringVariableAddress = hex$(varPtr(stringVariable))
  string stringDataAddress     = hex$(strPtr(stringVariable))
  
  printL lSet$(stringVariableAddress, 20 using " ") +
         lSet$(stringDataAddress, 20 using " ") +
         stringVariable  
end function
```
*Download as [introduction02.tbasic](https://github.com/petrSchreiber/Article-series-User-defined-types/blob/master/01_introduction/code/introduction02.tbasic)*

Of course, it is not possible to anticipate the maximum size in all the scenarios, and that is where more complex systems, such as string builders come to play.
I prepared one such for thinBasic, and you can [download it](www.thinbasic.com/community/showthread.php?12447-StringBuilder-for-ThinBASIC-GitHub&highlight=string+builder) and use in your projects.

##User defined types
Imagine a situation. You are about to create a game and you need to store data about a ship cruising through the space. What do we need - position, rotation, maybe health, shields and ammo, right? The data model of this could end up as something like:
```thinbasic
number positionX, positionY, rotation, health, shields, ammo
```
Would this work? Sure. Now make it for two players:
```thinbasic
number positionX(2), positionY(2), rotation(2), health(2), shields(2), ammo(2)
```
Eh... nice try. But what about wrapping it in more descriptive user defined type?

```thinbasic
type Starship
  positionX as number
  positionY as number

  rotation  as number

  health    as number
  shields   as number

  ammo      as number
end type
```
With such a declaration, you can prepare two ships with same set of properites as easily as:
```thinbasic
dim player(2) as Starship
```
How big is our single ship in memory then?
```thinbasic
printL sizeOf(Starship)
```
60 bytes. 60 bytes! Really?!

This is where we are hitting what we talked in the beginning - not every property needs to be 80bit, right? In this case, none of them.

The transformational values as well as health and shields can be easily 32bit and for ammo we can restrict it to something like 16bit unsigned integer.

Where do these estimations come from? From simply looking up the [table](www.thinbasic.com/public/products/thinBasic/help/html/numericvariables.htm) and knowing what the fields will be used for.

So after facelift:
```thinbasic
type Starship
  positionX as single
  positionY as single
  
  rotation  as single
  
  health    as single
  shields   as single
  
  ammo      as word
end type
```

...and voila! We are suddenly at 22 bytes per ship, that is almost 1/3 of the original size.

##Conclusion
We are living in 21st century, we have computers with huge amounts of RAM. That does not mean we should waste it, remember me the next time you will shutdown your browser from task manager Be proud to be in control of the situation, and use the thinBasic language resources to your advantage.

Today we learned that user defined types can be of great help to logically group your variables together, to easily create copies of data models and measure their memory footprint. Stay tuned for more, as there is a lot more of exciting stuff to learn!
