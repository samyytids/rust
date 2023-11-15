Expect is somewhat similar to [[unwrap]] in that it is a less verbose way of handling Results. Expect differs in that it allows you to specify your error message.
```rust
use std::fs::File;

fn main() {
	// Generally expect is more commonly used in production code as you can 
	// provide specific context as to why one would expect the code to always
	// succeed. 
	let file = File::open("hello.txt")
		.expect("Hello.txt not found");
}
```