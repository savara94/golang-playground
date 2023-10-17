# 1. Names

Predeclared names come in form of constants, type and built-in functions. These are *not reserved* so they may be used in declarations.

## 1.1 Constants

true
false
iota == ?
nil

## 1.2 Types

```
break default func interface select
case defer go map struct
chan else goto package switch
const fallthrough if range type
continue for import return var
```

## 1.3 Built-in Functions

```
make len cap new append copy close delete
complex real imag
panic recover
```

If an entity is declared inside a function it is *local* to that function. If declared outside of function, it is visible within that package. If name starts with capital letter, then it is *exported* -- it visible to other packages that use that package.

Package names should always be lower-case.

# 2. Declarations

You can have four major kinds of declaration:

1) var
2) const
3) type
4) func

## 2.1 Vars

Each declaration has the general form:

`var name type = expression`

Either *type* or *expression* can be omitted.
If expression is omitted, variable is initialized to zero value of that type -- meaning:

1) for booleans -> false
2) for numerics -> 0
3) for strings -> ""
4) for interfaces -> nil
5) for reference types -> nil

Package level variables are initialized before *main* starts. Local variables are initialized when as they are being encountered inside a function.

In Go, there is no such thing as a uninitialized variable.

There is other way to declare variables - through short initialization:

`name := expression`

*=* and *:=* should not be mixed for one another!

There is one subtle thing though - this acts as an assignment for variables that are already declared.

Comparing to other languages, it is perfectly safe to return address that is local to some function.

*new* function creates unnamed variable of type T, initializes zero-value of T and returns a T*.

Life-time of variables is defined as:

1) package-level variables are alive for execution of a whole program
2) local variables are created when they are encountered in code and are alive until they become unreachable

A compiler may choose to allocate local variables on the heap or on the stack ???

## 2.2 Type Declarations

A type declarat ion defines a new named type that has the same underlying type as an existing
type. The named type provides a way to separate different and perhaps incompatible uses of
the underlying type so that they can’t be mixed unintentionally.

`type name underlying_type`


```
package tempconv

import "fmt"

type Celsius float64
type Fahrenheit float64

const (
    AbsoluteZeroC Celsius = -273.15
    FreezingC Celsius = 0
    BoilingC Celsius = 100
)

func CToF(c Celsius) Fahrenheit { return Fahrenheit(c*9/5 + 32) }
func FToC(f Fahrenheit) Celsius { return Celsius((f - 32) * 5 / 9) }
```

In this very example, even though *Celsius* and *Fahrenheit* do have the same underlying type they cannot be compared or combined in arithmetic expressions. Programmer is forced to do explicit conversions to avoid implicit mistakes.

A conversion from one type to another is allowed if:

1) both have same underlying type, or
2) both are unnamed pointer types that point to variables of the same underlying type

Conversions *may* change the internal representation of the data (example of float <-> int).

## 2.3 Packages and files

Package = library, module in other language providing modularity, separate compilation and reuse. Consists of one or more .go files.

Each package serves as a separate namespace for its declarations. So you can have two functions that have same name, but if you intend to use them together, you would need to fully qualify them.

Packages let us hide information by controlling which which names are *exported*. This is being done by starting such identifier with an upper-case letter.

Package level names are accessible in all files belonging to that package like they are written in a single file.

Package initialization begins by initializing package-level variables in order in which they are declared, except that dependencies are resolved first.

```
var a = b + c // a initialized third, to 3
var b = f() // b initialized second, to 2, by calling f
var c = 1 // c initialized first, to 1
func f() int { return c + 1 }
```

You can also specify `func init {...}` that will initialize the package.

# 3. Data types

Go's types fall into four categories:

1) basic types
2) aggregate types
3) reference types
4) interface types

Basic types include numbers, strings and booleans.
Aggregate types include arrays and structs.
Reference types include pointers, slices, maps, functions and channels.

## 3.1 Numbers

1) int/uint = most efficient size signed/unsigned on a particular platform
2) rune = synonym for int32, used within Unicode context
3) byte = uint8 but indicates rather raw data than a small numeric quantity

## 3.2 Strings

String is a *immutable* sequence of bytes.

Built-in *len* function returns the number of bytes (not runes) in a string.

Substring operation has a format `s[i:j]` where result contains j-i bytes. If index is out of bounds a panic results.

## 3.3 Arrays

Array is a fixed-length sequence of zero or more elemnents of a particular type. Arrays are passed by *value*.

## 3.4 Slices

Slices are variable-length sequences whose elements all have the same type.
Slice has three components: pointer, length and a capacity.
Slicing beyond cap(s) causes a panic, but slicing beyond len(s) extends the slice so the result may be longer than original.
Only legal slice comparison is against nil.
Nil slice has no underlying array -- but there are slices that have capacity and length of zero that are not nil.

## 3.5 Maps

Map is a reference to a hash table. Keys must be comparable using ==.
Storing to a nil map causes panic.

## 3.6 Structs

A struct is an aggregate data type that groups together zero or more named values of arbitrary types as a single entity.
Each value is called a *field*.

The name of a struct field is exported if it begins with a capital letter; this is Go’s main access control mechanism.
A struct type may contain a mixture of exported and unexported fields.

The zero value for a struct is composed of the zero values of each of its fields.
If all the fields of a struct are comparable, the struct itself is comparable. Comparable struct types, like other comparable types, may be used as the key type of a map.

### 3.6.1 Struct embedding

Go lets us declare a field with a type but no name; such fields are called *anonymous fields*.
The type of the field must be a named type or a pointer to a named type.

```
type Point struct {
    X, Y int
}

type Circle struct {
    Center Point
    Radius int
}

type Wheel struct {
    Circle Circle
    Spokes int
}
```