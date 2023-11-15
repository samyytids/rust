```bash
cargo run
cargo run -- arg1 arg2 // syntax for providing arguments via the cli when 
					   // running a programme. 
```
runs our programme

```bash
cargo build

```
builds our programme
If your programme has no main function instead of producing a binary crate you create a library crate. 
What this basically boils down to is you create a series of functions that can be accessed by other projects rather than creating a project in and of itself. 

```bash
cargo update
```
updates our dependencies

```bash
cargo doc --open
```
creates documentation for our dependencies and opens it in our browser.

```bash
cargo new project-name
cargo new library-project --lib
```
Create a new rust project.
--lib will create the project as a library project.

```bash
cargo test
cargo test --help
cargo test -- --help

cargo test -- --test-threads=1 // Sequential instead of parallel execution.
cargo test -- --show-output // Prints even if the function passes.
cargo test function_name // Runs this specific function.
cargo test function_preffix // Runs all functions with this prefix.
cargo test -- --ignored // Runs only the ignored tests. 
```
Creates a test runner binary and runs our functions annotated with the test annotation [[test_functions]]. 
Both --help and -- --help provide a list of potential options, they just provide different "depths" of options.