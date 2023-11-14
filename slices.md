### Syntax
I have no real analogy for the syntax for slicers. 
Sort of close to slices in python I guess, but they use the .. range syntax from rust and are combined with referencing from cpp. Seems like a similar concept to spans from cpp. 
```rust
fn main() {
	let s = String::from("Hello world");
	let hello = &s[0..5];
	let world = &s[6..11];
}
```

### Purpose
Slices are used to reference to a contiguous series of elements from within a collection.

A slice consists of 2 parts, a pointer to the start of the slice and the length of the slice. 

Much like normal references when the actual value goes out of scope the slice is no longer valid.

### Tip
use &str instead of &[String](string.md) for inputs to functions, this is because &str allows usage with both str and String while &String will only allow you to use Strings. 