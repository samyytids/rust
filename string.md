### Characteristics
Strings are of an unknown size, unlike the types mentioned in [scalars and compounds](scalars_compounds.md). 

### Functionality#
#### from()
This creates a string (non-literal) from a given string literal.
```rust
fn main() {
	// This creates a string using the string lteral in the parentheses. 
	let s = String::from("Hello");
}
```

#### push_str()
```rust
fn main() {
	let mut s = String::from("Hello");
	// This will put the following string onto the end of the previous string.
	s.push_str(", world!");
	println!("{}", s);
}
```

#### push()
```rust
fn main() {
	let mut s = String::from("lo");
	// push effectively does the same thing as push_str, but takes a char as an
	// input rather than a String.
	s.push('l');
	
}
```

#### new()
```rust
fn main() {
	// This creates a new string with no initial value.
	let mut s = String::new();
	let data = "String literal";
	// This has similar functionality to String's from() method. But instead of 
	// setting the value using a literal the literal now returns a String tha you
	// can set the value of a String with.
	s = data.to_string();
}
```

#### Concatenation
```rust
fn main() {
	let s1 = String::from("Hello, ");
	let s2 = String::from("world");
	// Note because of the copy = move trait of heap objects s1 is no longer in
	// scope and is as invalid. 
	// The + operator in this instance is a function that takes a string and a
	// slice. Slice because &String is coerced into &str via &str[..].
	// This may seem complex but it is more efficient than producing copies of
	// all the strings.
	let s3 = s1 + &s2;
}
```

#### Indexing
```rust
fn main() {
	let s1 = String::from("hello");
	// This will cause an error. 
	// Rust doesn't support indexing by integer. 
	// The reason that is the case is because of non-UTF8 characters. 
	// If I did an index like this on Russian characters I would get incorrect
	// values because some Russian characters are represented by numbers. 
	// let hello = "Здравствуйте";
	// See Bytes, scalar and grapheme clusters for anothe reason why it isn't
	// allowed. 
	// A final reason is that we cannot guarantee an 0(1) lookup speed with
	// strings because of the variable nature of their character sizes. As such
	// rust would need to exam all the characters on the way to the index to 
	// measure what index it is at. 
	
	let h = s1[0];
}
```

#### Iterating
```rust
fn main() {
	let string = String::from("Hello");
	// This assumes that your strings are best split by chars.
	for c in string.chars() {
		println!("{c}");
	}

	// This assumes that your strings are best split by bytes.
	for b in string.bytes() {
		println!("{b}")
	}

	// Doing the above with grapheme clusters is complex and as such 
	// is not part of the standard library.
}
```

### Ownership
We have used string literals in lots of places. Think of the hard coded strings in [guessing_game](guessing_game.md), these are great when you know exactly what string you are going to need. But, they are immutable and as such are not suitable for all scenarios. 

### Bytes, scalar values and grapheme clusters
These are the three ways that Strings can be represented in rust. 
To tldr; 
Some languages don't split well into bytes or scalars. You end up with meaningless sub-letters (think extracting stroks 3-7 of a 12 stroke kanji). As such rust allows programmes to use whatever is the most appropriate option depending on its context. 
