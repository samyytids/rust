### More about the new type pattern.
The [[advanced_traits#Using the new type pattern to implement external traits on external types.|new type pattern]] can also be used for type safety and abstraction.
New types can be used to provide a wrapper that limits the access to the private (inner) types API. Limiting the functionality that the end user has in case giving them full control may be unwise. 
It can also be used to create explicit types that like also used in the [[advanced_traits#Default generic type parameters and operator overloading|meter and millimeter new types]] we used before. This ensures that the u32 that we have stored within meter and millimeter is properly converted. If I just kept them both as u32 I would have to manually keep track of which variables are measured in millimeters vs meters. 
We can also hide the inner complexity of a type that isn't needed by the end user. Think something like a newtype that hides the fact that an id needs to be created for each instance of a piece of data added to a HashMap.  This way the user can add a property to this hashmap without ever needing to know about the internal ID that is used for say DB insertion. 

### Aliases
We can alias types which gives some of the benefit of the newtype pattern mentioned above, instead of wrapping the type to keep track of what is what we can use:
```rust
type kilometers = i32;
let x: i32 = 5;
let y: kilometers = 6;
```

Although this gives a visual indicator that y is in kilometers it doesn't implement any of the type safety that we get from the newtype wrapper. Meaning that I still need to make sure that I remember that y is measured in kilometers.

The main use for aliasing is avoiding code repetition with really long type signatures. 
For example when we were talking about [[trait_objects]] the syntax can get very unwieldly.
```rust
// This is a trait object that has a static lifetime, is Send and implements the 
// Fn trait. 
Box<fyn Fn() + Send + 'static>
```

Rewriting the above type over and over again is both tedious and potentially error prone, so we can instead use aliasing. Compare the below to segments of code. 
```rust
let f: Box<dyn Fn() + Send + 'static> = Box::new(|| println!("hi"));

fn takes_long_type(f: Box<dyn Fn() + Send + 'static>) {
	// --snip--
}

fn returns_long_type() -> Box<dyn Fn() + Send + 'static> {
	// --snip--
}

// This is both easier to read and write compared to the above. But does require
// knowledge of what Thunk is. 
type Thunk = Box<dyn Fn() + Send + 'static>;

let f: Thunk = Box::new(|| println!("hi"));

fn takes_long_type(f: Thunk) {
	// --snip--
}

fn returns_long_type() -> Thunk {
	// --snip--
}

```

### The never type.
The never type is mostly used as a way to highlight specific behaviours rather than to actually be used as a type. 
```rust
fn bar() -> ! {

}
```

The above basically says that the function bar never returns. 

#### Why do I want to use these though?
One reason that this might be useful is the scenario where I have a match statement that doesn't return the same type. 
```rust
// Note that num and contiue are not the same type. And the rust compiler insists
// that the type of guess is known at compile time, but we don't know that for
// certain yet. 
let guess: u32 = match guess.trim().parse() {
	Ok(num) => num,
	Err(_) => continue,
};
```

In this instance ! tells the compiler that the continue will never return a type, so the compiler can then determine that the value of guess will be a u32. The formal way of describing this is that ! can be coerced into any type. 
! is also used with [[panic|panics]], if we think about the [[option]] that we have seen many times before the code snippet below shows how option implements the unwrap method. 
```rust
impl<T> Option<T> {
    pub fn unwrap(self) -> T {
	    // The panic! macro returns an !, which tells the compiler that unwrap
	    // will only return the type T as in the other situation where panic! is 
	    // called panic! returns nothing and panics.
        match self {
            Some(val) => val,
            None => panic!("called `Option::unwrap()` on a `None` value"),
        }
    }
}
```

### Dynamically sized types and the Sized trait
By default generics will only accept types with a known size (memory allocation reasons). If we want a generic to take an unsized type we need to make sure we specify that. 

```rust
// This is the default implementation of anything with a generic, where we 
// specify that it needs to implement the sized trait. 
fn generic<T: Sized>(t: T) {
    // --snip--
}

// To use a dynamically sized type we need to use the following. 
fn generic<T: ?Sized>(t: &T) {
    // --snip--
}
```
