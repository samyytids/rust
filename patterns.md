Patterns appear in numerous places beyond just [[matching]]. 

### Examples

#### If let
```rust

let favorite_color: Option<&str> = None;
let is_tuesday = false;
let age: Result<u8, _> = "34".parse();

// If let uses a refutable pattern, IE it can be refuted so it can handle not 
// succeeding. If you pass it an irrefutable pattern you will get a warning 
// because using an irrefutable pattern with an if let is pointless, you may as
// well just use a standard let (spoiler let is also a pattern).
// What we haven't covered before is that you can chain if lets. The main benefit
// of this compared to a match is that it allows you to use multiple different
// conditions on different values and they don't need to be exhaustive, we will 
// talk about how match guards can achieve this too later. 

// This arm executes if we have provided a favourite colour. IE the Option has a 
// Some value and is not None. So, this time it does not execute.
if let Some(colour) = favourite_colour {
	prinltn!("Using your favourite colour, {colour}")
	// This executes if is_tuesday is true, which it isn't. 
} else if is_tuesday {
	println!("because it's Tuesday we are using green");
	// This executes if our age has been provided a value. Which since we are 
	// parsing a string will occur only if the string can be parsed into a valid
	// integer. 
} else if let Ok(age) = age {
	// We then perform a check based on the value we provided in the Ok. 
	// This is what actually executes since our age can be parsed and is over 30.
	if age > 30 {
		println!("you're old so we are using grey");
	} else {
		println!("You're young so we are using baby blue");
	}
	// Simple else used to make sure this block always evaluates to something.
} else {
	println!("well screw you we are using brown");
}
```

#### While let
```rust
let mut stack = vec![3,2,1];

// While let similar to if let requires a refutable pattern, this is refutable 
// because the stack.pop() won't always return a Some. It could return a None.
// In this instance we effectively pop each element and one by one print out the
// top element of our stack. 
// Once we have run out of elements on the stack then stack.pop() will return 
// None and as such the while loop will stop. 
while let Some(top) = stack.pop() {
	println!("{}", top);
}
```

#### For loop
```rust
let v = vec!['a', 'b', 'c'];

// This pattern within the for loop effectively destructures the tuple produced
// by the enumerate .method.
// It breaks down the result into 2 named variables index and value.
for (index, value) in v.iter().enumerate() {
	println!("{} is at index {}", value, index);
}
```

#### Let
```rust
// Let requires an irrefutable pattern as it can't handle being passed something 
// that can fail. 
let x = 5;
// Let can be a bit more complex as much like the for loop above it can be used 
// destructure results into separate variables. 
// This splits the tuple into 3 variables x, y, z. 
// The number of variables on the left and right need to match up. 
let (x,y,z) = (1,2,3);
```

#### Function parameters
```rust
// In both instances here the function parameters can be thought of as a pattern
// which means that we can use the pattern tricks mentioned later on in this c
// chapter.
fn foo(x: i32) {

}

fn print_coordinates(&(x,y): &(i32, i32)) {
	prnitln!("current location: ({},{})", x, y);
}
```

### Refutable and irrefutable
Simply put in some scenarios it is completely fine for a pattern to be able to fail but it isn't allowed in others. 
Refutable patterns are those that are allowed to fail while irrefutable are those that are not allowed to fail. 

#### Let
```rust
// The let pattern can't be refutable as it would lead to None values and bad
// ju ju.
// This doesn't mean that you can't store somes, just the the variable in the let
// itself cannot be instantiated as a Some on the left hand side. 
// The right hand side can be Some. 
// As such this won't compile.
let Some(x) = a_function_that_returns_something();
```

#### If let
```rust
// On the other hand if let doesn't like being given irrefutable patterns.
// Primarily because the use of an if let is pointless if the pattern is 
// irrefutable. 
if let x = 5 {
	println!("{}", x);
}
// What's the point in using an if let here? you would ge the same functionality
// with a simple let. 
let x = 5;
println!("{}", x);
```

### Pattern syntaxing

#### matching literals

```rust
let x = 1;

match x {
	1 => prnitln!("one"),
	2 => prnitln!("two"),
	3 => prnitln!("three"),
	4 => prnitln!("four"),
	// _ is widely used as the chatchall in rust. 
	_ => println!("Something else"),
}
```

#### matching named variables
```rust
let x = Some(5);
let y = 10;

match x{
	Some(50) => println!("50"),
	// Note that we are matching x here, so x is being matched to Some(var) not
	// matching y. So this will print matched y = 5 not match y = 10.
	// This is because named variables in match statements shadow those in the 
	// scope they are in. 
	Some(y) => println!("matched y = {y}"),
	_ => println!("Default case, x = {:?}", x),
}
```

#### Multiple patterns
```rust
let x = 1;
match x {
	1 | 2 => println!("one or two"),
	3 => println!("three"),
	_ => println!("something that isn't 1, 2 or 3."),
}
```

#### Matching ranges of values with ..=
```rust
// We may remember ..= from way earlier on in this journey, this details an 
// inclusive range. 
let x = 5;

match x {
	1..=5 => println!("one through 5"),
	_ => println!("something else"),
}

// This kind of range matching can be used with chars too!
let x = 'c'

match x {
	'a'..='j' => println!("Early letter."),
	'k'..='z' => println!("late letter."),
	_ => println!("Not even a letter."),
}
```

#### Struct destructuring
```rust
struct Point {
	x: u8,
	y: u8,
}

fn main() {
	let p = Point {x: 0, y: 7};

	// What this is doing is breaking down the point within the struct into 
	// two separate variables a and b.
	let Point {x: a, y: b} = p;
	assert_eq!(0,a);
	assert_eq!(7,b);

	// This can be abbreviated by using the same variable names within the 
	// struct.

	let Point {x, y} = p;
	assert_eq!(x,0);
	assert_eq!(y,7);
}
```

#### Using a combination of generics and specifics in a match
```rust
let p = Point {x: 0, y: 7};

match p {
	// We don't care about the x value here, we only care if the y = 0.
	Point {x, y: 0} => println!("The point is on the x axis at {}", y),
	// We don't care about the y value here, we only care if the x = 0.
	Point {x: 0, y} => println!("The point is on the y axis at {}", x),
	// This is effectively our catch all here. 
	Point {x,y} => println!("The point is not on either axis {x}, {y}"),
}
```

#### Destructuring enums that contain data.
```rust
enum Message {
	Quit,
	Move { x: u8, y: u8},
	Write(String),
	ChangeColour(i32, i32, i32),
}

fn main () {
	let msg = Message::ChangeColour(255,255,255);
	
	match msg {
		Message::Quit => {
			println!("The quit variant has no information to destructure")
		},
		Message::Move { x, y } => {
			println!("We can destructure the move, {x}, {y}")
		},
		Message::Write(text) => {
			println!("Message is: {text}")
		},
		Message::ChangeColour(r, g, b) => {
			println!("The new colour is, {r}, {g}, {b}"),
		}
	}
}
```

#### Destructuring nested enums
```rust
enum Color {
    Rgb(i32, i32, i32),
    Hsv(i32, i32, i32),
}

enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(Color),
}

fn main() {
    let msg = Message::ChangeColor(Color::Hsv(0, 160, 255));

    match msg {
	    // Pretty simple, you specify the enum arm and then specify the inner
	    // data and how it should be destructured and what matters. 
	    // In this case we don't care about the specific contents. 
        Message::ChangeColor(Color::Rgb(r, g, b)) => {
            println!("Change color to red {r}, green {g}, and blue {b}");
        }
        Message::ChangeColor(Color::Hsv(h, s, v)) => {
            println!("Change color to hue {h}, saturation {s}, value {v}")
        }
        _ => (),
    }
}
```

### Destructuring structs and tuples
```rust
// simple combination of destructuring tuples and structs at the same time.
let ((feet, inches), Point { x, y }) = ((3, 10), Point { x: 3, y: 10 });
```


### Ignoring patterns with \_.

#### Ignoring function arguments.
```rust
// This isn't really useful as an actual end code solution, but it is good for
// when you are debugging and checking code. 
// For example: If I am in the process of writing this code and I know what I 
// will have as inputs but I haven't completed the logic yet. I can avoid unused
// param warnings by substituting the variable name with an _.
fn foo(_: i32, y: i32) {
	println("This is a print {}", y);
}

fn main() {
	foo(3, 4);
}
```

#### Ignoring parts of patterns.
```rust
let mut setting_value = Some(5);
let new_setting_value = Some(10);

match (setting_value, new_setting_value) {
	(Some(_), Some(_)) => {
		println!("If both have a value then we do nothing, we don't care what
		either value is though.");
	}
	_ => {
		// In any other situation we setting the old setting value to the new 
		// value.
		setting_value = new_setting_value;
	}
}

let numbers = (1, 2, 3, 4, 5, 6);

match numbers {
	(first, _, third, _, fifth, _) => {
		println!("Some numbers {first}, {third}, {fifth}");
	}
}
```

### Ignoring variables that you aren't using yet.
```rust
let _x = 5;
let y = 10;

// Normally since the x variable isn't used you would get an error/warning about
// unused variables. But, by prefixing the name with an underscore you tell the 
// compiler that you know that this value hasn't been used.
// The main difference between this and _ is that a pure _ doesn't store any 
// value while _x still binds values but just specifies that it hasn't been used
// yet.
foo(y);
// As such the below won't compile.
fn main() {
	let x = Some(5);
	if let Some(_x) = x {
		println!("Number exists");
	}

	// This won't compile because the value in x is now stored in _x.
	println!("This won't compile {}", x);

	// But this will compile. 
	let s = Some(String::from("Hello!"));

    if let Some(_) = s {
        println!("found a string");
    }

	// Because the value in s cannot be bound to _ it simply matches the pattern
	// and executes the code and leaves the value in s. 
    println!("{:?}", s);

}
```

### Ignoring slices of patterns using .. .
```rust
struct Point3D {
	x: i32, 
	y: i32,
	z: i32,
}

fn main() {

	let p = Point { x: 3, y: 4, z: 5 };
	match p {
		// In this case .. is used to say that I don't care about anything in
		// this struct after the parameter x.
		Point { x: 3 .. } => {
			println!("This point lies at 3 on the x axis");
		}
		Point { x .. } => {
			println!("This point does not");
		}
	}
}
```

#### Limitations of ..
```rust
fn main() {
	let tuple = (1, 2, 3, 4);

	match tuple {
		// The reason this doesn't work is because the compiler doesn't know 
		// which element in the tuple is represented by third. 
		// .. doesn't give any instruction of how many to skip, it just says to
		// skip. As such it needs to be surrounded in values in order to be
		// compilable.
		(.., third, ..) => {
			println!("This match doesn't work");
		}
		(.., last) => {
			println!("This math does work");
		}
		(first, ..) => {
			println!("This match also works");
		}
		(first, .., third, last) => {
			println!("This match also also works");
		}
	}
}
```

### Match guards
Match guards are simply match statements that contain an internal if statement, these allow for some extra functionality not available with normal match statements but they also have their drawbacks. For example we lose the exhaustive check from the compiler. But we can now use multiple values within the same match block.

```rust
fn main() {
	let x = Some(6);
	match x {
		Some(_) if x % 2 == 0 => {
			println!("Executs if x is a Some value and said value is divisible
			by 2");
		}
		_ => println!("Catchall"),
	}

	// The reason why we need to create a match guard for this is because a 
	// standard match statement creates shadowed versions of the named variables
	// in the match statement. So, if I were to use y inside the match statement
	// without the guard I wouldn't be comparing the outer y, I would be using
	// the value of x shadowed into the variable y.
	let y = true;
	match x {
		Some(_) if y => {
			println!("Executes if x is a Some value and y is true");
		}
		_ => prinln!("catchall"),
	}
}
```

### And and Or patterns
These are the same as the | and && operators in cpp.
```rust
fn main() {
	let x = 5;
	let y = false;
	match x {
		// Note that the way that this match guard works is as if we had the or
		// patterns in a set of brackets. 
		// (4 | 5 | 6) if y. So, the if applies to all of the ors not just the 
		// last one.
		4 | 5 | 6 if y => {
			println!("Executes if x is 4, 5 or 5 and y is true");
		}
		_ => println!("Catchall"),
	}
}
```

### Using the @ operator to store values.
Using things like the .. range to create match statements will result in values not being stored. For example:
```rust
fn main() {
	let x = 5;
	match x {
		// Because this statement simply asserts whether x is within its range 
		// or not we can't pass x to the print statement. So, if we want to make 
		// use of x once it has been matched we need to manually store it.
		1..=10 => {
			println!("x is between 1 and 10");
		}
		_ => println!("Catchall"),
	}
	
	match x {
		// By storing the value in my_var we can use the value within the match
		// arm after matching it. 
		x: my_var @ 1..=10 => {
			println!("The value is between 1 and 10 and that value is {}"
			, my_var);
		},
		_ => println!("Catchall"),
	}
}
```

