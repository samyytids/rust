### Syntax
Pretty simple!
The type annotation is technically optional, however it is required if you are not going to instantiate the vector with values.
```rust
let v: Vec<i32> = Vec::new();
// Below is a usage of the vector macro which will instantiate a vector 
// containing the values provided in the initialiser list. 
let v = vec![1,2,3];
```

### Populating a vector
```rust
// I believe that the type annotation can be dropped here because rust looks
// forward during compilng and infers the type of the vector based on the values
// pushed into it.
let mut v = Vec::new();

v.push(5);
v.push(6);
v.push(7);
v.push(8);
```

### Accessing vector elements
```rust
let v = vec![1,2,3,4,5,6];

// This option will cause the programme to panic if we try to access an element
// that does not exist within the vector.
let third: &i32 = &v[2];

// This will not panic because it uses the Option wrapping. 
let thirds: Option<&i32> = v.get(2);

match third {
	Some(third) => println!("The third value is: {third}");
	None => println!("There is no third value.");
}
```
The choice between these two methods depends on the logic you want your programme to have. 

#### Ownership concerns
We cannot make changes to the vector once a reference to one of its elements has been created and remains in scope. The reasoning for this is that if we were to push an element into a vector it may be too large for its current place in memory and as such need to be moved. This would mean that out reference would now point to de-allocated memory.

Once a vector leaves scope so do its elements.

### Iterating
```rust
// we can perform modifications on the elements in a vector but we cannot edit
// the vector itself. IE vec[1] + 1 = fine. vec.push_back(!) != fine.
let mut v = vec![1,2,3];
for i in &mut v {
	// In order to make adjustments to the values in a vector in this manner 
	// we need to de-reference the element.
	*i += 1;
}
```

### Vectors of enums
As was mentioned in [[enums]] the base type of en enum allows you to store multiple different datatypes in data structures that can normally only contain one datatype.
```rust
enum Example {
	Int(i32),
	Float(f64),
	Text(String),
}

let example_vector = vec![
	Example::Int(4),
	Example::Float(5.4),
	Example::Text(String::from("Text")),
]
```

### Vectors of traits
These can also be used to create vectors that can contain differing types. [[traits|Traits]] however can be used when you don't know what types might be in the vector at compile time. Unlike enums which require you to know which types might be in the vector even if they aren't all going to be the same type.