#### Placeholder types with associated types
We have seen these before in examples from previous sections. The syntax below should seem familiar. 
```rust
pub trait Iterator {
	type Item;
	fn next(&mut self) -> Option<Self::Item>;
}
```

We can think of an associated type as the Remoraid under a Mantine's wing, they are separate things but they are inherently attached. So, in the case of an iterator trait the generic Item type effectively converts to being the type contained within a given datatype. Below is an implementation of the next function in this trait.
```rust
impl Iterator for Counter {
	type Item = u32;

	fn next(&mut self) -> Option<Self::Item> {
		// Here we put in the implementation of the iterator for this class.
	}
}
```

##### Why not just use a generic type?
If we instead used the below definition for the Iterator trait:
```rust
pub trait Iterator<T> {
    fn next(&mut self) -> Option<T>;
}
```

This definition looks more flexible because it lets us use any type for our iterator, while the associated type restricts us to a single concrete type. But, say our Counter could need to iterate over multiple different types. We may need to implement multiple different next methods depending on what we would need to do with the data. If we used generics to do that we would need to provide type annotation every time that we made use of the next method, but with an associated type then we would have this handled by the compiler. 

Associated types also become part of the contract of the trait, meaning that the implementor of the trait needs to provide information regarding the type of values the trait's methods are going to be used with. 

### Default generic type parameters and operator overloading
Supplying default types to generic types prevents the need for providing type annotation when using said default type.
```rust
// This is the definition of the Add trait, the add trait defines the behaviour
// when the + operator is used. 
// Add uses the default generic type of Rhs = Self. Meaning that by default
// the + operator will assume that it will work with 2 instance of the same type.
// So, it will add together Point and Point types to return a Point type in this
// case. 
trait Add<Rhs=Self> {
    type Output;

    fn add(self, rhs: Rhs) -> Self::Output;
}

use std::ops::Add;

#[derive(Debug, Copy, Clone, PartialEq)]
struct Point {
    x: i32,
    y: i32,
}

// Here we are specifying using the associated type output that the + operator
// should return a Point type.
// and since we are not setting a generic type manually in the add trait we are
// allowing add to use the defualt of self + self = associated type.
impl Add for Point {
    type Output = Point;

    fn add(self, other: Point) -> Point {
        Point {
            x: self.x + other.x,
            y: self.y + other.y,
        }
    }
}

fn main() {
    assert_eq!(
        Point { x: 1, y: 0 } + Point { x: 2, y: 3 },
        Point { x: 3, y: 3 }
    );
}

// If we instead provide a type to the impl Add for Point we can add more 
// specific behaviour. 
use std::ops::Add;

struct Millimeters(u32);
struct Meters(u32);

// Here we have specified that this is what we want to happen when we are adding
// meters and millimeters together. This implementation does the conversion from 
// meters to millimeters for you.
impl Add<Meters> for Millimeters {
    type Output = Millimeters;

    fn add(self, other: Meters) -> Millimeters {
        Millimeters(self.0 + (other.0 * 1000))
    }
}
```

#### Why would I do this though?
One of the main reasons to do this, is to allow users to make custom behaviour that in most cases you won't need. For example, 99% of the time I am just going to want to add two like types but there may be scenarios where the user may want to be able to make use of x + y = x. This allows the majority of users to not need to include type annotation into their code but it allows those with specific usecases to easily expand the functionality of the Add operator. 

The other side of this coin is that it allows you to keep x + x = x working while allowing new behaviour. If I didn't use a default generic here I may make a change to my addition operator that breaks the implementation of the default operator. 

### Calling methods with the same name. 
We may have some ambiguity in our code where one type has a method and another type has a method with the same name. How do I make sure that the right one is called?

```rust
trait Pilot {
	fn fly(&self);
}

trait Wizard {
	fn fly(&self);
}

struct Human;

impl Pilot for Human {
	fn fly(&self) {
		println!("Nyeeeeeom");
	}
}

impl Wizard for Human {
	fn fly(&self) {
		println!("Shhhhyom");
	}
}

impl Human {
	fn fly(&self) {
		println!("I CAN'T FLY");
	}
}
```

When I try to call the .fly method on my Human struct by default it will use the human.fly() implementation. But, if my human is in fact a Wizard human I want the Human fly to occur.
To call the wizard version I would need to do the following.

```rust
fn main() {
	let person = Human;
	Wizard::fly(&person);
}
```

### Calling associated functions with the same name.
The above only works when we are using methods (&self) if we don't have a self component to the function then we can't use the above methodology.

```rust
trait Animal {
	fn baby_name() -> String;
}

struct Dog;

impl Dog {
	fn baby_name() -> String {
		String::from("Spot");
	}
}

impl Animal for Dog {
	fn baby_name() -> String {
		String::from("Puppy");
	}
}

fn main() {
	// This will default to the Dog implementation of the associated function 
	// baby_name and as such will say that all baby dogs are called Spot instead
	// of Puppy.
	println!("A baby dog is called a {}", Dog::baby_name());

	// So to call an associated function from a trait instead of the main 
	// definition for that struct we use the following syntax.
	// Dog as Animal basically says to process this function call as an animal 
	// instead of using its main type of Dog. 
	println!("A baby dog is called a {}", <Dog as Animal>::baby_name());
}
```

We can however use the same syntax for both [[advanced_traits#Calling methods with the same name.|calling methods]]  and [[advanced_traits#Calling associated functions with the same name.|calling functions]] if we use the following:
```rust
fn main() {
	<Type as Trait>::function_name(&self/*if a method*/, args, kwargs);
}
```

### Using super traits
Super traits are traits that have other traits that rely on them in some way. For example if one of my traits requires that a type implements another trait this other trait is the supertrait in this scenario. 
The example we are going to use here is an OutlinePrint trait which prints things with a fancy out line but for this trait to work we need to have an implementation of the Display trait. Logically we can only print something with an outline if it can be printed to begin with. 

```rust
use std::fmt;

trait OutlinePrint: fmt::Display {
	fn outline_print(&self) {
		let output = self.to_string();
		let len = output.len();
        println!("{}", "*".repeat(len + 4));
        println!("*{}*", " ".repeat(len + 2));
        println!("* {} *", output);
        println!("*{}*", " ".repeat(len + 2));
        println!("{}", "*".repeat(len + 4));
	}


/*
************
* thing    *
* we are   *
* printing *
************
*/

struct Point {
    x: i32,
    y: i32,
}

// If we try to do this we will get a compilation error because the Point struct
// hasn't implemented the Display trat. 
impl OutlinePrint for Point {}

// So we would need to do the following in order to get the OutlinePrint trait
// the be implementable on our Point struct. 
impl fmt::Display for Point {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "({}, {})", self.x, self.y)
    }
}
```

### Using the new type pattern to implement external traits on external types.
As was (I believe) mentioned in [[traits]] we can only implement external traits on internally generated types. IE If I created a type I can use a trait from a use statement, and I can use an internally generated trait on a type from a use statement. 
If I want to use an external trait on an external type I will need to create a wrapper which now creates a "new type". 

```rust
use std::fmt;

struct Wrapper(Vec<String>);

// I didn't make the Display type nor did I create the Vec type as such I 
// wouldn't be able to apply the Display trait to the Vec type. But since I have
// created the Wrapper struct which is just a thin vaneer over the Vec type I can
// apply the Display trait to that. 
impl fmt::Display for Wrapper {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "[{}]", self.0.join(", "))
    }
}

fn main() {
    let w = Wrapper(vec![String::from("hello"), String::from("world")]);
    println!("w = {}", w);
}
```

This isn't a completely free lunch though because by using the wrapper type I lose direct access to the type it is wrapping. IE my Wrapper struct doesn't have an implementation to the .push method that the vector has. I could get around this however by creating a custom implementation of the deref trait which would unwrap the wrapper to allow access to the vector. 

```rust
fn main() {
	let w = Wrapper(vec![String::from("Hello"), String::form("World")]);
	// If I specify that the deref trait returns a reference to the self.0 of the
	// wrapper I can now "directly" access the methods of the vector within the 
	// Wrapper tuple struct. 
	&w.push(String::from("and you... I guess"));
}
```

