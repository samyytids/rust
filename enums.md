### Syntax
```rust
// Definition.
enum IpAddrKind {
	V4,
	V6,
}

// Instantiation.
fn main() {
	// Because of the way that enums are defined we can create functions 
	// that can take a generic IpAddrKind value. Which means it can process
	// both V4 and V6. I imagine with nested enums you could provide a function
	// with say IpAddrKind::V6 and it would be able to use any sub-enum but
	// would not be able to take a IpAddrKind::V4.
	let four = IpAddrKind::V4;
	let six = IpAddrKind::V6;
}
```

### Using enums with data

#### struct
We can use a struct to relate data to an enum type.
```rust
enum IpAddrKind {
	V4,
	V6,
}

struct IpAddr {
	kind: IpAddrKind,
	address: String,
}

fn main() {
	// This let's us know the type of the ipaddr as well as the ipaddr itself.
	let addres1 = IpAddr {
		kind: IpAddrKind::V4,
		address: String::from("127.0.0.1"),
	};
}
```

#### pure enum
But, we can also do this within the enum itself. 
```rust
enum IpAddr {
	V4(String),
	V6(String),
}

fn main() {
	let home = IpAddr::V4(String::from("127.0.0.1"));
}
```

### enum methods
Enums in rust can have methods
```rust 
enum Message {
	Quit,
	Move { x: i32, y: i32},
	Write(String),
	ChangeColour(i32, i32, i32),
}

impl Message {
	fn call(&self) {
		// Do something
	}
}

fn main() {
	let m = Message::Write(String::from("hello"));
	m.call();
}
```

### nested enums
These are enums where one (or more) of our enums contain data types from other enums
```rust
#[derive(Debug)]
enum UsState {
    Alabama,
    Alaska,
    // --snip--
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}

```