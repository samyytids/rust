### Characteristics
This is an enum type that consists of 2 parts, the result value that you expected or some kind of error. Both of which are generic types. 
```rust
enum Result<T, E> {
	Ok(T),
	Err(E),
}
```

### Example
```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
	// We have read a file... But we don't know if we did so successfully.
	let file = File::open("hello.txt");

	// That's where this comes in. We check whether the file matches either
	// Ok or Err, if Ok we return the file and move on. 
	// If Err we panic.
	let file = match file_result {
		Ok(file) => file,
		Err(error) => panic!("File not loaded", error),
	};

	// Expanded example:
	// This is described as primitive, later we will talk about closures.
	// See an unexplained example of closures below.
	let file = match file_result {
		Ok(file) => file, 
		Err(error) => match error.kind() {
			ErrorKind::NotFound => match File::create("hello.txt") {
				Ok(fc) => fc,
				Err(e) => panic!(
					"File not found, create file also broke {:?}", e
				),
			},
			other_error => {
				panic!("Problem opening the file: {:?}", other_error);
			}
		},
	};

	let greeting_file = File::open("hello.txt").unwrap_or_else(|error| {
		if error.kind() == ErrorKind::NotFound {
			File::create("hello.txt").unwrap_or_else(|error| {
				panic!("Problem creating the file: {:?}", error);
			})
		} else {
			panic!("Problem opening the file: {:?}", error);
		}
    });
}
```

### Error propagation
As far as I can tell this is just a fancy way of creating functions that return a result rather than a value, thus pushing the error handling to the context using the function rather than having the function handle it itself. 
```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
	// Reads the file
    let username_file_result = File::open("hello.txt");

	// Checks if the file read without errors
    let mut username_file = match username_file_result {
        Ok(file) => file,
        Err(e) => return Err(e),
    };

	// Creates an empty string 
    let mut username = String::new();

	// We return the result value Ok(username) or the Err(e)
    match username_file.read_to_string(&mut username) {
        Ok(_) => Ok(username),
        Err(e) => Err(e),
    }
}
```

#### The ? operator
Error propagation is incredibly common in rust so there is a dedicated operator for it. 
The heavy tldr; of the ? operator is that it replaces the match case format of error handling detailed above. It either returns our result or returns the error.
```rust
use std::fs::File;
use std::io::{self, Read};

fn read_username_from_file() -> Result<String, io::Error> {
    let mut username_file = File::open("hello.txt")?;
    let mut username = String::new();
    // This is the function defined in the snippet above.
    // The ? returns either the value in the Ok() or it returns the Err
    // the purpose of this is that ? converts the specific error into a generic
    // error defined within the function. 
    username_file.read_to_string(&mut username)?;
    Ok(username)
}
```

A nuance to think about is that the ? operator will return a function early if an error occurs but will allow the function to continue otherwise.
```rust
fn last_char_of_first_line(text: &str) -> Option<char> {
    text.lines().next()?.chars().last()
}
```

There are more features for this operator that I will have to learn through experience or more reading.