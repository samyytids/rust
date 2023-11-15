[[cargo|Look here for instructions on how to create a project]]
### Accepting CLI agruments

The most important aspect of this is equivalent to pythons sys library. 
```rust
// import sys
use std::env;

fn main() {
	// args requires value unicode. 
	// If you expect non-valid unicode arguments use arg_os. This produces an 
	// iterator that returns OsStrings instead of Strings. 

	// .collect() converts an iterator into a collection (vector) of the items
	// in the iterator. What collection it parses the data into requires explicit
	// annotation since it is difficult to infer what kind of collection you want
	// the data to be stored in. 
	let args: Vec<String> = env::args().collect();

	// Print but pretty.
	// One of the things to take into account is that the first argument will be 
	// the location of the binary that was ran. 
	dbg!(args);
}
```

#### Turning the list into a series of arguments we can use more easily

Keeping this as a vector is *an* option, but it's probably a good idea to split them into separate variables or into a struct or some other data structure that allows you to access them in a more intuitive way.

```rust
fn main() {
	// --snip--
	let query = &arg[1];
	let filename = &arg[2];
	println!("Looking for {}", query);
	println!("Checking in {}", filename);
}
```

### Reading a file

```rust
use std::env;
use std::fs;

fn main() {
	// --snip--
	// This reads in the file and converts it to a string
	let contents = fs::read_to_string(file_path)
		// If we get a result continue as normal, if not produce the following
		// error code. 
		.expect("Should have been able to read the file");

	// print file contents with a line break between With text: and the content. 
	println!("With text:\n{contents}", contents);
}
```

### Improving modularity

As is our main.rs function performs 2 tasks, opening and parsing the code. We should really split this into two separate parts. Our main function is also handling errors, we could also separate this out too. This isn't too bad now because our programme is small, but as the file gets larger and larger and more and more complex continuing in this fashion will make it difficult to interpret what our code does and harder to test our code. 

Another issue is that our environment variables are stored in separate variables which once again, fine while our programme is simple. But, the more complex it becomes the more difficult it will be to keep track of the purpose of these variables and what variables they relate to. 

A further issue is that our expect is very generic, it doesn't specify how or why the file failed to load. 

Finally As alluded to in issue 1, our error handling should probably be abstracted out so it's easier to tweak our errors as well as easier simpler to find where our error logic is coming from. 

#### Extracting argument parsing
##### Option 1: basic
This option moves out the argument parsing making the code more modular.
It also more explicitly relates the query to the file_path, implying that the query is likely intended to be used on that file_path.
```rust
fn main() {
	let args: Vec<String> = env::args().collect();
	let (query, file_path) = parse_config(&args);

	// --snip--
}

fn parse_config(args: &[String]) -> (&str, &str) {
	let query = &args[1];
	let file_path = &args[2];

	(query, file_path)
}
```

##### Option 2: Simple struct
This option makes it even more explicit that the arguments are related and makes accessing them simpler via .notation rather than arbitrary indexing. 
```rust
fn main() {
	let args: Vec<String> = env::args().collect();
	let config = parse_config(&args)

	println!("Checking file: {}", config.file_path);
	println!("Looking for string: {}", config.query);

	let contents = fs::read_to_string(config.file_path)
		.expect("Should have been able to read file");
}

struct Config {
	// Could instead use String and call &args[1].clone() which creates a full
	// copy of the string in args[1];
	file_path: &str, 
	query: &str,
}

// since I am using the reference option instead of String copies I may need
// to consider lifetime annotation for this function since I believe this doesn't
// get covered by rules 1-3. 
fn parse_config(args: &[String]) -> Config {
	let query = &args[1];
	let file_path = &args[2];

	Config {
		query,
		file_path,
	}
}
```

##### Option 3: A struct with a constructor
This has the benefit of directly associating the parse_config() function with the struct. 
```rust
// --snip--
struct Congig {
	file_path: &str,
	query: &str,
}

impl Config {
	fn new(args: &[String]) -> Config {
		let query = &args[1];
		let file_path = &args[2];

		Config {
			query,
			file_path
		}
	}
}
```

#### Extracting read_to_string
We'll forgo commentary here, the rationale is basically identical to the above. 
```rust
fn main() {
    // --snip--

    println!("Searching for {}", config.query);
    println!("In file {}", config.file_path);

    run(config);
}

fn run(config: Config) {
    let contents = fs::read_to_string(config.file_path)
        .expect("Should have been able to read the file");

    println!("With text:\n{contents}");
}

// --snip--

```

### Fixing error handling
One of the big issues with this code is that at no point do we handle the occurrence where the user supplies the wrong number of arguments. 

```rust
impl Config {
	fn build(args: &[string]) -> Result<Config, &'static str> {
		if args.len() != 2 {
			return Err(
				"Expected 2 arguments, query and file_path. Got {}",
				args.len(),
			);
		}
	let query = args[1].clone();
	let file_path = args[2].clone();

	Ok(Config {query, file_path})
	}
}
```

```rust
use std::process;

fn main() {
	let args: Vec<String> = eng::args().collect();

	// unwrap_or_else is how we can handle non-panic error handling. 
	// unwrap occurs if we have the value we expect and the else occurs
	// if we have an error.
	// This else statement is a closure. Which is an anonymous function (which 
	// I assume basically means a function with no formal definition).
	// Process::exit(1); closes the game much like panic does, but produces
	// the arror code you passed it. 
	let config = Config::build(&args).unwrap_or_else(|err| {
		println!("Problem parsing arguments: {err}");
		process::exit(1);
	})
}
```

Now that we have extracted the run component of our code (the actual file's text extraction). We can work on the error handling there too. 
```rust
use std::error::Error;

// --snip--

// We now return either the unit type (AKA nothing?)
// Or a dynamic error. 
// Box is a trait thing which means a function will return a type that implements
// the error trait. 
// Returning Ok(()) is used as a way to say that we are not trying to return 
// something with this potentially errorable function. For example all we care 
// about here is that this function prints the text and that's it. We never 
// use the contents again so we don't need to return its value. 
fn run(config: Config) -> Result<(), Box<dyn Error>> {
	// Remember the ? operator performs a similar role to a match case checkig 
	// for Ok() or Err()
    let contents = fs::read_to_string(config.file_path)?;

    println!("With text:\n{contents}");

    Ok(())
}

```

If we run the code in [[IO_project#Extracting read_to_string|read to string]] we will get a compile warning because at no point do we handle the potential error generated by run()
```rust
// --snip--
	// No error handling
	run(config);
// --snip--
```

To handle this error we can use the following:
```rust
fn main() {
	// --snip--
	// If we get an error run the code in the curly braces.
	// We aren't using unwrap_or_else here because of the lack of a meaningful
	// return value in run(config);
	if let Err(e) = run(config) {
		println!("Application error: {e}");
		process::exit(2);
	}
}
```