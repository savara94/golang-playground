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