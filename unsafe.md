Unsafe rust is rust that allows you to do things that are not ensured to be safe from the compiler. This doesn't mean that when writing unsafe code you can do whatever you want, there are still rules to rust. There are just some extra things that rust will allow you to do, now moving the onus of ensuring memory safety onto the programmer instead of the compiler. 
The main reason that you may wan to do this is because the compiler is inherently conservative, it would rather reject something it shouldn't rather than accept something that it shouldn't. As such there may be occasions where you will need to manually tell the compiler trust me bro, I know what I am doing.

#### What are these new things I can do?
1. Dereference a raw pointer.
2. Call an unsafe function or method.
3. Access or modify a mutable static variable.
4. Implement an unsafe trait.
5. Access fields of union S.

### Raw pointers
These are similar to your standard [[references]], they can be both mutable and immutable. They however take a different syntax for achieving the above.
```rust
let t = 5;
// The * in raw pointers is unrelated to the * operator, it is instead just part
// of the type signature.
// Immutable raw pointer.
let rp1 = &t as *const i32;
// Mutable raw pointer.
let rp2 =  &t as *mut i32;
```

#### What makes them special?
1. They can have both immutable and mutable pointers and they can have multiple mutable pointers as well.
2. No guarantee they point to anything valid.
3. Can be null.
4. No automatic clean up.

These abilities allow for potential performance gains relative to rusts safe behaviour as well as the ability to interface with other languages like C (C doesn't use the rules of rust, so communicating with C using rust's safe code won't work).

#### Using raw pointers
Note that in [[unsafe#Raw pointers|the code block above]] we aren't using an unsafe block to create these pointers. This is because we can create raw pointers in safe rust but we can't dereference them within a safe block.
```rust
fn main() {
	// Here we are declaring data that exists at an arbitray memory address that
	// we have no guarantee is valid and if it is valid we have no idea what is
	// there.	
	let arbitrary_address = 0x012345usize;
	// Here we are now creating a raw pointer to this memory and assume that it 
	// is an i32 datatype.
	// Dereferencing this pointer is inherently unsafe because we have no idea 
	// what is there. 
	let rpi = address as *const i32;
	// Note that we have now created an immutable pointer and a mutable pointer.
	// Since these are raw pointers this code will still compile. 
	let rpm = address as *mut i32;
	
	// Hence we have to run any dereference within an unsafe block.
	unsafe {
		// Running these prints may cause panics depending on what happens to be
		// at this address.
		println!("rpm is {}", *rpi);
		println!("rpm is {}", *rpm);
	}
}
```

### Unsafe methods and functions.
It only makes sense that unsafe functions can be called within unsafe blocks and not outside. However, there is a nuance here. Not all safe functions contain purely safe code. A safe function can contain unsafe code. This is referred to as a safe abstraction over unsafe code. This allows you to interact with unsafe elements of rust from a safe context. Here's an example where we try to get references to two separate parts of a slice.
```rust
use std::slice;

// This function takes a mutable referece to a collection and the position you // want to split the the collection at and then returns a tuple of the two
// "halves" of that collection.
fn split_at_mut(values: &mut [i32], mid: usize) -> (&mut [i32], &mut[i32]) {
	// The reason for this part here is that we need to check that the split
	// point is within the collection. If it isn't then our unsafe code would
	// actually be unsafe. So long as the split point is within the collection
	// the operation we are trying to make is still safe.
	let len = values.len();
	let ptr = values.as_mut_ptr();

	// Panics if the safety requirement isn't true.
	assert!(mid <= len);

	// Here we execute the "unsafe" code.
	unsafe {
		// Here we create the tuple of mutable references to the collection
		// that we return at the end of the function.
		(
			slice::from_raw_parts_mut(ptr, mid),
			slide::from_raw_parts_mut(ptr.add(mid), len - mid),
		)
	}
}

fn main() {
	let v = vec![1,2,3,4];
	// Creating a mutable reference to the entirety of the v vector.
	let r = &mut v[..];
	// Here is where the unsafe part kicks in. The below code will create two 
	// mutable pointers to the same slice. The compiler isn't smart enough to 
	// realise that this is safe because these two references do not overlap and
	// as such there's no risk of any data races or other multi-mutable reference
	// issues.
	// As such we the .split_at_mut() method needs to make use of unsafe code 
	// that is verified as safe by the programmer. The definition of this 
	// function is detailed above.
	let (a, b) = r.split_at_mut(2);
	assert_eq!(a, &mut [1,2,3]);
	assert_eq!(b, &mut [4]);
}
```

### Interacting with other languages
As was mentioned in the [[unsafe#Raw pointers#What makes them special?|raw pointers section]] other languages don't necessarily obey rust's rules, as such in order to interact with them we will need to use unsafe code. We can interact with other languages by using extern blocks.
```rust
// This provides us with access to the C function abs.
extern "C" {
	fn abs(input: i32) -> i32;
}

fn main() {
	// Obviously because C doesn't make any of the safety assertions that rust 
	// makes we have to call it within an unsafe block.
	unsafe {
		println!("Absolute value of -3 according to C: {}", abs(-3));
	}
}
```

We can also use extern blocks to call rust functions from other languages.
```rust
// No mangle disables some of the stuff that the rust compiler does. Mangling
// refers to the process of setting a compiler name for a function that contains
// more information than is given by the real name of the function. Obviously
// not all languages handle their mangling in the same way and as such we want to
// tell rust not to mangle the function name as we intend to call this function 
// from a C context instead.
#[no_mangle]
pub extern "C" fn call_from_c() {
	println!("Just called this from C");
}
```

### Accessing or modifying a mutable static variable
Time to clarify the differences between statics and constants. A constant can duplicate its data and fire it around our code base and they cannot be changed. In comparison statics can have their data changed (although this is unsafe) and the data always occupies the same place in memory. As such they cannot be copied. 
Statics are defined using SCREAMING_SNAKE_CASE.

```rust
static mut COUNTER: u32 = 0;

fn add_to_count(inc: u32) {
	unsafe {
		COUNTER += inc;
	}
}

fn main() {
	add_to_count(5);
	// Because we are accessing the value of a mutable static variable we have to
	// call the print in an unsafe block.
	unsafe {
		println!("Count is {}", COUNTER);
	}
}
```

### Implementing an unsafe trait
I may want to look at this in more detail at some point but in terms of just implementation, all you need to do is add the prefix of unsafe to both the trait declaration and the unsafe method within the trait itself. 
Note: A trait is considered unsafe if any of its methods are unsafe. 