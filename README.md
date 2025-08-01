This repository contains design notes for the Yz programming language, like examples, features and design questions. 


> [!WARNING]
> There's no [compiler for the Yz programming language yet](https://github.com/oscarryz/yzc), making all the content here vaporware. 

# Quick overview

```javascript
// Factorial in Yz
factorial: { n Int
  n > 0 ? { n * factorial(n - 1) }
          { 1 }
}
print("`factorial(5)`")  // prints 120
```
Yz is a programming language that explores the possibility to simplify concurrency, data, objects, methods, functions, closures, classes under a single artifact: a code of block.

# Blocks
In Yz, almost everything is a block of code. 

A code of block is a series of expressions between `{` `}`, their variables can be accessed from outside the block (behaving like objects) and they can be executed (behaving like methods/functions/closures) and they are concurrent (behaving like actors).


```javascript
language: {
  name: 'Yz' // `name` is a String with value 'Yz'
}
main: {
  print(language.name) // Access the `name` variable
}
```

To execute a block, use `()` as you would with a function, parameters will be assigned to the block variables starting from the top, return values are the last expressions evaluated starting from the bottom - n

```js
// A block with 3 Int expressions
sum: {
   a : 0
   b : 0
   a + b
}
// Assigns values and executes.
//  a is 1 and b is 2
// result == 3 (the last expression was `a + b`)
result: sum(1,2)

// "Multiple" return values
x, y, z : sum(1, 2)
// x, y, z are the  last n - 3 expressions thus:
// x is 1 (the variable a)
// y is 2 (the variable b)
// z is 3 (the expresion a + b) 

// Also, the `()` invocation can name the variables, behaving like named parameters
sum(b:10, a:20) // 20 + 10 
```

## Creating instances of a block

If the identifier starts with uppercase then it defines a new type. When executed, an instance is created. 

```javascript
// Defines a new type `Point` with integer variables `x` and `y` 
Point : {
    x Int
    y Int
}
p1 Point =  Point(x:10, y:20) // p1 variable declared and initialized an a new instace created
p2: Point(40, 40) // p2 type `Point` inferred.
```

Variables can be accessed from outside the block and blocks can be variables too, thus a nested block can be used as methods. 

The following defines the `to_string` block that access the variables `x`  and `y` from the outer scope behaving like a method.

```javascript
Point : {
    x Int
    y Int
    to_string: {
      "`x`,`y`" // `expr` for string interpolation
   }
}
p: Point(0, 0)
print(p.to_string()) //prints `0,0`
```

### Non-word variable names
If a "method" is a non-word name ( e.g. `+`, `>`, `<`, `>>=` etc.), it can be executed without the `.`_name_`()` notation. 
This makes the code look more like traditional operators.

```javascript
Point : {
    x Int
    y Int
   // `+` is a non-word name block/method
   + : { 
       other Point
       Point( x + other.x, 
              y + other.y)
    }
}
// p1 and p2 are Point's
result : p1 + p2 
// is the same as 
result : p1.+(p2)
```

## Expression blocks

It is also possible to have blocks with no variables and only expressions.

```javascript
one_two: { 1, 2 }
```
They don't take parameters when executed, and their values cannot be accessed by name, only by index.

```javascript
one_two: { 1, 2 }
a, b: one_two() // a:1 b:2
// or 
one_two()
a : one_two.0 // 1 is the first computed value
b : one_two.1 // 2 is the second computed value
```

## The block signature

The type of a block is defined by its signature. The syntax for a block signature is `#(` _optional list of types_ `)`.  
If they don't have variable names they are expected to be expressions of the given type.  

The following declares a variable `two_ints` of type block, that contains a named variable `a` of type `Int` and a unnamed expression of type `Int`.

```javascript
// block signature: "block with an `Int` variable names `a` and a unnamed `Int` expression"
two_ints #(a Int, Int ) = {
  a: 1
  2
}
```
If the short declaration form is used (`:`) the type is inferred.
```js
// Same as above.
two_ints: {
  a: 1
  2
}
```

## Generics 

If the type is a single upper case leter, then it's a generic.

```js
// Box is a new type with signature "block with variable `data` of generic type T"
Box #(data T) = {
   data T
}
int_box : Box(1) // data is 1
string_box : Box("Hi") // data is "Hi"
```

A generic without variable can be used to "instantiate" the generic type:

```js
Box: {
  T
  data T
}
// Declares a variable `int_box` of type Box of Int
// The variable `data` is still not initialized but wil be of type Int
int_box Box(Int) 
int_box = Box(data:"Hi") // Compilation error: Cannot assing Box(String) to Box(Int) 
``` 


## Structural typing

Yz uses structural typing to know if a block can be assigned to a variable.  
The match of the structure includes the variable names.  

```js
thing #(Int, Int) // `thing` is a variable of type block with two integers

// Matches because time has two inta
time: {
   hour: 12
   minute: 24
}
thing = time

// Can be assigned because `Point` has two Int
Point: {
  x Int
  y Int
}

thing = Point( 1 , 2 )

// Can be assigned because the expresion block has two Ints
thing = { 3, 4 }
```

## Concurrency (or in Yz the functions color are ... purple)<sup>1</sup>

### Purple functions + Structured concurrency

Every function call async, unless the return value is used right away.

Say for instance you have two functions:
```js
fetch_image(iid)
fetch_user(uid)
```
If you assign a return value, the code will wait until the function finishes:

```js
// Fetch the image 
image: fetch_image("129-083")

// The following line will run
//  only when image
// fetch completes because
/  we assigning the value
// to the variable `image`
user: fetch_user("mr_pink")
```

But if you don't assign the value, then they just run asynchronously:

```js
// These two run concurrently
fetch_image("129-083")
fetch_user("mr_pink")
```

Using a structural concurrency like approach, they would synchronize at the end of the enclosing block/function. 

```js
enclosing_block: {
  // These two run concurrently
  fetch_image("129-083")
  fetch_user("mr_pink")
  Profile(fetch_iamge.data, fetch_user.data)
}
```

It is usually better to create a new instance to keep the state separated.

For instance with a `Fetcher` type, with a `data` attribute would be like this: 
```js
enclosing_block : {

  // Create `Fetcher`s
  img : Fetcher()
  usr  : Fetcher()

  // fetch data async
  img.fetch("129-083")
  usr.fetch("mr_pink")

  // When they reach the
  //  "bottom" of 
  // `enclosing_block`, 
  // the data is ready 
  // to use.
  // Create a new object:
  profile : Profile(usr.data,
                    img.data)
}
```

This allow sync and async calls with a very simple syntax and rules; if you need the value right away then it is sync, if you need it later then is async.  

 

<sup>1: [What Color Is Your Function? - stuffwithstuff.com](https://journal.stuffwithstuff.com/2015/02/01/what-color-is-your-function/)</sup>

