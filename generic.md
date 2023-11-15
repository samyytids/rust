Generics in rust are basically template argument from cpp. They allow you to supply any value to a function/datastructure/datatype. We've seen the use of generics in [[result]], [[vectors]] and [[option]].

### Functions
Generics are really useful for reducing code duplication.
```rust
fn largest_i32(list: &[i32]) -> &i32 {
	let mut largest = &list[0];
	for item in list {
		if item > largest {
			largest = item;
		}
	}
	largest
}

fn largest_char(list: &[char]) -> &char {
	let mut largest = &list[0];
	for item in list {
		if item > largest {
			largst = item;
		}
	}
	largest
} 
```
Both of these functions achieve the same thing using the same code, the only difference is the input type. We can reduce these down to 1 function that takes and returns a generic type. 

We can do this by doing the following:
```rust 
fn largest<T>(list: &[T]) -> T {
	let mut largest = &list[0];
	for item in list {
		if item > largest {
			largest = item;
		}
	}
	largest
}
```

But this won't actually work because we need to make use of [[traits]] the reason we need to use traits is that we have T which is completely generic but we are making  a > comparison. This requires that we only accept a orderable type. As such we need to specify that this generic can only accept types with the orderable trait.

### Structs
```rust
// Thing to note that since we only have 1 generic type T x and y must be of the
// same type. 
struct Point<T> {
	x: T,
	y: T,
}

// This struct can use 2 types
struct Point<T,S> {
	x: T, 
	y: S,
}

fn main() {
	let point1 = Point { x: 3, y: 3 };
	let point2 = Point { x: 3.5, y: 5.7 };
}
```

### Enums
```rust
enum Test<T> {
	Item(T),
}
```

### Methods
```rust
// --snip--
impl<T> Point<T> {
	fn x(&self) -> &T {
		&self.x
	}
}

// we can also imply restrictions on our generic struct
impl Point<i8> {
	fn function(&self) -> i8 {
		println!(self.x);
	}
}
```

### Performance implications
There is none. This is achieved by the compiler effectively doing the reverse of [[#Functions|what we did in the function section]] instead of taking multiple functions and converting them into a generic the compiler converts our generic function and produces all the personal versions of that function that will work with concrete types.