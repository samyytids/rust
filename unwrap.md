Unwrap is a method if the [[result]] type, I have put it here because there are many different derivatives of unwrap and it seems to be important.
Unwrap is basically used to circumvent the need for a match block. If we have a Result containing the Ok() then unrwap will return that value,  otherwise it will panic
```rust
use std::fs::File;

fn main() {
	let file = File::open("hello.txt").unwrap();
}
```