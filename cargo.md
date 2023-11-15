```bash
cargo run
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
```
Create a new rust project.
--lib will create the project as a library project.