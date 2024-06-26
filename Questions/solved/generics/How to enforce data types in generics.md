
#answered Answer: Use single upper case letter and specify in parameters using parenthesis. e.g. 

```javascript
Node: {
	data T
	left Node(T)
	right Node(T)
}
// if need to specity the type, a single type can be used
Box: {
   T 
   data T
}
b Box(Int)
b.data = 1
```
TLDR: 
I wanted to use [qdbp](https://www.qdbplang.org/docs/examples#:~:text=as%20a%20function.-,Generics,-Methods%20are%20generic) generic style: 
But there's no way to enforce a child element to use the same generic: 

qdbp:
```js
Foo { 
	data 
}
f1 : Foo('hola')
f2 : Foo(2)
```

Problem:

```js
Foo { 
	data 
	bar: { 
		param 
		// cannot assert data and param have the same type
	}
}
f1: Foo('Hi')
f1.bar(1) // sure... 

```

That's why we need to have a way to enforce it, in Java this would be like

```java
class Foo<T> { 
	T data
	void bar(T param) {
	}
}
var f1 = new Foo<String>()
f1.data = "hi"
f1.bar(2) // compilatin errror 
```

Also is hard to add a second  param

```js

// qdbp style: doesn't work
Foo { 
	data
	moredata
	
}
Foo(1 'text')
// java aprox
class Foo<S,T> {
	S data
	T moredata
}
```

The goal is to add a syntax like this without using too much angle brackets (or not using them at all) as much of the Yz code is generic, e.g. 

```js
if : { 
	cond Bool
	action  { T } // action is a generic block 
	else    { T } // else is a generic block, should be the same type
}
r: if isTime() { 
	1 
  } {
	2
  }
```

### Options: 

### 1 Use `<>` as Java
```js
if<T>: {
	... 
}
r: if<Int> isTime() {
	1
  } {
	2
  }
```


We can define a data is generic by not specifying the data type and bind it on first usage, however we cannot enforce instances of the same type to be the same generic data

```javascript
Node {
	 data // generic
	 left Node // want to make left and right same type as `data`
	 right Node 
}
a = Node(1)
a.left = Node("B")
```

### Exploration

We can receive the generic type as argument to to block 

```js
b: {
	T // generic data type 
	data T // data is of tye T
}
// invocation
b(String 'hola')
// can be inferred? 
b('hola')

if(Int isTime() {
  1
}{
  2
})

```

Could have the reserved word "type" and use it in similar to [Zig](https://ziglang.org/documentation/master/#toc-comptime)

The problem is, it's hard to tell between a new type declaration and the declaration of a type 
```js
// New type Foo with a generic type T
Foo { 
	T type
	data T // data is of type T
}
//
foo Foo{T} // foo is of type Foo<T> 

if:  {
	T type
	condition Bool
	then { T }
	else { T }
}
if Int isTime() {
	1
} {
	2
}

```

```js
b: { 
	T type
	data T
}
b(String 'Hola')
b('hola')


HashMap { 
	K
	V
	put: { 
		key K
		value V
	}
}
map: HashMap{String Int}
map.put('64' 64)

list: { 
	T
	{ 
		items []T
		len Int
	}
}
list = List(String)

Node {
	T
	data T 
	left Node{T}
	right Node{T}
}
n: Node{String}
```


```js
none: std.option.none
tree: {
	Node {
	   T
	   data T
	   left Node = None(T) // Type Node { T; data T; left Node{T}; right Node}
	   right Node = None(T)
	}
	builder: {
		{
			T
			data T // generic data
			Node(T data)
		}
	}
}
node: tree.builder(Int)
root: node(1) // the builder `node` is now bound to `Int`
root.left = node(2)
root.right = node(3)
node_str: tree.builder(String)
a : node_str('hi')
a.left = node_str('Bye')
// this would be still valid however
a.right = node(1) // a.right of type Node{String} is not compatible with Node{Int}


```


```js
something: {
	T { len {Int} }
	data T
	data.len() // data is generic but needs a `len()` method
}
something(1) // comp err, Int doesn't have a `len()` method
something("Hi") // ok
something([1 2 3) // ok.. we wanted soemthing with `len()` after all.
```



```js
stack: {

	new: {
		Stack() 
	}
	Stack { 
		T 
		values []T // generic array
		push: {
			v T // generic value
			values << v
		}
		<< : { v T  push v }
		pop: {
			values.len() == 0 ? {
			   option.None(T)
			} { 
				option.Some(value.pop())
			}
		}
	}
}
main: {
	s: stack.new(Int)
	s.push 42
	s << 43 
	// This is an error, as the stack inner `values` is now bound to `Int` types
	stack.push 'Oh no!'

    // Trying to use a generic type before is bound would return an error
    t: stack.new(Int)
    x: t.pop() // not known yet, probably compile error
	10 + x.get() // yeah... probably not... or maybe, there's no way to know


    as_param: {
	    t stack.Stack{T}
		10 + t.pop().get() // would require t.value to be bound to Int // :? maybe
    }

}
```



```js
User {
    id Number
    name String
}
Post {
    id Number
    user_id Number
    title String
    body String
}
Repo {
	T
    db DB
    find_by_id: { id Number
        table_name: info(T).name
        db.query_one('select * from {table_name} where id = ?', id)
    }
}
new_repo: {
	T
    db DB
    Repo{T db}
}
db = some_library.new_db()
user_repo = new_repo(User db)
post_repo = new_repo(Post db)

user: user_repo.find_by_id(1)
post: post_repo.find_by_id(1)
```

```js
compare: {
	T
    a T
    b T
    if ({a < b }, { <- -1 })
    if ({a > b }, { <- 1 })
    0
}

compare: {
	T
	a T
	 b T
    a > b ? { return -1 }
    a > b ? { return 1 } 
    0            
}
// compare<Integer>
println(compare(Int 1 0))
print compare Int 1  0
// compare<String>
println(compare(String 'a' 'b'))
// compare<Decimal>
print compare Decimal 'a' 'b'
println()

```



```js
// Reduce reduces a []T1 to a single value using a reduction function.
reduce: {
	T1
	T2
    s []T1
    initializer T2 
    f {T2 T1 T2}
	
    r: initializer
    s.for_each {
        index Int
        v T1
        r = f r v 
    } 
    r
}

// Filter filters values from a slice using a filter function.
// It returns a new slice with only the elements of s
// for which f returned true.
filter: <T> { s []T; f {T Bool}
    r []T
    r.for_each {_ Int; v T1
        f v ? { r << v }
    } 
    r
}
```


```js
option { 
	T
	None{}
	Some{T}
}
Some[]

```

### Lisp Style

```js
Foo { 
	T
	data T
}
param: {
	foo Foo T
}
param(Foo(String))

```

```js
// Java:
// New type Declaration 
class Foo<T> { 
	T data
	Foo<T> child
	// Function Declaration
	void <T> foo (T param) {
	}
}
// Generic variabla declaration
Foo<String> a
// Usage
a = new Foo<String>()
a.data = "hi"
a.child = new Foo<String>()
a.<String>foo("hi"); // although it's inferred


// Yz
Foo { 
	T
	data T
	child Foo T 
	foo: { 
		T 
		parm T
	}
}
a Foo String
a.data = 'hi'
a.child = Foo(String)
a.foo(String 'hi')

// Yz
Foo { 
	<T>
	data T
	child Foo T 
	foo: { 
		T 
		parm T
	}
}
a Foo String
a.data = 'hi'
a.child = Foo(String)
a.foo(String 'hi')


```



```js
// Reduce reduces a []T1 to a single value using a reduction function.
reduce: {
	<T1>
	<T2>
    s []T1
    initializer T2 
    f {T2 T1 T2}
	
    r: initializer
    s.for_each {
        index Int
        v T1
        r = f r v 
    } 
    r
}

// Filter filters values from a slice using a filter function.
// It returns a new slice with only the elements of s
// for which f returned true.
filter: {<T> s []T; f {T Bool}
    r []T
    r.for_each {_ Int; v T1
        f v ? { r << v }
    } 
    r
}

```


## Type parameters

```js
min: { 
	x Decimal
	y Decimal 
	x < y ? { x } { y }
}
Ordered : x.exp.constraints.Ordered
gmin: {
    <T>
	x Ordered
	y Ordered
	x < y
}
gmin(Int 2 3)


Tree { 
	<T>
	value Tree{T}
	left Tree T
	right Tree T 
	lookup: {
		 x T
		 // returns Tree{T} 
	}
	
}
Ordered { 
	Int
	String
	Float
}
```



```js
List list = new LinkedList();
list.add(1);
i = lit.iterator().next();


list : LinkedList{Int}
list << 1 
i Int = list.get(0)


from_array_to_list: {
	<T>
	a []T
	arrays.stream(a).collect(collectors.to_list)
}
int_array = [1 2 3 3 4 5]
string_list List{String} = from_array_to_list(int_array)

```

```js
// differnet ways to type
// Map<Thing<String, Int>, Option<Box<Char>>>

map Map(Thing(String Int) Option(Box(Char)))

Option {
	<T> 
	None
	Some(T)
}
Some {
	data <T>
}
None {
	<T>
}
user? : find_user(123)
find_user: {
	...
	user User
	result Option {User}
	...
	if user.is_valid() { 
		result = Some(user)
	} {
		result = None(User)
	}
}
some Option{User} = Some(user)



```

#### Closer

```js
// different ways to type
// Map<Thing<String, Int>, Option<Box<Char>>>

map Map(Thing(String Int) Option(Box(Char)))

Box {
	<T>
}
Option {
	<T> 
}
Some {
	data <T>
}
None {}
user? : find_user(123)
find_user: {
	...
	user User
	result Option(User)
	...
	if user.is_valid() { 
		result = Some(user)
	} {
		result = None()
	}
}
some Option(User) = Some(user)

```


```js
Box {
 data <T>
}
```


## Proposal

Single letter, upper case, means generic. 

```js
Node : {
	data T
	left Node(T)
	right Node(T)
}
root : Node('text') // bound to `String`
root.left = Node('left text') // ok
// root.right = Node(1) // compiltion error
```

```js
// different ways to type
// Map<Thing<String, Int>, Option<Box<Char>>>

map Map(Thing(String Int) Option(Box(Char)))
Box : {
	data T
}
Option : {
	T
}
Some :  { 
	data T
}
None {}
user? : find_user(123)
find_user: {
	...
	user User
	result Option(User)
	...
	if user.is_valid() { 
		result = option.some(user)
	} {
		result = option.none()
	}
}
some Option(User) = Some(user)

```


## Resources
- [What are some syntax options for describing generic ("templated") types?](https://langdev.stackexchange.com/questions/122/what-are-some-syntax-options-for-describing-generic-templated-types)


