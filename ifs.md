#### Syntax
The syntax of if statements within rust are a halfway house between cpp and python.
```rust
fn main() {
	let number = 3;
	if number < 4 {
		// foo
	} else {
		// bar
	}
}
```

If statements don't try to automatically convert non-booleans into bools.
```rust
fn main(){
	let number = 3;
	// This will cause an error
	if number {
	
	}
}
```

Extra conditions uses the following syntax.
```rust
fn main() {
	if foo {
	
	} else if bar {
	
	} else {
	
	}
}
```

Using ifs within a let, the type in each condition need to be identical.
```rust
fn main() {
	let x = if foo {5} else {8};
	
	// This is not valid
	let y = if foo {6} else {'6'};
}
```

Using multiple conditions in one if.
This is identical to how it is done in cpp
```rust
fn main() {
	if foo && bar {
	
	} else if foo || bar {
	
	} else {
	
	}
}
```