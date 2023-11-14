### Purpose
Ownership in rust is why we can have cpp style efficiency without the need for garbage collection.

### Ownership and the stack/heap
[The stack and the heap](stack_and_heap.md) are ways of managing memory, the main purpose of ownership is to manage that memory management process for you so you don't need to consciously think about what is where too often.

### Ownership rules
- Each value in rust has an *owner*.
- There can only be one owner at a time.
- When the owner exits scope the value is dropped.

### Scopes
It seems that the easiest way to think of scopes as being measured by curly braces. 
```rust
fn main() {
	{ // Scope starts here.
		let x = 5;
	} // Scope ends here, so x dies.
}
```

### Memory and allocation
Using the [String type](string.md) as an example, we can explain how Rust's ownership works.
The reason why Strings can be mutable while String literals can't be mutable is because of how they deal with memory and allocation. 

A string literal is known at compile time, which is why they are so fast and efficient. If they were mutable they would lose this efficiency. 

If we want to be able to maintain mutability so have a string of text that can be expanded and altered, we would need to keep this on the heap in order to facilitate this behaviour. 

Allocation is achieved with things like String::from(), de-allocation instead of being handled by a garbage collector it is handled by being popped when the variable with ownership of that memory goes out of scope. 

```rust
fn main() {
	{ // scope starts
		let s = String::from("Hello"); // allocated on the heap
	} // scope ends, s goes out of scope. Data is de-allocated. 
}
```

### Moving data
Stack copying
```rust
fn main() {
	// This creates two variables x and y both with the value of 5
	// because these are simple known size types they are both instantiated on 
	// the stack.
	let x = 5;
	let y = x;
}
```

Heap copying is actually moving.
```rust
fn main() {
	// Instead of createing 2 separate instances of a value on the stack
	// because String is "what I will call a complex type". 
	// We end up with a dead s and a living s1. If I try to manipulate or use s
	// I will get an error as s is no longer valid. 
	// This makes a shallow copy of a heap object a movement as the original
	// value is made invalid.
	let s = String::from("Hello, ");
	let s1 = s;
}
```
This has an implication that rust will never make expensive copy operations when using automatic operations. 

Heap deep copy.
```rust
fn main() {
	// With this method I can still use both of them. This does mean I have 2
	// identical instances of the same data on the heap.
	let s = String::from("Hello, ");
	let s1 = s.clone();
}
```

Non-returning functions and ownership
```rust
fn main() {
    let s = String::from("hello");  // s comes into scope

    takes_ownership(s);             // s's value moves into the function...
                                    // ... and so is no longer valid here

    let x = 5;                      // x comes into scope

    makes_copy(x);                  // x would move into the function,
                                    // but i32 is Copy, so it's okay to still
                                    // use x afterward

} // Here, x goes out of scope, then s. But because s's value was moved, nothing
  // special happens.

fn takes_ownership(some_string: String) { // some_string comes into scope
    println!("{}", some_string);
} // Here, some_string goes out of scope and `drop` is called. The backing
  // memory is freed.

fn makes_copy(some_integer: i32) { // some_integer comes into scope
    println!("{}", some_integer);
} // Here, some_integer goes out of scope. Nothing special happens.
```

Returning functions and ownership
```rust
fn main() {
    let s1 = gives_ownership();         // gives_ownership moves its return
                                        // value into s1

    let s2 = String::from("hello");     // s2 comes into scope

    let s3 = takes_and_gives_back(s2);  // s2 is moved into
                                        // takes_and_gives_back, which also
                                        // moves its return value into s3
} // Here, s3 goes out of scope and is dropped. s2 was moved, so nothing
  // happens. s1 goes out of scope and is dropped.

fn gives_ownership() -> String {             // gives_ownership will move its
                                             // return value into the function
                                             // that calls it

    let some_string = String::from("yours"); // some_string comes into scope

    some_string                              // some_string is returned and
                                             // moves out to the calling
                                             // function
}

// This function takes a String and returns one
fn takes_and_gives_back(a_string: String) -> String { // a_string comes into
                                                      // scope

    a_string  // a_string is returned and moves out to the calling function
}
```

It would appear that the general rule of thumb is that unless a value is explicitly pushed out of a scope (returned) it exits scope that scope and is cleaned up. If it is returned the data moves somewhere else and exists in that scope now. 
This logic all only applies to heap objects. Stack objects get copied around and are therefore perfectly usable even after they leave a function that uses them's scope.

We can avoid exiting scope using the following:
```rust
fn main() {
    let s1 = String::from("hello");

    let (s2, len) = calculate_length(s1);

    println!("The length of '{}' is {}.", s2, len);
}

fn calculate_length(s: String) -> (String, usize) {
    let length = s.len(); // len() returns the length of a String

    (s, length)
}
```

However the above is somewhat tedious, returning both values over and over would be a lot of work. Instead we can use [references](references.md).