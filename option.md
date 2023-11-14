Option is an enum that handles the functionality of something either having a value or having no value. Think bn::optional from butano.

### Syntax
```rust
// <T> is a generic type parameter. Think of this as a template decorator in cpp.
// It basically means that any type can be passed to this enum.
// All the types in this enum are included in the prelude (prelude just means 
// that you don't need to use anything to use them).
enum Option<T> {
	None,
	Some(T),
}

fn main() {
	let some_number = Some(5);
	let some_char = Some('e');
	// Explicit type declaration required so the compiler knows what should be
	// put into null_number.
	let null_number: Option<i32> = None;

	let number: i8 = 8;
	let some_number: Option<i8> = Some(8);

	// This will cause an error because i8 and Option<i8> are not technically of
	// the same type and some_number may actually be None.
	let sum = number + some_number;
}
```

### Functionality
#### is_some()
This returns a boolean value if the Option contains a value. 
```rust
let x: Option<i8> = Some(8);
let y: Option<i8> = None;

x.is_some(); // -> true
y.is_some(); // -> false
```

#### is_some_and()
This returns a boolean value if the Option contains a value and that the value matches some costraint.
```rust
let x: Option<i8> = Some(8);
let y: Option<i8> = None;
let z: Option<i8> = Some(9);

x.is_some_and(x == 8); // -> true because some and 8 == 8
y.is_some_and(y == 8); // -> false because None
z.is_some_and(z == 8); // -> false because some and 9 != 8
```

.
.
.

