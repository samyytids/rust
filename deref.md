The deref trait is the trait that determines how an object behaves when the de-referencing operator (\*) is called. By implementing this trait with your own structs you can provide your own smart pointers. 

### Implementation
```rust
// This is a custom implementation of the Box smart pointer. (However, this stores data on the stack and not the heap). The backend of a Box is just a one element Tuple struct.
struct MyBox<T>(T);

// This is implementing an initializer method.
imple<T> MyBox<T> {
	fn new(x: T) -> MyBox<T> {
		MyBox(x)
	}
}

impl<T> Deref fir MyBox<T> {
	// Deref's associated type is the generic type T. 
	type Target = T;
	// This returns a reference to the value of type Target contained in the 0th
	// element of our tuple struct.
	// This works because on the backend * does the following: 
	// *(y.deref()).
	// This is done so that rust can use the * operand on anything that has the 
	// deref method rather than having to consciously know whether a deref method
	// needs to be called or not. Think butano .get(); which I have to manually
	// call for some of their inbuilt smart pointers. 
	fn defer(&self) -> &Self::Target {
		&self.0
	}
}
```

### Deref Coersions
To put it simply when you call a deref operator on a type if the type your argument/variable expects doesn't match the type provided by the dereference it will try to dereference the returned type repeatedly until we get a match. This is why you can provide &String to &str functions. &String can be further dereferenced to return a &str. 
Deref coersion also works across mutability, it can take a mutable reference to another type and take a mutable reference to a sub-type. 
- From `&T` to `&U` when `T: Deref<Target=U>`
- From `&mut T` to `&mut U` when `T: DerefMut<Target=U>`
- From `&mut T` to `&U` when `T: Deref<Target=U>`
