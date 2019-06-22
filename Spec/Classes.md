# Classes

## Constructors

The constructor of a given class is the name of the class with no return type.  If no constructor is defined, one is created, called the default, no args constructor.  If there is a constructor defined, then the default, no args constructor must be explicitly designed as well.

A class may have as many constructors as it wants through polymorphism.  Constructors may have any desired accessibility, as any other method.

```
class A {
    public A() {
        stdio.println("A is constructed!");
    }
}
```

An instance of the class is created by simply calling the constructor.  The `new` keyword is unnecessary as the compiler knows the function is a constructor because it shares a name with the class.

```
final a:A = A();
// or with inference
final a := A();
```

Constructors are often used to set fields with their initial values.  Fields are listed in the parameter list of the constructor, and must match spelling, case sensitivity and type of the fields.  Provided the parameters match the fields exactly, there's no reason to assign them in the body of the constructor.

```
class Point {
    private x:int;
    private y:int;

    public Point(x:int, y:int) {}
}
```

Of course, if desired, the non-shorthand way can be done as well:

```
class Point {
    private x:int;
    private y:int;

    public Point(x:int, y:int) {
        this.x := x;
        this.y := y;
    }
}
```

The above example is virtually the same as the first one.


# this

`this` only exists in a class or object context and always refers to the class in which it exists in.

```
class Point {
    private x:int;
    private y:int;

    public Point(x:int, y:int) {}

    public override toString():string {
        return f"({this.x},{this.y})";
    }
}
```

If an object is defined within another object, `this` refers to the most local object that it is within.

```
const person = {
    private name:String = "Tom",
    private age:int = 28,
    private address:{} = {
        private street:String = "123 Anywhere"
        private zip:String = "12345"
        getString(): String {
            return this.street;
        }
        getZip(): String {
            return this.zip;
        }
    }
    getName():String {
        return this.name;
    },
    getAge():int {
        return this.age;
    },
    getAddress(): {} {
        return this.address;
    }
}
```

In the above example, `this` while within the `address` object refers to the `address` object, while `this` in the `person` object refers to the `person` object.  It doesn't matter that `address` is within `person`, once inside the `address` object, context switches to `address`.

It works the same in classes.

```
class LinkedList<T> {
    private class Node<T> {
        data:T;
        next:Node;
        prev:Node;

        getData():T {
            return this.data;
        }
    }

    private head:Node;

    getHead():Node {
        return this.head;
    }
}
```
`this` while within the `Node` class definition refers to the `Node`, while within in the `LinkedList` definition refers to the `LinkedList`.