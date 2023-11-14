### Characteristics
Strings are of an unknown size, unlike the types mentioned in [scalars and compounds](scalars_compounds.md). 

### Functionality#
from()
This creates a string (non-literal) from a given string literal.
```rust
fn main() {
	let s = String::from("Hello");
}
```

push_str()
```rust
fn main() {
	let mut s = String::from("Hello");
	s.push_str(", world!");
	println!("{}", s);
}
```


### Ownership
We have used string literals in lots of places. Think of the hard coded strings in [guessing_game](guessing_game.md), these are great when you know exactly what string you are going to need. But, they are immutable and as such are not suitable for all scenarios. 

