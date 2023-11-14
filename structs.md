### Syntax
Very similar to cpp.
```rust
// Definition
struct User {
	active: bool,
	username: String,
	email: String,
	sign_in_count: u64,
}

// Instantiation
fn main() {
	let mut user1 = User {
		active: true,
		username: String::from("samyytids"),
		email: String::from("STA.James@outlook.com"),
		sign_in_count: 1,
	};

	// Attribute access
	// Note, structs must either be entirely mutable or not mutable at all.
	user1.username = String::from("Sprunging");
}
```

### Update syntax
Update syntax copies/moves the value of one instance of a struct to another. This uses the assignment operator so comes with the same caveats mentioned in  [[ownership#Moving data]]. That being that stack variables can be copied while heap variables can only be moved. As such depending on what values you copy to the new instance you may invalidate the copy-ee instance.
```rust
fn main() {
	user1 = User {
		// Move
		username: String::from("samyytids"),
		email: String::from("STA.James@outlook.com"),
		// Copy
		active: true,
		sign_in_count: 1,
	}

	// This would invalidate user1
	user2 = User {
		username: String::from("sprunging"),
		..user1
	}

	// This would not
	user2 = User {
		username: String::from("sprunging"),
		email: String::from("sxj957@student.bham.ac.uk"),
		..user1
	}
}
```

### Storing references in structs
You can pass references to structs, however to do so you need to make use of [lifetimes](lifetimes.md). The use of lifetimes is required as they ensure that the data in the struct remains valid for as long as the struct remains valid. If this is not done we would have dangling references and as such if we pass references without lifetime information we will get compile errors.
```rust
struct ReferencedUser {
	active: bool,
	username: &str,
	email: &str,
	sign_in_count: u64
}

fn main() {
	// This will cause an error because of the lack of lifetime information.
	let user1 = ReferencedUser {
		active: true,
		username: "samyytids",
		email: "STA.James@outlook.com",
		sign_in_count: 1
	}
}
```

### Field init shorthand
If you have a function that creates an instance of a struct and it has arguments with identical names to the struct attributes you can directly assign attributes with those values without having to specify which attribute the value is being applied to. 

```rust
fn main() {
	let instance = create_instance("STA.James@outlook.com", "samyytids");
}

fn create_instance(email: String, username: String) -> User {
	// Remember you can return implicitly by ommitting the ;
	User {
		active: true,
		username,
		email,
		sign_in_count: 1,
	}
}
```

### Method syntax
Methods are the same as they are in any programming language, just a function within a class. However the syntax is sort of analogous to .h and .cpp pairs in cpp. 
```rust
#[derive(Debug)]
struct Rectangle {
	width: u32,
	height: u32,
}

// This is an implementation block.
// This just implements a function in the context of the name given.
// In this case we are implementing the area function (method) within the 
// context of our Rectangle type.
impl Rectangle {
	// &self is just shorthand for self: &self
	// which is itself shorthand for self: &TypeInImplBlock
	// This however is specifically a immutable reference to self
	fn area(&self) -> u32 {
		self.width * self.height
	}

	// This is the same principle but using a mutable reference to set an
	// attribute to a new value.
	fn set_width(&mut self, new_width: u32) {
		self.width = new_width;
	}
}

fn main() {
	let rect1 = Rectangle {
		width: 50,
		height: 50,
	};
	println!(
		"The area of this rectangle is {}",
		rect1.area()
	);
}
```

### Static method syntax
Static methods are methods that do not require an instance of the object. We have encountered these in [strings](string.md) with the String::from() method.
```rust
// --snip--
impl Rectangle {
	// Self as mentioned in method syntax, Self in this context is just
	// shorthand for the type in the impl block declaration.
	// This method would be called using Rectangle::square(5); and would 
	// return a Rectangle instance of dimensions 5x5.
	fn square(size: u32) -> Self {
		Self {
			width: size,
			height: size,
		}
	}
}
```

## Tuple structs
These add more meaning to tuples by giving them names, but doesn't ascribe specific names to the attributes.
```rust
struct Colour(i32, i32, i32);
struct Point(i8, i8);

fn main() {
	let black = Colour(0, 0, 0);
	let origin = Point(0, 0);
}
```

## Unit-like structs
These are structs with no fields.
These are useful when you want to implement a [trait](traits.md). But you don't have any data that you want to store in the type itself.
```rust
struct AlwaysEqual;

fn main() {
	// Although this will be covered later, brief bg. These can be used to 
	// provide behaviours without the need for data. The example given here
	// could have traits set such that subject always returns true when using 
	// ==. This can be useful for testing purposes. 
	let subject = AlwaysEqual;
}
```