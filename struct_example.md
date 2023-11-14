width * heightLet's start by using no structs at all.
```rust
fn main() {
	let width1 = 50;
	let height1 = 50;

	println!(
		"The area of this rectangle is {} square pixels.",
		area(width1, height1)
	);
}

fn area(width: u32, height: u32) -> u32 {
	width * height
}
```

Now let's use tuples, this introduces a relationship between the two values. IE we know they are related to one another unlike the example above. Downside however is that due to the lack of naming it is hard to infer what is what and requires knowledge of the structure of the tuple's values to use the right values in the right places. 
```rust
fn main() {
	let rect1 = (50,50);
	println!(
		"The area of this rectangle is {} square pixels.",
		area(rect1)
	);
}

fn area(dimensions: (u32,u32)) -> u32 {
	dimensions.0 * dimensions.1
}
```

Now let's use structs, this keeps the related nature but also adds naming to make it much more obvious what is happening and how things are related and also requires less explicit knowledge of the structure of the data (names are easier to track than indexes).
```rust
struct Rectangle {
	width: u32,
	height: u32,
}

fn main() {
	let rect1 = Rectangle {
		width: 50,
		height: 50,
	};

	println!(
		"The area of this rectangle is {}",
		area(&rect1)
	);
}

fn area(rectangle: &Rectangle) -> u32 {
	rectangle.width * rectangle.height
}
```

Simple way of adding a string representation of a struct
```rust
#[derive(Debug)]
struct Rectangle {
	width: u32,
	height: u32,
}

fn main() {
	let rect1 = Rectangle {
		width: 50,
		height: 50,
	};

	// :? is a special way of formatting a string. Instead of printng the raw
	// value :? says we want to use the Debug format. 
	// But to use the Debug format we must use the #[derve(Debug)] on our struct
	// I assume derive automatically generates a string representation of our 
	// struct. I also assume that I could do this manually if I wanted to.
	println!("Rect1 is {:?}", rect1);
	// This will print the following:
	// rect1 is Rectangle { width: 50, height: 50 }
	println!("Rect1 is {:#?}", rect1);
	// This does effectively the same thing as :? but uses pretty print.
	// rect1 is Rectangle {
	//     width: 50,
	//     height: 50,
	// }
}
```