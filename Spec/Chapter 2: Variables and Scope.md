# Variables

## Declaration

There are two types of variables:

Instance variables (known from here out as _fields_) and block-scope variables (known from here on out simply as _variables_).  Fields are associated with a class or object and will be discussed in later chapters.  Variables are containers for values declared at a global or block scope, depending on where they're defined.

Variables are declared with three key words:

- `var`
- `const`
- `final`

`var` is a mutable variable.  It may be reassigned and object properties may be modified.
`const` is non-reassignable.  However, objects may be modified.
`final` means the object is totally immutable, the variable can be neither reasigned or modified.

`const` and `final` variables may be reassigned only once, and that's only from undefined to having a value.

For example, this is fine:

```
final PI:double;
PI = 3.1415;
```

Programmers are adviced to prefer `final` over `const` and `const` over `var`.

## Typing Variables

All variables that are not initialized must be given a type.  In addition, variables that cannot have their types determined at compile time must be given a type.  Furthermore, programmers are suggested to give variables a type when it isn't obvious by reading.

A variable is initialized with the assignment operator (`=`) and a value which is a Jellyfish expression

```
[Declaration] [Identifier]:[Type] = [Value];
final A:int = 3;
```

Type annotation follows the identifier with a colon or a double colon.  The single colon indicates a Type that the value must conform to, a double colon indicates inheritence and is only valid if the value is an object.  We will discuss the double colon later.

```
final PI:double = 3.1415;
```

Here we've declared PI as a `final` and declared its type to be `double`.  Types can be types, classes and combinations of types.  Types and how to create and combine them are discussed in the Types chapter.

Technically because PI is given a value that is a literal double, the type information is not necessary and inference is used.  If the compiler can infer the type, type inference can be used with the type inference assignment operator, which is `:=`

```
final PI := 3.1415;
```

> A colon before any operator indicates to Jellyfish that type inference and coercion is desired.  Inference and coercion must be _opted in_ to use.

## Identifiers

Identifiers can use any alphanumeric UTF-8 characters, dollar sign or underscore.  They must start with an alphabetical character, a dollar sign or an underscore, but cannot start with a number.

> The dollar sign is for convention sake as many programmers use it for various reasons.

Identifiers are case sensitive.  `var A` is different than `var a`.  Type information is not a part of a variable's signiture and shadowing is not allowed.  Declaring a variable, unless it is declared with `var`, multiple times with the same identifier, even with different types, will not compile.

## Scoping

There are three levels of scoping.   Block, closure, and global.  Variables declared in a function are block scoped.  This means they live from the moment they're declared to the closing of the curly bracer that defines their block.

```
{
    var x := 3;
}
stdio.print(x);     //Will not compile, x is out of scope
```

Variables are available in lower scopes, but not larger ones:

```
{
    var x := 3;
    {
        x:= 4;
    }

}
```

Global scope variables are technically block scoped, but because they're declared outside of any function or block, their scope is the entire program.  This is frowned upon, but available if needed.

Closure scope variables are variables accessible in a function created by an outer function.  This is explained in greater detail on the section on functions.  A brief example however:

```
func makeAdder(x:int):(int) => int {
    return (y:int) => x + y;
}

final addFive: (int) => int = makeAdder(5);
stdio.print(addFive(10));   //prints 15
```

Here the parameter `x` is held by the internal anonymous lambda function, even once the makeAdder function's execution context is destroyed.  `x` will exist with whatever value the outer function gave it, whenever the inner function is called.

Again, more on closures in the section on functions.

