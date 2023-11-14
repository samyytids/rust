Although rust can infer typing depending on the value provided and what we do with the value. There are times where explicit type annotation is required. 
For example from the [guessing game](guessing_game.md) when we convert the guess string to a number we need to specify what kind of number or else rust will get angry.

### Scalar types
These represent single values. Rust has 4 primary scalar types.
#### Integers

| Length | Signed | Unsigned |
|--------|--------|----------|
| 8-bit  | i8     | u8       |
| 16-bit | i16    | u16      |
| 32-bit | i32    | u32      |
| 64-bit | i64    | u64      |
| 128-bit| i128   | u128     |
| arch   | isize  | usize    |

Number range for signed integers is as follows 
$-(2^{n-1})$ to $2^{n-1}-1$ inclusive.

Unsigned integers.
$0$ to $2^n-1$ 

Arch uses the architecture of your system eg 32 or 64 bit.

Integers allow for type suffixing such that 57u8 implies 57 in this instance of an 8 bit unsigned integer.

You can also write number literals in any of the following formats: 

|Number literals|Example|
|---|---|
|Decimal|`98_222`|
|Hex|`0xff`|
|Octal|`0o77`|
|Binary|`0b1111_0000`|
|Byte (`u8` only)|`b'A'`|

If we experience integer overflow rust will [panic](panic.md).

#### Floating points

Rust has 2 primitive types for floating point numbers. 
```rust
fn main() {
	let x = 2.0; // defaults to f64 (f32 is sort of redundant with modern CPUs)
	let y: f32 = 3.0;
}
```

For both integers and floating point numbers rust supports the standard mathematical operators.
```rust
fn main() {
	let multiply = 1 * 1;
	// Note much like cpp if you divide using integers your result will be an 
	// integer and by default will be rounded down.
	let divide = 1 / 1;
	let add = 1 + 1;
	let subtract = 1 - 1;
	let remainder = 1 % 1;
}
```

#### Booleans
No real need to talk about these much.
```rust
fn main() {
	let t = true;
	let f: bool = false;
}
```

#### Characters
Rust can also represent emojis as characters. Delineation between string and char is done in the same was as it is in cpp "" vs ''.
```rust
fn main() {
	let c = 'z';
	let z: char = 'Z';
	let heart_eyed_cat = '😻';
}
```


### Compound types

#### Tuples
The elements of a tuple can all be of differing types but they are also of a fixed length! However, these types are fixed upon instantiation.
```rust
fn main() {
	let x: (u8, u32, i16) = (255, 1234, -256);
}
```

You can also unpack the values of a tuple in a similar fashion to python.
Unpacking is referred to as destructuring in rust.
```rust
fn main() {
	let tup = (500,64,1);
	let (x,y,z) = tup;
	println!("The vaue of y is: {y}");
}
```

Accessing elements is done using . notation rather than dict notation.
```rust
fn main() {
	let x: (u8,u8,u8) = (1, 2, 3);
	let one = x.0;
	let two = x.1;
	let three = x.2;
}
```

#### Array
Unlike tuples array elements must all be of the same type and arrays are of a fixed length. Arrays unlike tuples are instantiated on the stack rather than on the heap. [stack/heap](stack_and_heap.md). Vectors are more flexible but they will be dealt with later.
```rust
fn main() {
	let a = [1,2,3,4,5];
}
```

Other ways to instantiate an array:
```rust
fn main() {
	// This is an explicit declaration of the type and nuber of elements of this
	// array.
	let a1: [i32; 5] = [1,2,3,4,5];
	// This is creating an array containing the number 3, 5 times.
	let a2 = [3;5];
}
```

Accessing elements in arrays is done using dictionary notation.
```rust
fn main() {
	let a = [1,2,3,4,5];
	let one = a[0];
	let two = a[1];
}
```

If you try to access an out of bounds index rust will [panic](panic.md)
