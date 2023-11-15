At it's very core, a test function is a function that's annotated with the test attribute [[structs#Method syntax|(we've used annotations here)]]. 
To create a test module use [[cargo]] test, this will create a module where you can set up all your tests. This module can contain non-test functions but obviously only include non-test functions that are needed as part of your test functions.

### Syntax
```rust
#[cfg(test)] // This basically marks this moduel as only needing to be compiled
			 // when you run cargo test and not when you run cargo build/run.
mod tests {
	// Indicates that this function is a test.
	#[test]
	fn test_1() {
		let result = 2 + 2;
		// The assertion that we need to pass in order for our test to succeed.
		assert_eq!(result, 4);
		// assert_ne! is used for != assertions.
	}

	#[test]
	fn larger_can_hold_smaller() {
		let larger = Rectangle {
			width: 8,
			height: 10,
		}
		let smaller = Rectangle {
			width: 6,
			height: 9;
		}
		// the default assert macro works with boolean values
		assert!(larger.can_hold(&smaller));
	}

	#[test]
	fn smaller_cannot_hold_larger() {
		let smaller = Rectangle {
			width: 1,
			height: 1,
		}
		let larger = Rectangle {
			width: 5,
			height: 5,
		}
		// Note the ! before the assert param to signify negation.
		assert!(!smaller.can_hold(&larger));
	}
}
```

### Custom failure messages
```rust
    #[test]
    fn greeting_contains_name() {
        let result = greeting("Carol");
        assert!(
            result.contains("Carol"),
            // The first argument is a context providing string with placeholders
            "Greeting did not contain name, value was `{}`",
            // This/these are the arguments to be put into the placeholders, this
            // can be used to provide more specific context as to what should
            // have happened but didn't.
            result
        );
    }

```

### Checking for situations where the programme should panic
```rust
pub struct Guess {
	value: i32,
}

impl Guess {
	pub fn new(value: i32) -> Guess {
		if value < 1 || value > 100 {
			// We would likely want to be more specific about how we handle the
			// panic to make sure we know why we are panicking to ensure that 
			// this panic is actually the panic we want to happen and not some
			// other issue.
			panic!("Unacceptable value: {}", value);
		}

		Guess {value}
	}
}

// This version is better as it breaks down the panic scenarios so that we can 
// make sure that we know exactly what panic is occurring.
impl Guess {
    pub fn new(value: i32) -> Guess {
        if value < 1 {
	        // the panic checks if the error contains the text from this panic
	        // as such we can get a fail based on the specific 1 bound.
            panic!(
                "Guess value must be greater than or equal to 1, got {}.",
                value
            );
        } else if value > 100 {
            panic!(
                "Guess value must be less than or equal to 100, got {}.",
                value
            );
        }

        Guess { value }
    }
}


#[cfg(test)]
mod tests {
	use Super::*;

	#[test]
	// This makes the test anticipate a panic, as such we pass if it does and
	// fail otherwise.
	#[should_panic]
	fn greate_than_100() {
		Guess::new(200);
	}

}
```

### Using [[result|Result]]<T,E> in tests
Same basic idea as using assert, but instead a fail state is based on the error state and a pass state is based on the ok state. 
We can inverse this logic by combining it with assert!(value.is_err());
```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() -> Result<(), String> {
        if 2 + 2 == 4 {
            Ok(())
        } else {
            Err(String::from("two plus two does not equal four"))
        }
    }
}
```

### Controlling tests

#### Consecutive vs Sequential execution
Tests by default are ran in parallel, we can opt to run the consecutively. Using, 
```rust
cargo test -- --test-threads=1
```

This can be useful if your tests create some file, instead of having to create the file and then recreate and refill it over and over, you can simply have your tests run in order. 

#### Show println! output
Tests by default suppress print statements in successful tests.
```rust
$ cargo test -- --show-output
```

#### Running specific tests
You can run a specific test or specific tests by providing their name or part of their names. 
```rust
fn entity_test_1
fn entity_test_2
fn entity_test_3
```

```bash
cargo test entity_test_1 // This will run only entity_test_1.
cargo test entity // This will run all tests with the prefix entity.
```

#### Ignore a test by default unless requested
```rust
#[test]
fn this_is_a_test() {}

// The ignore decorator means this test will be ignored unless the --ignored
// option is used. 
#[test]
#[ignore]
fn this_is_also_a_test() {}
```

```bash
cargo test -- --ignored
```

### Test organisation
Rust by default allows you to directly test private functions although "there is some debate as to whether private functions should be tested directly."

Rust splits tests into 2 primary categories:
#### Unit tests
These are tests that isolate a specific functionality to see if it behaves as anticipated. 
The general convention is to have all your tests in your files that are in the source folder and have a tests module at the top of said file. 
```rust
#[cfg(test)]
mod tests {
	// Your tests go here.
}
```


#### Integration tests
These tests are entirely external to your library, they call functions to test in the same way that any other code would. Your integration tests should be held in a separate file within a tests folder. 
```text
adder
├── Cargo.lock
├── Cargo.toml
├── src
│   └── lib.rs
└── tests
    └── integration_test.rs
```

Integration tests do not need to be declared inside a module with a cfg(test) decorator. The separate folder is inherently treated as a cargo test only folder. But the individual functions do need to be decorated with the test decorator. 

If we want to use helper functions such as a set up function within our integrated tests, we shouldn't create them in a file within our tests folder as that will make the compiler think the file should be included in your testing and as such will fill your test output with garbage.
Instead use the following structure with common mod.rs as an example file containing helper functions.

```text
├── Cargo.lock
├── Cargo.toml
├── src
│   └── lib.rs
└── tests
		// Not this uses the older format of structuring modules into separate 
		// files. 
    ├── common
    │   └── mod.rs
    └── integration_test.rs
```

### Binary crates
Binary crates can't implement integration tests because you cannot import modules from other crates that do not have a lib.rs file. As such it makes sense to separate your module structuring and actual running code into a main.rs and a lib.rs file. 
