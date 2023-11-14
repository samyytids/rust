
- [ ] Create project using cargo

```bash
cargo new guessing_game
cd guessing_game
```

This step will create our basic file structure as well as our cargo.toml file and the main.rs file in the source folder

#### cargo.toml

```toml
[package]
name = "guessing_game"
version = "0.1.0"
edition = "2021"

[dependencies]
```

### main.rs
```rust
fn main() {
	println!("Hello, world!");
}
```

- [ ] You can then compile the code using the following:

```bash
cargo run
```


Here is a basic function that can be used to take user input

```rust
use std::io;

fn main() {
	// These two lines simple print to the console 
	// ! indicates that this is a macro rather than a function
	println!("Guess the number!");
	println!("Please input your guess.");

	// This create a mutable variable called guess.
	// This variable is instantiated with a blank string.
	// ::new(); is a common functionality used by most types to create
	// an empty value.
	let mut guess = String::new();

	// This block is using IO's stdin (think cpp), it is reading in the
	// line provided to the console and providing a mutable reference
	// of the value from the console to our guess variable.
	// .expect is how we provide error handling to rust. 
	// Unlike python rust has no inbuilt error checking system and 
	// instead you provide your own errors for your functions.
	// Most (I assume most and not all) functions return a result
	// results come in the shape of Ok(()) and Err(()).
	// Ok(()) contains the result given no error
	// Err(()) contains the error information if a result cannot
	// be produced.
	io::stdin()
		.read_line(&mut guess)
		.expect("Failed to read line");

	println!("You guessed: {guess}");
}
```

printing using placeholders

```rust
// This line works much like how f strings work in python.
println!("You guessed: {guess}");

let x = 5;
let y = 10;

// You can also use placeholders in a similar fashion the .format in python.
println!("x = {x} and y + 2 = {}", y + 2);

```

This code can then be compiled and ran using 
```bash
cargo run
```

We can add further functionality to our project by adding dependencies to our cargo.toml file.
```toml
[package]
name = "guessing_game"
version = "0.1.0"
edition = "2021"

[dependencies]
rand = "0.8.5"
```
The way that dependencies are handled in rust is that they use the most modern version that will still be compatible with your build. 
EG. 0.8.9 but not 0.9.0

You can manually update your dependencies using 
```bash
cargo update
```

Adding the random number generation to our previous code
```rust
use std::io;
use rand::Rng;

fn main() {
    println!("Guess the number!");

	// thread_rng() this uses a local thread specific to the current
	// thread of execution that is set by the OS. 
	// .gen_range(1..=100) think of this as random.randint(1,101) from 
	// python.
	// In rust the syntax 1..=100 is akin to range in python but it is
	// inclusive on both ends of the range rather than exclusive on the
	// upper bound.
    let secret_number = rand::thread_rng().gen_range(1..=100);

    println!("The secret number is: {secret_number}");

    println!("Please input your guess.");

    let mut guess = String::new();

    io::stdin()
        .read_line(&mut guess)
        .expect("Failed to read line");

    println!("You guessed: {guess}");
}
```

Comparing the user input to the random number.
```rust
use rand::RNG;
// This functionality is basically bringing in the ability to make < > and == 
// comparisons using a one of the std enums Ordering.
use std::cmp::Ordering;
use std::io;

fn main() {
	// see above

	println!("You guessed: {guess}");
	
	// This code effecitvely runs a switch/if else block and returns whichever
	// value of the Ordering enum is returned. 
	// If our number is smaller than the secret number then we end up with a
	// Ordering value of ::Less and as such the first print macro is called.
	match guess.cmp(&secret_number) {
		Ordering::Less => println!("Too small!");
		Ordering::Greater => println!("Too big!");
		Ordering::Equal => println!("You win!");
	}
}
```

But, the issue here is that rust is a typed language and because our guess value is a string while the secret number is a number. Rust does some implicit work on the backend depending on what value is provided to a let x = instantiation. 
You can be more specific about what kind of data you want by providing a specific type. 
```rust
    // See above
    let mut guess = String::new();

    io::stdin()
        .read_line(&mut guess)
        .expect("Failed to read line");

	// Rust allows for shadowing values by providing a new version of it with
	// of a different type.
	// This saves us from having to create 2 variables guess_str and guess.
	// .trim() is the rust equivalent of .strip(), it also removes escape chars
	// eg \n \r.
	// .parse() this is the equivalent of int(string_var)
	// .expect("Please type a number"); handles what to print if parsing hits an 
	// error.
    let guess: u32 = guess.trim().parse().expect("Please type a number!");

    println!("You guessed: {guess}");

    match guess.cmp(&secret_number) {
        Ordering::Less => println!("Too small!"),
        Ordering::Greater => println!("Too big!"),
        Ordering::Equal => println!("You win!"),
    }
```

Now we are going to add in looping in order to allow for multiple guesses.

```rust
    // --snip--

    println!("The secret number is: {secret_number}");

    loop {
        println!("Please input your guess.");

        // --snip--

        match guess.cmp(&secret_number) {
            Ordering::Less => println!("Too small!"),
            Ordering::Greater => println!("Too big!"),
            Ordering::Equal => println!("You win!"),
        }
    }
}
```
At the moment the only way out of this loop is to either crash the game (you could type quit). Or to hard exit using ctrl+c.

You can exit because of a correct guess using the following.
```rust
loop {
	println!("Please input your answer: ");
	match guess.cmp(&secret_number) {
		Ordering::Less => println!("Too small!");
		Ordering::Greater => println!("Too big!");
		// Instead of just printing things to the console
		// This allows for us to be able to execute multiple lines
		// of code for a specific Ordering value.
		Ordering::Equal => {
			println!("You win!");
			break;
		}
	}
}
```

What if I don't want the player to crash the game if they make a typo, eg 8u.
```rust
	// --snip--

	io::stdin()
		.read_line(&mut guess)
		.expect("Failed to read line");

	// So, instead of .expect(msg); This executes code dependent on the state of
	// our result variable.
	// __ in this context is a catch all. Think blank excep vs except
	// IntegrityError. As such no matter the error continue will be called.
	// Continue functions in the same way as it does in python. It doesn't break
	// the loop it merely moves onto the next iteration of the loop.
	let guess: u32 = match guess.trim().parse() {
		Ok(num) => num,
		Err(_) => continue,
	};

	println!("You guessed: {guess}");

	// --snip--

```

# Complete code snippet
```rust
use rand::Rng;
use std::cmp::Ordering;
use std::io;

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1..=100);

    loop {
        println!("Please input your guess.");

        let mut guess = String::new();

        io::stdin()
            .read_line(&mut guess)
            .expect("Failed to read line");

        let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };

        println!("You guessed: {guess}");

        match guess.cmp(&secret_number) {
            Ordering::Less => println!("Too small!"),
            Ordering::Greater => println!("Too big!"),
            Ordering::Equal => {
                println!("You win!");
                break;
            }
        }
    }
}
```
