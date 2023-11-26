std's mem::drop function effectively triggers the .drop method of an object manually. 

### Implementation
```rust
fn main() {
	let c = CustomSmartPointer {
		data: String::from("I will be dropped early");
	}
	println!("I have created a smart pointer");
	// Obviously I need to import this function from std::mem in order to use it.
	drop(c);
	println!("The smart pointer I created was dropped ealy");
}
```