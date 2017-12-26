# Polymorphismtion

## C++

C++ has 4 types of polymorphism:

1. Function name overloading - multiple definitions of the same function taking different arguments. 
2. Coercion - implicit type conversion, e.g. assigning a double to an int or a bool.
3. Parametric - compile type substitution of parameters in templates
4. Inclusion - subtyping a class with virtual methods overloads their functionality. Your code can use the pointer to a base class, yet when you call the method you are calling the function implemented by the subtype.

That is to say, the same named function can be overloaded with different parameters. 

### Function name overloading


```c++
class Variant {
public:
  void set(); // Null variant
  void set(bool value);
  void set(int value);
  void set(float value);
  void set(Array *value);
};
```

One of the biggest issues that you might begin to see from the above example is that is too easy to inadvertantly call the wrong function because C++ will also implicitly convert types. 

```c++

// Sample code
Variant v;
//...
v.set(NULL);
```

This example will call the integer overload because `NULL` evaluates to 0. One of the changes to `C++11` was to introduce an explicit `nullptr` value and type to avoid this issues.


## Rust

Rust has very limited support for polymorphism. 

1. Function name overloading - there is none. In so doing it avoids a lot of complexity and mess around overloading, especially in constructors. On the flip side it can make code look elegant. See section below for alternatives.
2. Coercion. Rust allows limited, explict coercion between numeric types using the `as` keyword.
3. Parameteric - similar to C++ via generics
4. Inclusion - there is no inheritance in Rust. The nearest thing to a virtual method in rust is a trait with an implemented function that an implementation overrides with its own. However this override is at compile time.

### Alternatives to function name overloading

If you have a trivial overloaded function you can just disambiguate your functions, e.g.

```rust
fn new(name: &str) -> Foo;
fn new_age(name: &str, age: u16) -> Foo;
```

Another way you can do this is with _traits_. A standard trait is called `Into<T>` where `T` is the type you wish to convert from and you implement this trait on the thing you wish to be able to convert into. 

So let's say we want to write a `new` constructor function for struct `Foo`.

We'll first implement the `Into` trait multiple times for different values that our struct can be made from

```rust
// This Into works on a string slice
impl Into<Foo> for String
    fn into(v: String) -> Foo {    
        //... constructor
    }    
}

// This Into works on a tuple consisting of a string slice and a u16
impl Into<Foo> for (String, u16)> {    
    fn into(v: (String, u16)) -> Foo {    
        //... constructor
    }    
}
```

Now we will produce a `new` function that takes anything that implements `Into<Foo>`. Calling it is So you can produce constructors such as this:

```rust
impl Foo {
  pub fn Foo<T>(v: T) -> Foo where T: Into<Foo> {
    v.into()
  }
}
//...
let f = Foo::new(String::from("Bob"));
let f = Foo::new((String::from("Mary"), 16));
```

So Rust allows polymorphic-like code but it makes you think about it a little differently.

### Coercion

Coercion also makes use of the `Into<T>` trait. If we want to coerce one type to another we just implement the `Into` trait and invoke `into()` on it.

```rust
impl Into<SquarePeg> for RoundHole> {
  fn into(v: RoundHole) -> SquarePeg {
    // Impl
  }
}
// Convert the type and use diamond pattern to indicate output type
let v = RoundHole { /*..*/ }.into::<SquarePeg>()
```

We might also use the `From` trait which is reciprocal with `Into` allowing you to convert _from_ something. 

```rust
use std::convert::From;

impl From<&RoundHole> for SquarePeg {
  fn from(v: &RoundHole) -> Self {
    // Impl
  }
}

// Make one thing from the other, but in this case use a reference
let rh = RoundHole { /*..*/};
let v = SquarePeg::from(&rh);
```