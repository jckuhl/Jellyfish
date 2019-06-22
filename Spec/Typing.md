# Typing

Typing refers to giving type annotations to a variable.  It is required when type annotiation cannot be inferred by the compiler and preferred when type annotation is not obvious to the programmer.

Types are defined types, primitive types and classes, or composite types.  They follow a variable or field identifier, separated by a colon.

```
final PI:double = 3.1415;
```

## Type Declaration

A Type Declaration is akin to an interface, it declares an object's shape.  Fields and methods in a type must be implemented in any object that implements the Type.

Example:

```
Type Person {
    name: String;
    age: Number;
    greeting(): void;
}

const Tom:Person = {
    name: "Tom";
    age: 23;
    greeting() {
        stdio.println(f"Hello, I am {this.name}");
    }
}
```

Note how the `Tom` object has the same fields and methods as the `Person` type.

> As convention all types should be capitalized, except for primitives.  Wrappers like `Int` are objects whereas `int` is a primitive.  `Object` is not primitive and therefore capitalized, same goes for `String`.

## Special Types

There are several special built in types
- `Any`
- `Null`
- `Never`
- `Void`
- `Truthy`
- `Falsy`

`Any` means the value may have any type.  The programmer is opting out of type safety.  This is to be avoided but occasionally necessary

```
// imagine hitting a URL where we don't have control over what shape the incoming data is:
final data: Any = HTTPRequest.get(someURL);

// Any is for situations like this.
```

`Null` simply means there is no value.  It is different from `Void` because a null value may be returned from a function.  All types, unless explicitly set to `!Null` (not null) can be set to the null value.  Null as a value is lowercased, null as a type is capitalized.

```
var x: int = null;  // perfectly valid
```

`Never` indicates the function never returns because it either quits the program, contains an unbroken while loop or throws an error when called.  Never is only for return types.

```
func quit(): never {
    System.exit(0); //syntax is still undetermined
}
```

`Void` is also primarily for return types.  It indicates that the function doesn't return a value.  This is distinct from `Null` in that `Null` is not just for return types and `Null` is, counterintuitively, a value.  (You can set a variable to `null` but not `void`);

```
func hello(name:String): void {
    stdio.println(f"Hello {name}");
}
```

`Truthy` means the variable of any type that can be converted to truthy/falsy.  If a value is not `null`, `0`, empty string, then it is truthy.  It's any value that, when coerced to `Boolean` will return true.

Here's a division function that doesn't allow division by zero:

```
func divide(a:int, b:int+Truthy): double {
    return a / b;
}

divide(3, 0); // compile time error
```

`Falsy`  Opposite of Truthy, equivalent to `!Truthy`

if(:object)

> Be careful however, sometimes `0` is a desired numeric value, so avoid truthy/falsy if this is the case.  Truthy/falsy is only relevant where type coersion is allowed.

## Composite Types

Composite types declare what type of values a variable can have by combining types.  The operators are all boolean, with the exception of optional:

- `+` Union (And)
- `|` Or
- `!` Not
- `?` Optional

`+` is the most straight forward, it simply states that a variable most conform to both types.

In the following example, the object literal must implement both the `Supervisor` and the `Employee` types:

```
const manager:SuperVisor+Employee = { ... }
```

`|` allows a type to be one type _or_ another type.

Here a call is made with a fictional HTTPRequest object (not a part of JF spec) that might return either a JSONObject or an Error.

```
const data:JSONObject | Error = HTTPRequest.get(url);
```

>  While technically legal, using `|` with `Null` is redundant, `var x: int | Null`, because all types, unless specifically declared `!Null` can be set to null.

`!` is Not.  It means a variable _cannot_ be of that particular type.  Mostly used for Null in conjunction with `+`.

```
// here's a string that won't accept a null value:
var notNullString: String + !Null = "Hello";
// note it must be initialized because it can't be null.
```

`?` is for optional types in a function parameter list

```
func readFile(fileName: string, encoding:?string): Buffer {
    return stdio.open(fileName);   // if encoding is not provided, it defaults to UTF-8
}

readFile("./data.txt");
```

If a value is provided for the above function, it must be a string, but the parameter may be omitted as well.  You may also give it a default value, but that makes the `?` redundant as default values in parameter lists are by default optional and don't require a `?` character.  The character may be provided if the programmer wishes, however.

## typedef

A _typedef_ is a way to alias a composite type.