### Syntax
```rust
fn  add_one_v1   (x: u32) -> u32 { x + 1 }
let add_one_v2 = |x: u32| -> u32 { x + 1 };
let add_one_v3 = |x|             { x + 1 };
let add_one_v4 = |x|               x + 1  ;

#[derive(Debug, PartialEq, Copy, Clone)]
enum ShirtColour {
	Red,
	Blue,
}
struct Inventory {
	shirts: Vec<ShirtColour>,
}

impl Inventory {
	fn giveaway(&self, user_preference: Option<ShirtColour>) -> ShirtColour {
		// A closure basically runs a function while "capturing" its 
		// environment. 
		// You can use these in a variable or pass them to other functions. 
		// |*args| code to run
		// These can be one lines or they can be multi-line, multi-line 
		// closures require {} around the code you wish to execute. 
		// In this case, unwrap_or_else either returns the users_preference
		// or the self.most_stocked values supplied via the closure. 
		user_preference.unwrap_or_else(|| self.most_stocked())
	}
	fn most_stocked(&self) -> ShirtColour {
		let mut num_red = 0;
		let mut num_blue = 0;
		for colour in &self.shirts {
			match colour {
				ShirtColour::Red => num_red += 1,
				ShirtColour::Blue => num_blue +=1,
			}
		}
		if num_red > num_blue {
			ShirtColour::Red
		} else {
			ShirtColour::Blue
		}
	}
}
fn main() {
	
}
```


### Compiling
```rust
fn main() {
	// This closure will add 1 to a value provided to it.
	let test_closure = |x| x + 1;
	// I have now provided a value to the close so it knows that this closure
	// expects and returns an integer.
	let x = test_closure(1);
	// I cannot then do the following as the closure now has a defined set of 
	// types
	let y = test_closure(String::from("Hello"));
}
```

### Ownership
Closures can take ownership and borrow both mutably and immutably.
```rust
fn main() {
	let list = vec![1,2,3];
	let only_borrows = || println!("Vector printing {:?}", list);

	println!("I can print list here because this is an immutable closure {:?}"
	, list);

	only_borrows();
	// IF the closure took ownership in this context this print statement would
	// not work.
	println!("The vector still exists {:?}", list);

	// If I adjust the closure so that it adds a value to the list, the closure 
	// will now borrow a mutable reference
	// Note I don't need to pass the list as an argument |list| because a closure
	// exists only in this specific environment so it can use list directly.
	let mut only_mut_borrows = || list.push(4) ;

	// I cannot print the list here because I have passed a mutable reference to
	// the closure, as such that is the only reference that can be used.

	only_mut_borrows();

	// Although I haven't crossed a } yet, the reference used for only_borrows is
	// dead because the reference is not used between the line above and the end
	// of main. As such I can now use immutable references to the list again!
	println!("I can print the list again now {:?}", list);

	// You can force the closure to take ownership of a variable, but this is 
	// mostly useful when creating multi-threaded projects.
	// The ownership is provided to the closure despite only needing an immutable
	// reference to print because of the move keyword. 
	// We want to use ownership here because when one thread will die is unknown.
	// Our main thread may die before the new thread and vice versa, as such it
	// is sager to pass ownership when using a value from one thread in another
	// thread. 
	println!("Before defining the closure {:?}", list);
	thread::spawn(move || println!("From thread {:?}", list))
		.join()
		.unwrap();
	
}
```

### Traits and closures
[Traits](traits.md) can be applied to closures depending on their intended functionality.
1. FnOnce: This applies to closures that move values from their body because the value cannot be moved from its body more than once. 
2. FnMut: This applies to closures that don't move the value but may change it.
3. Fn: This applies to closures that do neither of the above.
Both FnMut and Fn closures can be ran multiple times. However Fn can be ran multiple times concurrently. 

Example s
```rust
impl<T> Option<T> {#
	// unwrap_or_else can only be ran once because you can only unwrap a value
	// once.
    pub fn unwrap_or_else<F>(self, f: F) -> T
    where
        F: FnOnce() -> T
    {
        match self {
            Some(x) => x,
            None => f(),
        }
    }
}

#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let mut list = [
        Rectangle { width: 10, height: 1 },
        Rectangle { width: 3, height: 5 },
        Rectangle { width: 7, height: 12 },
    ];
	// This uses FnMut because it calls the function and changes the values once
	// for each item in the list. 
    list.sort_by_key(|r| r.width);
    println!("{:#?}", list);
}
```