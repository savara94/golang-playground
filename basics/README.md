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

Thanks to embedding, we can refer to the names at the leaves of the implicit tree without
giving the intervening names:

```
var w Wheel
w.X = 8 // equivalent to w.Circle.Point.X = 8
w.Y = 8 // equivalent to w.Circle.Point.Y = 8
w.Radius = 5 // equivalent to w.Circle.Radius = 5
w.Spokes = 20
```


# 5. Functions

The parameter list specifies the names and types of the function’s parameters, which are the
local variables whose values or arguments are supplied by the caller. The result list specifies
the types of the values that the function returns. If the function returns one unnamed result
or no results at all, parentheses are optional and usually omitted. Leaving off the result list
entirely declares a function that does not return any value and is called only for its effects.

```
func name(parameter-list) (result-list) {
    body
}
```

The type of a function is sometimes called its signature. Two functions have the same type or
signature if they have the same sequence of parameter types and the same sequence of result types.


## 5.1 defer

A defer statement is often used with paired operations like open and close, connect and disconnect, or lock and unlock to ensure that resources are released in all cases, no matter how complex the control flow.

The function and argument expressions are evaluated when the statement is executed, but the actual call is deferred until the function that contains the defer statement has finished, whether normally, by executing a return statement or falling off the end, or
abnormally, by panicking.

Any number of calls may be deferred; they are executed in the reverse of the order in which they were deferred.

Deferred functions run after return statements have updated the function’s result variables.

## 5.2 panic

Go’s type system catches many mistakes at compile time, but others, like an out-of-bounds
array access or nil pointer dereference, require checks at run time. When the Go runtime
detects these mistakes, it panics.

During a typical panic, normal execution stops, all deferred function calls in that goroutine are
executed, and the program crashes with a log message. This log message includes the panic
value, which is usually an error message of some sort, and, for each goroutine, a stack trace 
showing the stack of function calls that were active at the time of the panic.

Although Go’s panic mechanism resembles exceptions in other languages, the situations in
which panic is used are quite different. Since a panic causes the program to crash, it is general
ly used for grave errors, such as a logical inconsistenc  in the program; diligent programmers
consider any crash to be proof of a bug in their code. In a robust program, ‘‘expected’’
errors, the kind that arise from incorrect input, misconfiguration, or failing I/O, should be
handled gracefully; they are best dealt with using error values.

## 5.3 recover

If the built-in recover function is called within a deferred function and the function containing
the defer statement is panicking, recover ends the current state of panic and returns the
panic value. The function that was panicking does not continue where it left off but returns
normally. If recover is called at any other time, it has no effect and returns nil.

# 6. Methods

Object is simply a value or variable that has methods, and a method is a function associated with a particular type.

A method is declared with a variant of the ordinary function declaration in which an extra parameter appears before the function name.

```
package geometry

import "math"

type Point struct{ X, Y float64 }

// traditional function
func Distance(p, q Point) float64 {
    return math.Hypot(q.X-p.X, q.Y-p.Y)
}

// same thing, but as a method of the Point type
func (p Point) Distance(q Point) float64 {
    return math.Hypot(q.X-p.X, q.Y-p.Y)
}
```

Because calling a function makes a copy of each argument value, if a function needs to update a variable, or if an argument is so large that we wish to avoid copying it, we must pass the address of the variable using a pointer.

In a realistic program, convention dictates that if any method of Point has a pointer receiver, then all methods of Point should have a pointer receiver, even ones that don’t strictly need it.

# 7. Interfaces

Interface types express generalizations or abstractions about the behaviors of other types. By generalizing, interfaces let us write functions that are more flexible and adaptable because they are not tied to the details of one particular implementation.

Many object-oriented languages have some notion of interfaces, but what makes Go’s interfaces so distinctive is that they are satisfied implicitly. 

When you have a value of an interface type, you know nothing about what it is; you know only what it can do, or more precisely, what behaviors are provided by its methods.

