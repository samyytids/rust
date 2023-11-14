By default all variables are instantiated are immutable.
```rust
fn main() {
	// This cannot be changed
	let x = 5;
	// This will cause an error since I am trying to mut and immutable variable.
	x = 6;
}
```

We can change this to being mutable by using the keyword mut.
```rust
fn main() {
	// This can be changed
	let mut x = 5;
	// This won't cause an error.
	x = 6;
}
```

Constants cannot be changed and cannot be made mutable. Think of this as being similar to constexpr in cpp. Const values MUST be provided with type annotations.
```rust
const MY_WIFES_NAME: String = "Honor";
```

Shadowing is allowed!
```rust
fn main() {
	let x = 5;
	let x = x + 1;
	{
		let x = x + 2;
		println!("the value of x in the inner scope is: {x}");
	}
	println!("The value of x in the outer scope is {x}");
}
```
Shadowed values exist within their scope. So, although I have incremented x to 8 in the inner scope. When we print x in the outer scope the final print only reflects the first increment and as such we end up with x = 6.

Note that shadowing is different to mutability. If I tried to do this without shadowing I would get compile time errors. Shadowing allows me to perform adjustments to variables without losing the immutability of the variable. 

Shadowing also allows type transformation. EG a shadow can be an int while the original was a String. 

Doing the same with a mutable variable will cause errors. 

```rust
fn main() {
	// This is all good
	let x = 4;
	let x = x + 1;
	let x = "I am a big boy";

	// This not good
	let mut x = 5;
	x = "Test";
	x = 5;
}
```