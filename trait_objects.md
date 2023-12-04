Trait objects are a way of allowing the use of [[traits#Traits as parameters|trait bounds]] but while being even more generic. While a generic type with a trait bound allows for the type to be generic, it is only generic until it is implemented in your code at which point it will only accept that type. A trait object however, allows for completely generic typing. So even after implementation the function/datastructure can still accept any type that has the trait bound.

The reason why generics can't do this is because they directly store/manipulate the memort of the [[generic|generic type]]. Which means they need to know how much data they are expected to take up at compile time. However, trait objects get around this by using [[box|box pointers]] this means that they are of a fixed size as the pointer will always be 4 bytes. 

### Comparison between trait bound generic and trait object.
```rust
// This is an example of a struct which contains a vector of trait objects. 
// These trait objects can be of any type that has the draw trait.
pub struct Screen {
    pub components: Vec<Box<dyn Draw>>,
}

// In comparison this is also generic, but the vector can only contain 1 type 
// that implements the draw trait. As such this will limit the functionality of 
// our screen struct. 
pub struct Screen {
	pub components: Vec<T>,
}

impl<T> Screen<T>
where 
	T: Draw,
{
	pub fn run(&self) {
		for component in self.components.iter() {
			component.draw();
		}
	}
}

// Here I am definig the structs used for the various components.
pub struct Button {
    pub width: u32,
    pub height: u32,
    pub label: String,
}

impl Draw for Button {
    fn draw(&self) {
        // code to actually draw a button
    }
}

struct SelectBox {
    width: u32,
    height: u32,
    options: Vec<String>,
}

impl Draw for SelectBox {
    fn draw(&self) {
        // code to actually draw a select box
    }
}


// Now this main function will only compile with the trait object because the 
// trait object will allow any type that has the draw trait (so long as it is
// held in a pointer).
use gui::{Button, Screen};

fn main() {
    let screen = Screen {
        components: vec![
            Box::new(SelectBox {
                width: 75,
                height: 10,
                options: vec![
                    String::from("Yes"),
                    String::from("Maybe"),
                    String::from("No"),
                ],
            }),
            Box::new(Button {
                width: 50,
                height: 10,
                label: String::from("OK"),
            }),
        ],
    };

    screen.run();
}
```

### Performance
The downside of using trait objects relative to a trait bound generic type is that trait objects require a run time performance hit, whereas trait bound generics do not. The reason for this is that trait bound generics allow for the compiler to compile a separate version of the function for each generic it is used with. While the trait object needs to discern that at run time. 