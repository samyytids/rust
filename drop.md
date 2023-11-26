This is a trait that controls what code is ran when an object goes out of scope. 
You cannot manually call the drop function, this means that in order to manually drop a pointer early you would need to use [std::mem::drop](mem_drop.md)

### Implementation
```rust
struct CustomSmartPointer {
	data: String
}

impl Drop for CustomSmartPointer {
	// This means that when the CustomSmartPointer goes out of scope it executes
	// the print statement that specifies what is within the pointer. 
	fn drop(&mut self) {
		println!("Dropping this value with data {}", self.data);
	}
}

fn main() {
	let c = CustomSmartPointer {
		data: String::from("my stuff");
	}
	let d = CustomSmartPointer {
		data: String::from("Not my stuff");
	}
	println!("My stuff has been created.");
}
```