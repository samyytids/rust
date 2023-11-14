### Syntax
Identical to cpp 
```rust
fn main() {
	let x = String::from("Hello world");
	
}
fn print_string(s: &String) {
	println!(s);
}
```

By passing a reference you engage in what rust refers to as [borrowing](borrowing.md). This is basically the idea of allowing access to a value without transferring ownership, by not transferring ownership we avoid the issue of having the value leave scope at the end of the function it was passed to and being de-allocated.

### Mutable vs Immutable
You can have as many immutable references as you want in rust, but you can only have 1 mutable reference at a time. This means however, that you cannot have any immutable references at the same time as having any mutable references.

This is to avoid the issue of data races, where multiple pointers try to make changes to something or to read something at the same time as something was changed by another pointer. 
```rust
fn main() {
	// We are fine
	let s = String::from("Hello world");
	let ms = &mut s;
	// we've fucked it
	let is = &s;
}
```

### Dangling references
If you create a reference in scope and the original value leaves that scope while the reference is passed to another scope you end up with a dangling reference. 
Rust doesn't allow this, this situation will cause a compile error.

```rust
fn main() {
	let dead_reference = create_dangle();
}

fn create_dangle() {
	let s = String::from("Hello");
	// s itself will exit scope once this function returns, but you are then 
	// returning a reference to s at the end of this function. This would
	// create a dangling reference an as such this won't be allowed by the 
	// compiler.
	&s
}
```