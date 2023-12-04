### Function pointers
These can be used to pass functions as arguments to other functions. The main thing to pay attention to here is the detail needed to describe the type of the function being passed to the function. 

```rust
fn add_one(x: i32) -> i32 {
	 x + 1
}

// We need to specify both the arguement types as well as the return type in the 
// function signature. 
fn run_twice(function: fn(i32) -> i32, value: i32) {
	function(value) + function(value)
}

fn main() {
	let value = run_twice(add_one, 4);
}
```

The fn type is different to the Fn trait used when passing [[closures]] as an argument. The fn type implements all of the closure traits we have spoken about before Fn, FnMut and FnOnce. As such we can always pass a function in place of something that calls for a closure. 

However, there is a situation where passing a fn is objectively correct where passing a closure will cause an error. This is when interfacing with other languages like C which do not have closures. 

### Using enum initializers as arguments
[[enums|Enums]] options that contain data gain initializers and these initializers can be passed as functions to simplify the syntax needed to populate collections with enum values.
```rust
enum Status {
	Value(u32),
	Stop,
}

fn main() {
	// What this effectively does is iterate through the range from 0 to 19 and 
	// map thoes value sinto the Status enums Value initializer and once the 
	// mapping has been executed the collect() method populates the 
	// list_of_statuses vector with Status::Values. Remember when using collect
	// to populate a collection you need to specify the value that you are ending
	// up with in the collection via the type dectorator. 
	let list_of_statuses: Vec<Status> = (0u32..20).map(Status::Value).collect();
}
```

### Returning closures. 
Closures need to be returned as [[trait_objects]] this is because the size of a closure is not known at compile time. 
```rust
fn returns_a_closure() -> Box<dyn Fn(i32) -> i32> {
	Box::new(|x| x + 1)
}
```

#### Why would I want to return a closure?
Being able to return closures allows you to create customized functions at run time. Based on some criteria you could create a function that manipulates data in a custom way.