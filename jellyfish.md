# Jellyfish

Because I called my . . . company? motto?  slogan? trademark? . . . project breakpoint, which has the initials PB, and so I thought of PBJ, and Jelly made me think of a song on the Suicide Squad album that mentions peanut butter and jellyfish . . . look, I have weird brain, okay?

Note that the following is a brainstorm and if I ever implement this language, ideas will be added/removed as necessary.

- [Paradigm](##Paradigm)
- [Typing](##Typing)
- [Control Structures](##Control%20Structures)

## Paradigm

Functional (supportive but not strictly)/ Classical OOP (again, supportive but not strictly)

Jellyfish supports functional programming but isn't pure itself

Primary objectives of Jellyfish are flexibility and readability.  The programmer can determine how strongly he wants his code to be typed, how "pure" he wants to be with functional programming, and when and how he desires to use objects, if at all.  The type system, with inference, `any` and conditional typing gives him more flexibility while maintaining error checking by both editor and compiler.

This is in part a response to Haskell and Java, both of which I think are obnoxiously rigid, the first saying "You _have_ to be functional!" and the second saying "You _have_ to be OOP."  Jellyfish aims to encourage both and require neither.  Most native methods are functional and immutable, but mutations and side effects are allowed.  Currying, partials, closures, etc, are all implemented and available as well.  `final`, `const` and `let` give the programmer control over mutability.  Most things in Jellyfish are objects, but there's no necessity to wrap all code into unnecessary classes.

Furthermore, the code is meant to be readable.  Conditional operators all have english equivalence.  The `switch` construct is tossed for a more flexible `match` construct.

JIT Compiled.  Script is entered at nearest `main()` function.  There must be at least one main.  There can be more than one `main()` in an application, but there cannot be more than one main per module.  The `main()` that is run depends on the file that gets executed.  Only files with a `main()` can be run

```
func main(...args) {
    final fact := factorial(args[1]);
    try {
        stdout.println(f"Factorial of {args[1]} is {fact}");
    } except Exception as e {
        stdout.println(e.message);
    } finally {
        stdout.println("Have a good day!");
    }
}

func factorial(x: int): int {
    if(x <= 0) {
        throw Exception("Value must be positive!");
    }
    if(x == 1) {
        return 1;
    } else {
        return x * factorial(x - 1);
    }
}
```

Jellyfish is inspired in part by:

- Java
- TypeScript
- Python

## Typing

#### Static/Inferred

Two types of variables:
* Local - declared in some scope
* Property - proprety of an object

Declaration:
* `let` - mutable
* `const` - can't be reassigned, mutable
* `final` - can't be reassigned or mutated

Note: `final` and `const` do the same for primitives, they're different for objects like a dict or a class.  A dict, for example, marked `const` can't have it's reference reassigned, but can have it's key/value pairs mutated.  A dict marked `final` can be neither reassigned or mutated

Type is declared using semicolon after identifier.

Example:

    let a: int = 3;
    const b: string = 'hello';
    final word: dict<string, string> = { polymorphism: 'of having many forms' };

Types can be combined with operators

- `|` or: this type or that type
    - `let x: int | float;` x can be int _or_ float
- `+` and: this type and that type
    - `let person: Manager + Employee;` person must implement both Manager and Employee
- `!` not: can't be of this type, another type must be declared or inferred along with it:
    - `let data: !null := someHTTPmethod();` infers type from `someHTTPmethod()`
    - `let name: string + !null;`
    - `let decimal: number + !int;`

If type can be inferred from its value, `:=` is used.

Example:

    let a := 3;
    const b := 'hello'
    final word := { polymorphism: 'of having many forms' };
    //note that in the case of the dict, the subtypes are
    //determined by the values within the value.

`any` can be used if the type is unknown at compile time

    let a: any;

> `any` is NOT implicit.  If the compiler cannot infer the type or the type is not explicitly declared, the compiler will whine.  `any` _must_ be explicitly declared

`final` and `const` must be initialized when declared for lexical variables, but not for object properties.  `final` or `const` properties can be uninitialized and then initialized later, but only once.

`undefined` is a value assigned to any uninitizalized variable.  It is not meant to be assigned by the programmer.  To indicate that a variable has no value, a programmer should use `null`.  `null` can be assigned to any variable, regardless of type.

`void` indicates the lack of a return type.  Applying `void` as a type to a variable is valid and will compile, but is also utterly useless as that variable won't be allowed to have a value.  Unless you want to alias `null`?

> Null is the only value that ignores the type system as all variables can be declared `null` to void them.  You can guard against this by using `!null` as a type (`let x: !null + int`)

#### Type Aliasing

Type Aliasing can be used to combine types and rename them:

Example:

```
final data: string + !null;
```

can be type aliased with the `typealias` command

```
typealias stringNotNull = string + !null;
final data: stringNotNull;
```

`typealias` is block scoped.

Primitives:
* int - integer
* float - float
* double - more precise decimal
* bool - boolean true/false
* char - char, string is a built in class constructed from char
* function - function, functions are first class citizens
* object - object

Assignment, whether `=` or `:=`, also returns the value being assigned.  The assignment itself is an expression.  Chaining assignments are therefore valid, as is using assignments in expressions:

    a := b := 2 //a == 2, b == 2
    a = (b := 2) + 1 //a == 3, b == 2

Generally though its better to avoid this as its confusing code

### Scoping

Scopes are based on block `{}`.

```
// global/file scope
func doSomething() {
    // scope 1
    if(condition) {
        // scope two
    } else {
        // scope three (separate from two)
        return func() {
            // closure scope
        }
    }
}
```

A variable declared or object instanciated is given the most minimum scope, block-scoped.  In the above, a variable declared in scope two, exists only in scope two, and won't even be available in the `else` block.

A variable in scope three is available in the closure scope, even after `doSomething` returns.

Variables in the global scope are only available in that specific file unless specifically imported (see [Modules](##Modules)).  If you import the file it exists in, so long as you don't use the wildcard `*`, it won't be available.  Also, if the module explicitly uses `export` and does _not_ export that specific variable, it won't be available, nor can it be imported.

### Char

char can be integer or a single character.  If a char is an integer, it is the ASCI equivalent

    let a:char = 65;    // a == 'A', note explicit casting is required or the compiler would treat a as an int
    let a := 'a'        // a == 'a', but can be cast to int, let _a: int = (int) a, _a == 97;

    //inferred casting:

    let _a: int = a // _a = 97

### Number

All numbers, float, double, int, etc, inherit from number, to allow for polymorphism.

For example:

    func add(a: number, b: number): number {
        return a + b
    }

can define a function that takes any number where as:

    func add(a: int, b: int): int {
        return a + b
    }

Only accepts integers.

Numbers can be of any radix between 2 and 16.

Binary, Octal and Hex have prefexes `0b`, `0` and `0x` respectively.  Invalid numbers throw errors:

- `08` - error (7 is the final digit!)
- `0b2` - error (1 is the final digit!)
- `0xG` - error (F is the final digit!)

Static function on Number class converts from one type to another

`Number.convert(number, radix)`

`Number.convert(2, 2)` returns `010`

For readability, Number also contains constants with number types

`Number.convert(16, Number.HEX)` returns `0x10`

The same function can do the reverse, just set radix to 10.

`Number.convert(0x10, 10)` returns `16`
`Number.convert(0x10, Number.DECIMAL)` returns `16`

Because it is a public static function, it's available on all Number subclasses, but there's no difference between calling it on `Number` or on a subclass.

`Float.convert(0x10, Float.DECIMAL)` returns `16`

`number.toString(places)` method will also control precision.  By default, omitting the number of decimal places will show all the decimal places, or `.0` if there are no decimal places after the integer.  Putting in an integer value gives you the number of decimal places to show.  `toString()` will round up by mathematical rounding rules

`Number.PI.toString(4)` returns `3.1415`.
`12.3244932873988923.toString(0)` returns `12`

> toString() is immutable, it produces a string object, it does not overwrite the calling number.
> the decimal point will be interpreted as a dot operator if the following token is not a digit.

`number.format(locale)` will write a string with commas and decimal places as per local custums.

`334209.329.format('us')` returns `334,209.329` as a string
`334209.329.format('eu')` returns `334.209,329` as a string
`number.currency(countryCode)` will convert it to a currency according to country, defaults to US if ommitted, because USD is most widespread currency basis (go us I guess.)  Includes rounding

`33.234.currency()` returns `$33.23`

Numbers may have commas and underscores, so long as the number does not begin or end with a comma or underscore, and so long as the comma/underscore isn't next to the decimal symbol.

### Mathematical operators:

> ALL OPERATORS MUST BE TRANSITIVE, which should go without saying _PHP_

* +, -, *, /, //
* +=, -=, *=, /=
* **, **= 
    * exponential
    * example: x **= 2 means 'take the value of x to the power of 2 and put the result back into x'
    * also, for readability, keyword `tothe`, example: 2 `tothe` 2

Division will always use the types its given.  Double divided by double gets a double.  Int divided by int gets an int (decimal is truncated without rounding.)  Integer division, `//` will coerce non-int numbers into int and then divide, by truncating without rounding.

All of the above can be overwritten by `_operator` for operator overloading

None of the above support type coercion.

To 'opt in' to type coercion, append a `:` before the operator.  Coercion with division will coerce the smaller decimal to a larger precision.

`+` and `*` are overloaded for strings and lists.  `+` is concatenation for strings and lists.  `*` is repeat for strings.

Example:

```
final hello := 'hello';
final number := 10;

hello + number
```

Above throws an error.  A number cannot be added to a string.  This can be fixed in three ways:

```
hello + (string) number;
'hello10'
```

or

```
hello :+ number;
// 'hello10'
```

or
```
hello + f'{number}';
```

### Relational operators

* :== value equality, checks value, but not type (not using === because :== seems more consistent w/ use of colons)
    * `'2' :== 2` is __true__
    * `'3' :!= 2` is __true__
        * or `'3' not :== 2` __true__
* == strict equality, checks value and type
    * `'2' == 2` is __false__
    * `3 != 2` is __true__
        * or `3 not 2`

* `typeof` checks if an object is of a specific type
    * `1 typeof int;` __true__

* `instanceof` checks if an object is an instance of a class
    * `bob instanceof Person;`

* `type()` returns the type of an object
    * `type(1) == 'int';`
    * `type(bob) == 'Person';`
    * `type([1,2]) == 'list';`

Compound relational operators:
- && or `and`: true if both conditions are true
- || or `or`: true if one or both conditions are true
    - | shorthand for using `or` on the same variable
- ^ or `xor`: true only if only one of the conditions are true (XOR)

Unary relational operator:
- ! or `not`

The unary relational operator can also be used in typing, to say a variable can't be of a certain type:

```
let x: int + not null;

function processData(array: Object[] + !null, callback: function): array {
    // . . .
}
```

If the first condition resolves to false in an `and`, the whole condition is short circuited and the second condition is _not_ evaluated (if it calls a function, that function _won't_ be invoked.)

If the first condition of an `or` statement resolves to true, again, the whole condition is short circuited.

Programmer has the responsibility to be aware of shortcircuiting to avoid or take advantage of it.

An example:

```
if(person and person.name == "Jefferson") {
    //...
}
```

This example will shortcircuit if `person` is falsy.  Thus, if person is undefined, an error is avoided in the second condition because it won't check `person.name`

An alternative using optionals:

```
if(person?.name == "Jefferson") {
    //...
}
```

Equivalent to the above.  The ? will shortcircuit the conditional if `person` is falsy

Optionals need only one `?` but the undefined must come before the `?` or it might still throw an error.  An example:

```
if(user?.orders.pending.includes("Effective Java")) {
    // do something
}
```

if `orders` or `pending` is undefined, then a `NullPointer` exception is still thrown.  To avoid it, the ? should be after `pending`:

```
if(user.orders.pending?.includes("Effective Java")) {
    // do something
}
```

Now the whole thing will be false if `users`, `orders`, or `pending` are null or underfiend and no exception is thrown.

If you want the exception to be thrown, wrap in a try/catch or, use `guard`

```
guard(users.orders.pending?.includes("Effective Java")) {
    // do something
} except [Exception] {
    // else is not available, but the stack trace is
    // except clause does not have to explicitly catch an exception
    // nor does it stop flow of execution unless programmer tells it to.
} finally {
    // finally is, and any code here will run whether the guard was successful or not
}
```

Guard/Except is the same syntax as try/except, except guard has a condtional with an optional.  Guard is only necessary if you want the exception, an if/else is sufficient, if all you need is a `false`.

#### String comparison

String comparison strictly compares strings by alphabetical order, based on ASCII code, but reversed to be more in line with conventional thinking

`'a' < 'b'` is false even though 65 is less than 66, because a comes before b in the alphabet.

Any other type of comparison (string length, etc) must be implemented by the programmer.

#### Truthy/Falsy
Using `:==` can return truthy/false

False:
* 0, null, undefined, false

`not` and `!` use truthy/falsy unless in conjunction with `==`

    if(!value)  //truthy
    if(not value)   //truthy
    if(!value == null) //strict
    if(not value == null) //strict
    if(!value :== null) //truthy

## Control Structures

Blocks: Declared by { }

### Decision Structures:

A condition in any expression that evaluates to a boolean, or any truthy/falsy value.

If (checks if a condition is true):

    if(condition) {

    } else unless/if(condition) {

    } else {

    }

Unless (checks if a condition is false)

    unless(condition) {
        
    } else unless/if (condition) {

    } else {

    }

Can be used in a statement:

    return x unless x == null

    return x if x == 3

    x = y if y = 3 else 7

    x = y unless y != 3 else 7

Not operators are `!` or `not`; both are equivalent, stylistic programmer choice.

Decision structures have redundancies, allowing programmer to decide on readability, if he prefers `if not` to `unless`.  Its a judgement call.

A single `|` within a condition is a shorthand:

```
if(x == 2 | 3)
```
is equivalent to

```
if(x == 2 || x == 3)
```

`x == 2 | 3` should be treated as one unit:

```
if(x == 2 | 3 && y == 4)
```

is the same as

```
if((x == 2 || x == 3) && y == 4)
```

**note the parenthesis:**  shorthand is always evaluated first


Finally, the switch statement:

> __With the fact that object literals behave like associative arrays, and the `match` structure, `switch` is redundant and will probably not be included if I ever make this thing.  Besides, Python does fine without the things.  Never been a fan of switches.  With that said, the following is how I would implement them if I do.__

    switch(condition) {
        case value1 ?

        case value2 ?
            fallthrough;
        default
    }

Switches automatically break.  Using the fall through command causes the next case to be called and will ignore any commands following the fallthrough in that case (causing an unreachable code compile error).  Default is not required, though all cases must be exhausted and the default is required if not all cases can be determined at compile time.

`fallthrough` can also be declared at the top to make it the default behavior in all cases.

    `switch(condition): fallthrough {
        case . . . 
    }

### Matching

    match(x: type) {
        pattern ?
            do something
    }

    match(x: int) {
        x > 21 ?
            return 'old enough to drink';
        x == 21 ?
            return 'just became legal!';
        x < 21 ?
            return 'no alcohol allowed!';
    }

Unlike switches, matches don't check constants, they check conditions and patterns

They work with regex

    final phoneRegex = r"/^(\+\d{1,2}\s)?\(?\d{3}\)?[\s.-]\d{3}[\s.-]\d{4}$/g"
    match(str: string) {
        phoneRegex ?
            return 'it's a phone number';
        default ?
            return 'not a phone number';
    }

With regex, you can also access the matches

    final phoneRegex = r"/^(\+\d{1,2}\s)?\(?\d{3}\)?[\s.-]\d{3}[\s.-]\d{4}$/g"
    match(str: string) {
        phoneRegex ?
            return phoneRegex.matches(string);
        default ?
            return null;
    }

    //alternatively
    phoneNumber = phoneRegex.matches(string) if phoneRegex.match(string) else null

Matches also have fall through as they match from top to bottom in order.  Unless fallthrough is declared (as with switch) then the match stops at the first match.  If fallthrough is used at the top, then all matches with true conditions will execute.  If fallthrough is used on a pattern, then that pattern and all patterns after, regardless of matching, will execute.

Thus:

```
match(day) {
    "monday" ?
        return "crunches";
    "tuesday" ?
        return "push ups";
    "wednesday" ?
        return "1.5 mile run";
    "thursday" ?
        return "squats";
    "friday" ?
        return "deadlifts";
    "saturday" ?
        fallthrough;
    "sunday" ?
        return "day off";
}
```

In the above, if the `day` matches `saturday`, it will fall through and return `day off`, same thing it would do if it was `sunday`

`return` breaks a fallthrough because it terminates the function execution the switch is in.  If the switch is outside of a function, then `return` is an invalid key word (can only be used in a function that is not a generator.)  Fallthrough is only explicitly called for anyways and is not the default behavior.

`global` will fire all cases that match in order from top down:

```
let x:int = 3
match(x): global {
    3 ?
        // triggered
    typeof(x) == "int" ?
        // also triggered
}
```

In the above, if global was omitted, only the first case would be triggered.

`always` is a case that will always, regardless of whether any matches are made or not, be triggered.

```
match(x) {
    x > 3 ?
        doSomething();
    always ?
        doThisToo();
}
```

Omitting a code block after a case is equivalent to fall through:

```
match(x) {
    pattern1 ?
    pattern2 ?
    pattern3 ?
        doSomething();
}
```

This is useful if you want something to happen so long as it matches at least one of three patterns.

### Loops:

> All loops and iterations automatically create an `index` variable (JF keyword) that can be used to track index position

> The condition of the loop and the first block (excluding children) are the same scope.

While loops:

    while(condition) {
        //executed while the condition is true
    }

Until loops
    until(condion) {
        //executed until the condition is false
    }

Do While/Until

    do {
        // gets executed at least once
    } until(condition);

Like with `if/unless`, the primary difference is semantics and readability.

There are two types of for loops:

Classical For: `for(initialization; condition; expression)`

    for(let i = 0; i < 3; i++) {
        //initialization can be neither final nor const
    }

    //Equivalent while:
    let i = 0;
    while(i < 3) {
        //do stuff
        i++;    //incrementer is always called after code runs
    }

Enhanced For: `for(let item in iterable)`

    for(let value in list) {
        // do something in list
    }

> final or const are invalid here, the looping construct requires reassignment

Enhanced for does not mutate the list.  You can force mutation:

    for(let value in list): mutates {
        // do something to the list
    }

Classical for loop on the other hand can mutate directly.  `mutates` keyword is used to tag other methods to trigger mutating capabilities such as:

    const grades = [55, 67, 88, 93, 91];
    grades.filter(grade => grade < 60): mutates;

    //equivalent to:
    grades = grades.filter(grade => grade < 60);

    //This is tagging, which is similar to
    //decorators/annotations

Finally, the `range(stop, start, step)` object is available for iteration.  Range is a generator that will create a value from (and including) the start parameter (zero if not provided) to and not including the stop.  Step, if provided, indicates how much that generator increases, or decreases, each pass.  Step is 1 if not provided, and will go backwards from the `stop` value to the `start` value if negative.

Example: `range(10, 0, -2)` gives the following values: `8, 6, 4, 2, 0`

Range is a generator (see [Generators](##Generators)) and thus has all the generator functions, like `next()`.  Next is implicitly called in an enhanced for loop.  If the value in range is unnecessary, you may leave it blank with a single underscore:

    for(_ in range(10)) {
        // do something 10 times
    }

The `in` operator is not simply available for generators, it is an operator for all iterables and by default returns whether or not a value is in the iterator.  If used in a looping structure however, it is used for so long as there is a value in the iterable, until we go over all values in the iterable.

`range` and other generators only work in enhanced for loops.  In a while loop, the `in` operator has its default behavior:

    while(let x in array) {
        // loop until x is removed from the array
    }

    until(let x in array) {
        // loop until x is added to the array
    }

    for(let x in array) {
        // loop over every element in the array, call it x for the current pass
    }

## Strings

Strings use ASCII and Unicode

Strings are not primitives, but members of the String class, building an immutable array of primitive `Char`.

String literals are declared by only double quotes as single quotes are reserved for `char` literals.

There are three prefixes you can add to a string to modify their behavior:

- `b` - indicates a byte stream
- `r` - indicates a regex
- `f` - indicates string interpolation
    - Example: `f"{player.name} scored {player.score} points"`

Strings are immutable, so any changes to a string produces a new string.

String methods (and related):
- `string.equals(string)` equivalent to `==`
- `string.length` property, gives number of chars in a string
- `string.toCharArray()` returns an array of characters
- `string.split(delimiter)` splits a string on a delimiter, `null` or empty string returns array of characters, equivalent to `toCharArray()`
- `string.codeAt(index)` returns the ASCII code value at given index
- `static Char.getCodeFrom(value)` static method from `Char` that gives you the character with the given value

## Primitive Wrappers

All primitives (except `null` and `undefined`) have wrapper classes.  These classes have the same name as the primitives, except denoted by a capital title (`Int()` vs `int`), parenthesis can be removed when used as a type (denotes and invokes the constructor).

These classes provide the equivalent to their primitive literals and are simply a means of providing methods based on that class and providing inheritence (all numeric types, `int`, `float`, `double`, inherit from `number`)

    func add(a:number, b:number): number {
        return a + b;
    }

add can take any type of primitive that is a Number.

## Functions

Functions in Jellyfish are functions if they don't belong to an object, methods if they do.

Function declaration:  Return type, identifier, then the parameter block:

    func add(x: int, y: int): int, throws exception {

    }

Multiple functions may have the same name and different implemenetation, so long as params, return and/or exceptions are different.  This is function overloading.

    func add(x: string, y: string): string {
        return x.concat(y);
    }

    func add(x: int, y: int): int {
        return x + y;
    }

`main()` has a specific syntax:

    func main(args: string[]): int {

    }

`main()` must have the args array, even if the args array isn't used.  It can be declared as `...args` or `args: string[]` but it must be an array of strings.  `main()` returns a system exit code in the form of an integer.  However main does not require a return statement, it will be provided by the compiler if not present.  This is the only case where a non-void function can have no return statement.

Functions that return no value return `undefined` and can be declared as `void` or `void` can be omitted.  `void` is merely to enhance readability if the programmer feels it is necessary.  However, `void` will enforce either no return statement, or a return statement that returns `undefined`.  The first is prefered as `undefined` is not a value meant to be assigned by the programmer.

    func noReturn(): void { }
    // same as:
    func noReturn() { }

Functions might also have a `never` return type.  If for example you have a function that throws an error, then it's return type is `never`, because it'll never return.

    func throwError(): never {
        throw Error('oops');
    }

Functions must have a body.  Methods in interfaces or abstract classes however, can be declared abstract without a body.  See the classes section for information.

Alternatively, functions can be declared as variables.  There's no difference between declaring them with const or final.  The `func` keyword is dropped and the function is assigned:

    final add: function = sum(x: int, y: int): int {
        return x + y;
    }

    //or

    final add := sum(x: int, y: int): int {
        return x + y;
    }

Anonymous use arrow notation:

    (x: int, y: int): int => x + y

    function partialAdd(x: num): function {
        return (x: int, y: int): int => x + y
    }

    partialAdd(5)(7); // returns 13

Parenthesis are always necessary, even if param block is missing

    (): void => stdout.println('Hello!');

Lambdas use arrow notation:

    final add := (x: int, y: int) => { return x + y; };

Lambdas do not need to denote return type if it can be inferred from return expression.  Lambdas can also be shortened, if there's only one line of code:

    final add := (x: int, y: int) => x + y;

And also if there's only one parameter:

    final addFive := x: int => x + 5;

IIFE functions use the keyword `invoke` to immediately get called at the point of definition.  `invoke` is optionally followed by parenthesis if there are arguments to pass to the function

    invoke(5) final addFive := x: int => x + 5;
    // will return 10 immediately

Functions are first class objects and may be passed as reference.  Doing so, one simply omits the parenthesis.  If the function requires arguments, the `bind` method is used:

    func add(int: a, int: b): int {
        return a + b;
    }

    final sum: function = add.bind(null, 3, 4);

`bind` takes two arguments, `this` and a list of arguments `varargs`.  This creates a function with the bindings already set.  If the number of `varargs` is less than the number of arguments the function requires, this creates a partial.  If it is more, it throws an error.

A partial:

    func add(int: a, int: b): int {
        return a + b;
    }

    final addSeven: function = add.bind(null, 7);
    let x := addSeven(10);  // returns 17

`apply` behaves exactly as bind does, except that it calls the function immediately (unless it makes a partial.)  It's purpose is to call a method that's on one object, as if a different object is calling it:

    final Person = {
        private name: 'Tom',
        public speak(phrase: sentence) {
            return '${this.name} says: ${sentence}';
        }
    }

    final Dog: any = {
        private name: 'Fido
    }

    Person.speak.apply(Dog, 'woof');

    // Fido says: woof

## Collections

All collections methods do not mutate unless marked with `mutates`  Collections comes from `collections` module.  Collections is a collection of data types other than `Object` and `List`.

### Lists

Lists of items.  Lists that are typed must all be of the same type.  Lists of type any can hold any different value.

There's a difference between the following:

    const a := [1, 2, 3]
    const b any[] = [1, 2, 3]

In that example, `a` is inferred to be a list of integers and therefore must be a list of integers.  `b` however is a list of any and therefore can take any type.  The fact that it happens to be a list of integers is irrelevent.  Note also the `[]` is optional as the list type is inferred.  You can also do:

    const c list<int>

There's also a difference between this:

    const d := [1, 2, 3]
    final e := [1, 2, 3]

`d` cannot be reassigned but `d.append(4):mutates` would be valid.  However `e.append(4):mutates` would be invalid as `e` is immutable.  `final` lists are Jellyfish's tuples.

Declaring a list can be done either with a list literal using square brackets or the List constructor:

    const listA = List() //empty list
    const listB = []

List constructor is overloaded.  Pass in a value and it allocates that number of elements.  Pass in an iterable and it makes a list of that iterable.

Accessing a list uses `[start:stop:step]` notation, the colon is optional.

    const nums = [1,2,3,4,5];
    nums[0] = 1;
    nums[2:3] = [3,4];
    nums[:] = // copy of nums
    nums[3:] = [4, 5];
    nums[-1] = [5];
    nums[::-1] = [5,4,3,2,1];
    nums[::2] = [1,3,5]; // index % 2 == 0

`[start:stop:step]` does not mutate unless appended with mutates:

    const slice = nums[3:4];   //assigns the slice to a new variable
    nums[3:4]:mutates;   //invalid because nums is constant

List comprehensions create a list on the spot.

[expression for element in iterable if/unless condition and/or condition]

Methods for list:

* `append()` adds an element to the end of the list.  Note that adding a list to `append()` does not concatenate, it will nest a list.  Returns new list with appended items

* `flatten()` flattens nested lists at only the first level.  Returns new list flattened
* `deepFlatten()` flattens the list entirely.  Returns new list flattened
* `concat()` adds a list to a list, also done with `+` operator

## Classes

All classes extend Object implicitly (no `inherits` keyword), meaning that all classes have access to the following methods:

> the keyword is changed from `extends` to `inherits` so as to not confuse it with `extension`

* keys() - returns all the properties in a class as a list
* values() - returns all the values of the properties in a list
* entries() - returns all the key/value pairs of values as an array of tuples (`final` lists)

    class Thing inherits OtherThing implements SomeInterface {
        private x: int = 3;         // technically the typing is unnecessary
        static y: string = 'bob';

        // constructor is a function like any other, however it is called "constructor"
        // classes may not have a method called constructor, nor may constructor be overridden
        // constructors don't need a return type as they will always return the object they construct
        constructor ThingConstructor() {
            this.x = 4  // this refers to the object constructed by constructor
        }
    }

`strictly` enforces that a class may only implement an interface and may have no other properties or methods

    class StrictThing strictly implements someInterface {
        //...
    }

Multiple interfaces are allowed, but multiple inheritance is not:

    class MultipleInterfaces implements InterfaceA + InterfaceB {
        //...
    }

Interfaces may be defined in place:

    class Point implements { x: int, y: int } {
        //...
    }

> if the compiler sees the `implements` token with a `{}` following immediately with no identifiers, it will assume the `{}` is an interface and not the class body.  The second set of `{}` is therefore the class body.

Constructors may be omitted if they have no body.  This is the default constructor and it only exists if no other constructor is present.

Constructors who have parameter names equivalent to class property names do not need to explicitly assign the parameters to the properties.

Example

```
class Point {
    private x: int;
    private y: int;
    constructor(x:int, y:int) {}
}
```

is equivalent to:

```
class Point {
    private x: int;
    private y: int;
    constructor(x:int, y:int) {
        this.x = x;
        this.y = y;
    }
}
```

> Curly bracers are still necessary to give the constructor a body.  Constructor has a body after all, we're just not explicitly showing it.  This is just short hand.

### Operator Overloading

Classes may overwrite all operators by implementing overrides of under-the-hood methods.  For example, `+` is an infix method that under the hood calls `_operator+`

The syntax is simple, its underscore + operator + operator symbol.

    class Point { x: int, y: int} {
        private x;  // type is inferred by interface
        private y;

        constructor(x:int = x, y: int = y);
        // omitting the constructor body and assigning the values directly
        // in the parameter block automatically sets the arguments to their
        // properties

        _operator+(rhs: Object, lhs: Object): number {
            // number is the basic type that float, int, double, etc all inherit from
        }
    }

`_operator` is always `public`, and therefore does not need accessibility modifiers.  `rhs` and `lhs` mean _right hand side_ and _left hand side_ respectively, but they can technically be called whatever the programmer wants.  Their types do not necessarily have to be `Object` either, nor does the return type have to be a number (example, adding two arrays produces a concatenated array).

`_operator` is the only identifier where special characters are allowed, but only after the word `operator` and only for operator characters.

| operator | operation     | method         |
|----------|---------------|----------------|
| ==       | equals        | `_operator==()`|
| >=       | greater/equal | `_operator>=()`|


### Underscore Methods

Methods marked with an underscore are special methods that work internally with the compiler, such as `_operator+()`.  These are used with classes to help define specific behaviors.  For example `_next()` can be overwritten to make a class iterable.

### Method overloading

Methods and functions have specific signitures.  In Jellyfish, methods belong to objects while functions are declared with `func` and do not belong to objects.  Signitures look like this for both:

- `func`:

_modifiers_ func _identifier_(_param_: _type_ . . .): _return type_

- `class`:

_modifiers_ _accessibility_ _identifier_(_param_: _type_ . . .): _return type_

If the identifier is the same, but any other part of the signiture is different, then the methods are considered different.

Therefore the following is valid:

    func add(a: int, b: int): int {
        return a + b;
    }

    func add(a: string, b: string): string {
        return a + b;
    }

Though this is redundant because you could also do this:

    func add(a: string | int, b: string | int): string | int {
        return a + b
    }

But the second option has some drawbacks.  First, the implemenation would be exactly the same for both strings and ints (though valid in this language as + is both integer addition and string concatentation.)  Furthermore, nothing guarantees that both parameters will be the same type.  Finally, this may also be more verbose than overloading, especially if more than two types are being allowed.

You may also add or remove as many parameters as you want.  However, mutability cannot be loosened (it may be strengthened) and accessibility cannot be loosened (it also only be strengthened, private may not become public, but public may become private)

The code body of course can be changed to anything at all as it is not a part of the method signiture

### Accessibility:

private, file, module, default, public

* `private` only available to the class it is defined in, excluding even subclasses
* `file` only available in the same file
* `module` only available in the same module
* `default` no keyword, only available in class and subclasses
* `public` available to all

### Inheritance

Subclasses can call and override all the accessible methods of their parent (excluding `private`).  They must also implement all abstract methods and types (if not already implemented by parent classes).

There is no multiple inheritence, but there can be multiple types.

```
Type A {
    doSomething(): int;
}

Type B {
    doSomethingElse(): bool;
}

class C implements A + B {
    // must implement A and B
}
```

### Polymorphism

* Overloading - a function can be defined multiple times with the same name and different implementation in the same scope so long as the parameters and/or return values are different

* Overriding - a method can be overridden by a subclass

Constructor functions:  A class can have multiple constructors and constructors can be marked with accessibility.  Constructors marked with `invoke` will immediately instantiate the object their class defines at the point of declaration (essentially making them object literals that can also be later called for further construction).

    const immediate = class ImmediatelyInstanciated {
        invoke constructor() {
            // constructor is immediately invoked and passed to immediate variable
            // classes may be set equal to a variable
            // if constructor is invoked, variable is equal to object
            // otherwise it is a reference to the class type
            // behaves exactly the same as calling the class name, so its just an alias
        }
    }

Constructors must have names, if there are more than one.  However, if there's only one constructor, the name can be ommitted (it will under the hood be given the name of the class)

    class Thing {
        constructor() {}
    }

    // constructor is given name `Thing` and therefore called by `Thing()`

Constructors may be given all accessibility modifiers, the `invoke` modifier, the `final` modifier.  They may not be `static`.  A `final` constructor may only build one object.  `final` and `invoke` can be combined.

Modifiers can be placed in any order. `final private constructor() {}` is the same as `private final constructor() {}`

`invoke final constructor()` and `final constructor()` are two ways of creating singleton.  The difference being the one with `invoke` is created at the time of definition

Constructors cannot be overridden by subclasses (doesn't make sense anyways)

Calling a constructor does not, of course, require an instantiated object, nor does it require to be accessed from a class, unlike other methods

    class Thing {
        constructor() {}
    }

    // no need for Thing.Thing()
    // just call Thing()

Not all objects need to be classes.  Classes are only necessary if multiple objects of the same type need to be made or if inheritance is desired.  If an object literal will do, the syntax is simple:

    const Person =  {
        private name: 'Tom',
        private age: 27
        public greeting() {
            print('${this.name} says hello')
        }
    }

Literals still have access to accessibility modifiers. In the above `Person.name` is invalid because `name` is `private`

Because `Person` was marked with `const`, the identifier `Person` cannot be reassigned.  However, the object itself can be mutated.  Changing `const` to `final` makes the object immutable as well as unreassignable.

Object literals inherit from only global Object.  If inheritence is desired from an object literal, use `invoke constructor() {}` on a class.

Typing literals:

Types come at the end of the identifier, like with any other variable.  If omitted, the type defaults to `object`, indicating the object can have any shape.  An object typed with `object` can match anything that inherits from `object`


These three are equivalent

    const Person: {} = {
        //...
    }

    const Person: Object = {
        //...
    }

    const Person = {
        //...
    }

Futher examples:

    const Person: { name: String, greeting: Function } = {
        //... object must have at least a string property name
        // and a method called greeting
    }

    const Person: { name: String, greeting: Function }, strict = {
        //... object can only have a string property name
        // and a method called greeting
        // it may have no other properties or methods
    }

    const Person: Employee = {
        //... object must implement Employee type
        // it may have other properties or methods
    }

    const Person: Employee, strict = {
        //... object must implement Employee type
        // it may have no other properties or methods
    }

    const Person: Employee + Manager = {
        // multiple interfaces separated by space
        // it may have other properties or methods other than the types
    }

    const Person: Employee + Manager, strict = {
        // multiple interfaces separated by space
        // it may have no other properties or methods other than the types
    }

Object literals may even be empty: `{}`

### Static

`static` marks a method or property as belonging to a class rather than an instanciated object.  Static properties can be called on instanciated object or their class type, but are always in the context of the class, not the object.  `this` is invalid in static methods.

Here's a deviation from Java

    class Spider() {
        final static legs = 4;
    }

    let webby = Spider();

    print(Spider.legs); // prints 4
    print(webby.legs); // prints 4

    let webby = null

    print(Spider.legs); // prints 4
    print(webby.legs); // throws error

Unlike Java, trying to access a static property on an object set to null throws an error because the object's type, as well as value, is invalidated.  `webby` is of type `null`, not `Spider` on the last line.

### Open

Open classes can be extended.  A class that is `final` cannot be extended.  Note, `extention` in this case refers to adding to a current class, not inheritence.  (Maybe change keywords to `class B inherits A implements D + E`?)

```
extension Arrays extends Array {
    // implement methods here
    public shuffle(): array {
        // do something
    }
}
```

Extensions do not inherit or implement, they simply add to a class.  `extension` is merely a definition, it has to be activated by namespacing:

```
using Arrays;

// extension is now valid
final myArray := [1,2,3,4];
myShuffledArray := myArray.shuffle();   //shuffle() is now on array class

stopusing Arrays;

//shuffle is no longer available.
```

If there's a naming conflict due to different extensions having the same method signitures, place the extension name between the object and the method, and alias the extension using `as`.

```
using Arrays as Arr, ArrayTools;

final myArray := [1,2,3,4];
final myShuffledArray := myArray.shuffle();   //ArrayTools shuffle, because alias is omitted
final myShuffledArray2 := myArray.Arr.shuffle();    //Arr shuffle, because alias is used

stopusing Arrays as Arr, ArrayTools;
```

`using` and `stopusing` are block scoped, `stopusing` is only necessary if you want the namespace to be stopped before the block scope ends.

# TODO implement

### Property Accessor Operator

The property accessor, also known as the dot operator is a period between an object and its properties or methods.  The dot operator can be changed for so long as the property belongs to the type returned by the property before it.

Whatever a function returns is that function's type, and therefore another function may be called immediately by the next.

`'123'.split().map(digit|>parseInt).reduce(digit|>sum)`

The above produces the value `6` by splitting the string `'123'` into an array of characters, then turning each character into an integer via map, and then adding them with reduce.

Map and reduce (reduce could handle both parseInt and summing at the same time) are redundant in this case, but serve to provide an example of chaining and piping.

All of the above methods would be native to the language (maybe rename `parseInt` to just `int`)

Alternative to the dot operator is bracket notation, which allows for dynamic access to any object's property.

```
final Person = {
    name: "Tom",
    age: 28,
    address: {
        street: "123 Any Street",
        city: "Any City",
        state: "Wisconsin",
        zip: 38291
    },
    saySomething(say): void {
        stdout.println(say);
    },
    favoriteFoods: [
        "chocolate",
        "cookies",
        "apples"
    ]
}
```

- `Person["name"]`: "Tom"

- `Person["address"]["street"]`: "123 Any Street"

- `const key := "name";`

- `Person[key]`: "Tom"

If the value is a function or a list, it can be called or sliced as if it is a function or list.

- `Person["saySomething"]("Hello!");`: prints `hello` to the console

- `Person["favoriteFoods"][1:]`: `["cookies", "applies"]`

- `Person["name"] = "Tim"`: Absolutely not, because `Person` is marked `final`.  If person was `const` or `let` then this would be acceptable.  `const` is acceptable, because you are not reassigning the value of `Person`, but of one of `Person`'s properties.

Objects are essentially dictionaries or associative arrays with inheritance.

### Pipe operator

The pipe operator `|>` is shorthand, used in any method that takes a callback.  So long as that callback has a single line, it _pipes_ the parameter passed to the call back into the function it is given.

Basically this:

`digit|>parseInt`   (or whatever I end up naming a string-to-integer function)

Is equivalent to

`(digit)=> parseInt(digit)`

It does not have to be used as a call back, it is a valid expression on its own.

`'5'|>parseInt` is the same as `parseInt.bind(null, 5)`.  However it does not _call_ `parseInt`, it binds it with a value to be called later.

Typing works as such:

`digit:char |> parseInt:int`

Because parseInt is already defined to return an int, no need to define a return type.  However, I'm providing it here as an example

It also takes multiple parameters, if the function it feeds takes multiple parameters

`(a: int, b: int)|> add`

Like `bind`, omitting a parameter makes it a partial, but may throw an error if its being used as a call back for a method like `map` or `reduce` that needs that parameter filled.

Neither the pipe operator nor dot operator can be or need to be overloaded.

If a method cannot be reduced to a single line expression, it cannot be represented with the pipe operator

> Compiler must look ahead when it encounters `|` to ensure that it doesn't try to interpet it as an `or` if there's a `>` immediately following it

## Types

A `type` declares the shape of an object and may be used to ensure an object fits a specific shape.  Functions are denoted with `()` and their specific parameters and return type

For example:

    type Person {
        private name: string;
        private age: int
        public greeting(statement: string): void;
        default getName() { return name; }
    }

Types do not have any values, they simply present a schema an object must conform to.

This can be used on an object to ensure type safety:

    final Tom: Person = {
        name: 'Tom'
        private age: 27
        public greeting(statemet) { print(f'{this.name} says {statement}'); }
    }

This fails to conform because name isn't private.  However, Tom does have access to default method getName(), without redeclaring it implicitly.  Functions in a type may only have a body if they're marked `default` or `static`.  These functions may be overridden unless marked `final`.

Types may have static functions that are called off the type.

Types are often used for typechecking in functions:

    type Point {
        x: int;
        y: int;
    }

    func slope(a: Point, b: Point): float {
        return (a.y - b.y) / (a.x - b.x);
    }

    // ensures that a and b have x and y properties

Type will throw `TypeError` if types don't match

I was going to also have interfaces, but come to think of it, types are interfaces, so never mind that

### Namespacing

The `namespace` keyword takes a list of types.  All types are defined by the types listed in the name space for all lines following the `namespace` keyword until either the end of the file, or the next `namespace` keyword, if the type is overridden there

`namespace myType, myOtherType;`

Setting a value to something else can be done like so, with the assignment operator.  Note that this creates `myType` as an alias but only for any lines following that namespace declaration

`namespace myType = SomeOtherType;`

Namespacing is primarily for name collision avoidance and is unnecessary if a project doesn't need to worry about such things.

## Enums

Enums are a special type of class, marked with enum.  Enums do not need a constructor and they're relatively simple

    Syntax:

    enum Identifier {
        enumeratedValue(associatedValue)

        method()
    }

    enum Days {
        Sunday(0),
        Monday(1),
        Tuesday(2),
        Wednesday(3),
        Thursday(4),
        Friday(5),
        Saturday(6)
    }

Enums are a list of values.  In parenthesis (optional) is an associated value.  You can access an enum through both the value and the associated value:

`Days.Sunday` - will give you 0.  Because the number literal is an int, type is inferred to be int.  If there was no associated value, the string `Sunday` would be the associated value.  By default, enum associated values are just the string representation of the value

`Days(0)` - will give you the enumeration that has the associated value of `0`.  If it doesn't exist, you'll get `null`.  Again, if there's no associated value, then you'd use the string representation: `Days('Sunday')`

Enums can also be accessed like arrays and objects with bracket notation

`Days[0]` gives you the first enumerated value.
`Days['Sunday']` gives the enumerated value that matches the string `'Sunday'`

Enumerations may have their own methods but cannot have their own constructors, nor can they be instantiated or use inheritence.

all enums inherits Enum object, which has a forEach() and a map() method.  forEach iterates through all the enumerated values without returning anything.  map() will produce a _new_ enum from the old one (allowing you to, for example, change the associated values)

Enum also has `add()` and `remove()` to add/remove enumerated values.  Note if the enum is marked final, these methods will throw an error.   The `toArray()` method creates an array with all the enumerated values (array of arrays if associated values exist, where inner array has two elements in a key/value pair matching)

Unlike Java, there's no need for getters and setters with enums.

Setting an associated value is done with the `set()` method inherited from 'Enum' class.

`Day.Sunday.set(7)` sets the associated value of Sunday to 7.

Enum associated values don't have to be declared with literals, you can declare them abstractly to be set later, using types.  `set()` will always expect the type passed as the associated value.

    enum Day {
        Sunday(name: string)
    }

then later you can say

    func setDay(day: Day, string: string): void {
        day.set(string)
    }

    setDay(Day.Sunday, 'Sunday');

    // associated value of Day.Sunday is now the string 'Sunday'
    // it's the same as saying Day.Sunday.set('Sunday')

Trying to get a value that is set as a type rather than a concrete literal value, will return `undefined`

Day.Sunday(); // undefined

Also, any enum marked final, must have associated values of either a literal value or no associated value.  `remove()`, `add()`, `set()` are not available for final enums.

All classes in `Enum` are final and cannot be overridden in an enum.

### Object

All classes and enums inherit from the Object class.  The Object class has a number of methods defined as follows, static only if explicitly mentioned (if not marked static, must be called on instantiated object):

* `static getName()` - returns the object or class's name.
* `static getConstructor()` - returns the constructor function.  If object is literal or object's constructor is private, returns null.  If object has multiple constructor, returns array.
* `toString()` - returns a string representation of the object (name + hex code for memory address).  Can be overridden
* `static getShape()` - returns an abstract representation of the object: see below

    class Wizard {
        name: string;
        type: string;
        
        public constructor(name: string, type: string) {
            this.name = name;
            this.type = type;
        }

        castSpell(spell: function) {
            spell();
        }
    }

    const Gandalf = Wizard('Gandalf', 'White');
    Gandalf.getShape(); 

    /* returns the following string: 
    'Wizard: {
        name: string,
        type: string,
        castSpell(spell: function): function
    }' 
    */
* `toArray()` - returns an array of key/value pairs for all properties (but not methods) of an object.
* `freeze()` - makes an object immutable (useful if you don't want the object marked as final to start out with)
* `gc(force=false)` - suggests Jellyfish collects the garbage.  Does not guarantee garbage collection, unless `force` is true

### This

`this` will always refer to the calling object.

```
array.map(()=> {
    if(this[0]) == 2 {
        // do something?
        // anyways, point is, the array called map, and therefore `this` points to the array
    }
});
```

## Modules

Exports are declared implicitly or explicity with the `export` keyword.  If `export` is assigned a value, only the value assigned to `export` is exposed to imports otherwise every object in the file is exposed to imports

> It is suggested to use `export`, the default implicit export is only meant for small modules that export one or two things.  After all, if your module only has `class Foo`, then why should we bother with an extra line and an `export` statement?  But if your module has fifty machine learning functions and you only need 3 of them to be exported to the user, there's no reason to expose the entire module.

```
const PI: float = 3.1415;
export = PI;
```

Or exporting multiple values and objects, use an object literal:

```
const PI: float = 3.1415;
const pythagorian = (a: double, b: double)=> Math.sqrt((a ** 2) + (b ** 2));
export = {
    PI,
    pythagorian
}
```

Anything can be exported, any declared variable, provided the appropriate accessibility modifiers.

`import` can bring in objects from other file

Format is as follows (and only accepts .jf files):

- `import * from './myClass.jf'`
- `import myClass as myAlias from './myClass.jf'`
- `import myClass, myOtherClass as myAlias from './myClass.jf'`
- `import myClass as myAlias from './myClass.jf'`
- `import myClass, myOtherClass as myAlias, myOtherAlias from './myClass.jf'`

> In the final example, aliases are applied in the same order that objects are imported

> Note objects don't have to be classes, they can be any object from the file provided they have the appropriate accessibility (`private` or `file` objects, for example, cannot be exported or imported)

Some standard libraries must be imported, like Collections:

- `import LinkedList from Collections`

stdin, system are by default always included.

## Generators

A generator is a function that produces a series of values iteratively, successively working through one value at a time per function call.  Generators are iterables, and the built in generator object is `range()`

Declaring a generator, replace `func` with `gen`

```
gen myGenerator() {
    // do something
    yeild someValue
}
```

`yeild` is the value that is returned each time the function is called.  Yield is only available in functions marked `gen`.  Generators can have all other `func` modifiers and can be methods in a class like any other function.  Generators may not use `return`.  Generators may be async.

Methods:
`myGenerator.next()` calls and returns the next value to be yielded, or `undefined` if finished
`myGenerator.exhausted()` true if the generator is exhausted its values, false if `next()` is still valid.
`myGenerator.peek()` peeks the current value without triggering next

## Async/Await

Async/Await is not multithreading, it is a way to halt the execution of one function while continuing the exection of other functions.

A function is marked as follows:

```
async func getHttp(url) {
    let data: JSON = await getSomeJSON(url);
    // where getSomeJSON, not a real function, is some API call to a server somewhere.
}
```

(too JavaScripty?)

## Generics

```
class A<T inherits P, implements R + S> inherits B implements G<F> + Q<U> {
    private T t;

    constructor(T t) {}
}

final a := A<int>(3);
final a := A<string>("hello");

// or

final a: A<string> = A<string>("hello");

// but because we're passing a string as T, we know what T is, therefore the type can be inferred
final a := A<>("hello")
final a: A<string> = A<>("hello");  // string in <> must be explicit
final a: A<> := A<>("hello");       // or use inference

// however, if the constructor took some other type, then of course we'd have to explicitly tell the compiler that T is a string.
```

## Input/Output

Input is from stdin module and output is from stdout module.

stdin has file io and keyboard input

`stdin.read(prompt)` Takes user keyboard input.  Prompt may be omitted, but provides the user a message
`const file = stdin.open(filepath, mode)` opens a file and returns a `File` object
`file.read()` reads the contents of the file, returning a string (which can be further broken by `split` and other methods)
`file.close()` closes the file

`stdout` has print methods.  There are two:

`stdout.print(string)` prints to the console with no new line
`stdout.println(string)` prints to the console with a new line

> C's printf sounds overly complex and redundant with string interpolation.  Many of printf's formatting methods could be shuffled off to the string class instead.

## System

Includes `system.env` which has access to environment variables.  Command line arguments are passed immediately to `main()` and therefore don't need to be provided by `system`

## Dates

Uses common UNIX system.

Don't be a dolt and fuck things up like PHP did and don't put an object called `datetime` in a module called `datetime` like Python did.

## Metaprogramming

The ability to define methods on the fly

```
define["identifier"](...params) for String {
    // some programmatically designed implementation?
}
```

`identifier` is a string and can be stored in a variable and programmatically designed.  `String` would be the name of the class or object it is being added to (cannot be added to `final` classes).  Class/Object owner can be omitted if you just want a new function.  Just called whatever the string identifier is (without quotes of course, and with ())  You could also set it as a variable:

```
const object = {
    newMethod = define["myNewMethod"]() {
        // . . .
    }
}
```

Not sure how to design implementation code though.
