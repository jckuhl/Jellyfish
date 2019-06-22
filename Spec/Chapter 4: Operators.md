# Operators

We've handled the assignment operator so lets take a look at all the rest

# Mathematical Operators

A colon before the operator opts the programmer into type coercion.

- Addition: `+` and `:+`
- Subtraction: `-` and `:-`
- Multiplication: `*` and `:*`
- Division: `/` and `:/`
- Exponent: `**` and `:**`
- Modulo: `%` and `:%`

Examples:

```
var x := 3 + 4;  // 7
x := x + 5;      //12
x := x ** 2;     //144
x := x / 7;      //TypeError x of type integer, not double
```
In the final example with the division, we would have to _cast_ `x` to double.  This we'll cover in the section on Types.