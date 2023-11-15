### Characteristics
A hash map consists of 2 "untyped" parts keys of type K and values of type V. 

### Functionality
### new()
```rust
use std::collections::HashMap;

let mut scores = HashMap::new();
```

#### insert()
```rust
// --snip--
scores.insert(String::from("Blue"), 10);
scores.insert(String::form("Yellow", 50));
```

#### get()
```rust
// --snip--
let s = String::from("Blue");

// .get() takes a reference to the key you want to look up, it returns an Option
// wrapped reference to the associated value or None.
// .copied() is a functionality of Option which just returns the value within the
// Option.
// .unwrap_or() sets the score to 0 if there is no value. 
scores.get(&s).copied().unwrap_or(0);
```

#### iteration
```rust
// --snip--
for (key, value) in &scores {
	println!("{key}: {value}");
}
```

#### entry()
```rust
// --snip--
scores.insert("Blue", 32);
scores.entry(String::from("blue")); // -> Entry sounds like it's similar to 
									// Option
```

### Ownership
If the values in the hash map are copyable then they are simply copied into the hashmap and the original value maintanis its validity.
If the values in the hash map are not copyable and instead invoke move then they behave as detailed in [[stack_and_heap#Heap|heap]].
This same logic applies to the keys.

### Updating a hash map
Rust by default overwrites the value associated with a key if the key you are inserting already exists. 
```rust
// --snip--
scores.insert("Red", 10);
scores.insert("Red", 12);
println!("{:?}", scores); // {"Red": 12}
```

You can also check if a key exists already and then do something if the key doesn't exist already.
```rust
// --snip--
// IF the entry Yellow doesn't exist then we insert the value 50.
scores.entry(String::from("Yellow")).or_insert(50);
```

You can also update the value based on the previous value.
```rust
use std::collections::HashMap;

let text = "hello world wonderful world";

let mut map = HashMap::new();

for word in text.split_whitespace() {
	// This either sets the count value to 0 if the word doesn't already appear
	// in the map.
	// Otherwise it returns the old count and increments it by 1. 
	let count = map.entry(word).or_insert(0);
	// This dereferences the count reference we get from .entry() and increments
	// it by 1, this change is reflected in the hashmap.
	*count += 1;
}

println!("{:?}", map);

```
