Functions are exactly what you think they are. A new function can be created using the fn keyword. Rust doesn't care where in a file that a function was defined, IE you can define a function "after" it is used.

```rust
fn main() {
	println!("Hello world");

	another_function();
}

fn another_function() {
	println!("Another function!");
}
```

Defining a function with arguments:
You must specify the type of arguments in the function definition as this allows for rust to provide more effective error messages.
```rust
fn main() {
	another_function(5);
}

fn another_function(x: i32) {
	println!("Input numer: {x}");
}
```

Defining a function with multiple arguments:
```rust
fn main() {
	another_function(6, 'h');
}

fn another_function(x: i32, c: char) {
	println!("Input values x: {x}, c: {c}");
}
```

## Statement vs Expression

Statements: a series of instructions with no return
Expressions: Evaluate to a result.

A very basic statement is the let statement which we are more than familiar with. 
```rust
fn main() {
	let x = 5;
}
```

Because statements don't return values you can't assign a value with a statement.
```rust
fn main() {
	let x = (let y = 5);
}
```

A very basic expression is assigning a variable a valuable using mathematics operators.
```rust
fn main() {
	let y = {
		let x = 4;
		// There is no semicolon here because this turns it into a return
		// expression, if I were to add a ; I would turn this into a statement
		// and as such I would experience an error.
		x + 5
	}
	println!(y);
	// 9
}
```
In this case the items within the curly braces used to set the value of y form an expression which returns a value of 9. 

Functions with return values:
```rust
use rand::RNG;

fn random_number() -> i32 {
	rand::thread_rng().gen_range(1..=100)
}
```
This function will return a value between 1 and 100.