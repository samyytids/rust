References are a type of [[generic]] instead of being concerned with what type a variable is they are concerned with how long the variable is alive. 
The main purpose of annotating lifetimes is to prevent dangling references. 
Lifetime annotations on parameters are referred to as input lifetimes and those on return values are output lifetimes. 
```rust
// Dangling reference example
fn main() {
	// Rust still has no null values. 
	// This variable will cause a compile error if we try to use it before 
	// we give it a value. 
	let rp;
	{
		let r = String::from("This is a thing");
		rp = &r
	}
	println!("This is a dangling reference {}", rp);
}
```

### How does Rust determine lifetimes
#### The borrow checker
```rust
// Using the visual representation of lifetimes is great, this summarizes what 
// the rust compiler does. The compiler checks whether the lifetimes of 'a and 
// 'b match up in a similar way. If not, compile error if so compile good. 
// The rust book uses the term "larger" to determine if a lifetime will cause a
// dangling reference or not, I think I prefer overlapping the end. I feel a more
// accurate description is that a variables lifetime must overlap the ending of
// a reference to that variable's lifetime in order to not create a dangling 
// reference. As a lifetime could be significantly larger but end 1 line earlier
// than the reference's final call and that would still cause a danlging 
// reference. 
fn main() {
	let r;                                           //---------+---   'a
	{                                                //         |
		let x = String::from("I am short lived");    //--+-- 'b |
		r = &x;                                      //  |      |
	}                                                //--+      |
	println!("I am a dangling reference {}", r);     //         |
}                                                    //---------+
```

### The rules for implicit lifetime annotation

The compile will use these rules to automatically provide lifetime annotations without the programmer (me 'n' you) from having to manually specify them.
#### Rule 1:
The compiler first assumes that all inputs require a lifetime annotation and each input needs a unique lifetime.

#### Rule 2:
If there is 1 input lifetime parameter then the output has the same lifetime.

#### Rule 3:
If there is a &self or &mut self the output parameter's lifetime is bound to &self as if the value associated with self goes out of scope we can guarantee the value referenced by the return should also go out of scope. 

### Lifetimes as parameters
As is the following function won't compile because we need to add a generic lifetime parameter.
```rust
// Won't compile.
fn longest_no_lifetime(s1: &str, s2: &str) -> &str {
	if s1.len() > s2.len() {
		s1
	} else {
		s2
	}
}

// The reason we need to provide these lifetime parameters is because the 
// compiler doesn't know how the lifetimes of the inputs and the outputs are
// related to one another. 
fn longest_lifetime<'a>(s1: &'a str, s2: &'a str) -> &'a str {
	// --snip--
}
```

#### What do these parameters do?
It doesn't have any impact on how long the parameters live, they simply provide a description of the relative lifetimes of each parameter. Generic lifetime parameters are equivalent to generic type declarations, they allow for references of any lifetime to be fed to the function. Individual lifetime annotation has very little meaning. 
The above generic basically says that all parameters provided to the function have an expected lifetime of at least 'a. What this boils down to is the return value of the function has a lifespan bound by the shortest lifespanned parameter.  
```rust
// --snip--
// Successful example
fn main() {
	let string1 = String::from("Hello");
	{
		let string2 = String::from(", world");
		let result = longest(&string1, &string2);
		println!("The longest string is: {}", result);
	} //String2 and result go out of scope here.
} // String1 goes out of scope here. 

// Failed example.
// If I don't set the lifetime generic in the function definition, this code 
// would pass despite the obvious error (hence why the compiler won't let you
// create this function without using lifetimes).
fn main() {
	let string1 = String::from("Hello");
	let result;
	{
		let string2 = String::from(", world");
		result = longest(&string1[..], &string2[..]);
	} // String2 goes out of scope here, as such takes the val in result with it
	print("The longest string is: {}", result); // uh-oh result is invalid ^
} // String1 goes out of scope here. 
```

#### When to provide lifetime annotations
Lifetime should be provided for references that are passed to functions and are actually potentially returned by the function (at least that's what I understand so far).
```rust
// We don't return y at any point so y doesn't need to have its lifetime examined
fn longest<'a>(x: &'a str, y: &str) -> &'a str {
	x
}

// This won't compile because we are passing a reference to a value created 
// in this function. As such, the actual value will go out of scope.
// As such our annotation is theoretically correct but the function is just dead.
fn longest<'a>(x: &str, y: &str) -> &'a str {
	let result = String::from("really long string");
	&result[..]
}
```

#### Structs with lifetimed parameters
Stucts can contain references to other data as attributes. But, they will require lifetime annotation for every one of these attributes.
```rust
struct ImportantExcerpt<'a> {
	part: &'a str,
}

fn main() {
	let novel = String::from("This is a novel. It is very good.");
	let first_sentence = novel.split(".").next().expect("No full stop (.)");
	let i = ImportantExcerpt {
		part: first_sentence,
	};
}
```

#### Methods with lifetime parameters
We won't often have to deal with lifetimes in the context of methods because 9/10 they will be limited by [[lifetimes#Rule 3|Rule 3: Self limiting]]. 
```rust
// Rule 1, no need to annotate (1 parameter ergo same as the 1 input)
impl <'a> ImportantExcept<'a> {
	fn level(&self) -> i32 {
		3
	}
}

// Rule 2 says self and str get their own lifetimes, Rule 3 says the return gets
// self's lifetime.
imple <'a> ImportantExcerpt<'a< {
	fn name(&self, s: &str) -> &str {
		println!("Attention please, {}", s);
		self.part
	}
}
```

### Static lifetime: a special case
This is a lifetime that specifies that the reference will last for the entire duration of the programme. String literals come with this lifetime by default. 
```rust
// string literals are permanent because they are hardcoded into the binary 
// of our executable.
let s: &'static str = "I have a static lifetime.";
```