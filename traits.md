These are used with [[generic|generics]] in order to impose restrictions on just how generic a generic can be. Traits are called in much the same way that methods are. Traits however can only be implemented on a type if at least one of the types or traits involved in the implementation is local to our crate. EG I can't use a type and a trait from an  external library and apply the trait to that type.

### Syntax
```rust
// With this we are declaring a trait, this specifies that the type will need a
// method summarize that uses &self and returns a String. How it does this is 
// of no import, the implementation is defined in the datatype.
pub trait Summary {
	fn summarize(&self) -> String;
}
```

#### Implementation
```rust
pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

// we use the name of our trait and the type/object it is being applied to.
impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}

pub struct Tweet {
    pub username: String,
    pub content: String,
    pub reply: bool,
    pub retweet: bool,
}

// we use the name of our trait and the type/object it is being applied to.
impl Summary for Tweet {
    fn summarize(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}
```

#### Default implementation
```rust
// This saves us from needing to create an implementation for all the types
// we apply this trait to.
pub trait Summary {
    fn summarize(&self) -> String {
        String::from("(Read more...)")
    }
}
```

#### Calling other traits from a trait
```rust
// With this version we only need to create an implementation for the 
// un-implemented summarize_author function.
pub trait Summary {
    fn summarize_author(&self) -> String;

    fn summarize(&self) -> String {
        format!("(Read more from {}...)", self.summarize_author())
    }
}
```

### Traits as parameters
```rust
// As you can see this function is generic so long as the type has an
// implementation of Summary.
// This syntax is best suited for function definitions with multiple params that
// need the same trait but can be different types.
pub fn notify(item: &impl Summary) {
    println!("Breaking news! {}", item.summarize());
}

// This is equivalent to the above but more verbose. This is referred to as a 
// trait bound parameter. 
// This is more appropriate when any type within the function definition that 
// needs to fit the trait binding also needs to be of the same type. 
pub fn notify<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}

// You can also set multiple trait bindings using the following syntax
pub fn notify(item: &(impl Summary + Display)) {
pub fn notify<T: Summary + Display>(item: &T) {

// This is especially useful when you have a large number of bindings and a 
// large number of variables that need bidnings. 
fn some_function<T, U>(t: &T, u: &U) -> i32
where
    T: Display + Clone,
    U: Clone + Debug,
{

// This is a counter example to the above example. 
fn some_function<T: Display + Clone, U: Clone + Debug>(t: &T, u: &U) -> i32 {

```

### Returning trait bound types
```rust 
// However this does not allow us to return using an if else that has 2 types
// that fit the trait bindings. That will be done later with closures. 
fn returns_summarizable() -> impl Summary {
    Tweet {
        username: String::from("horse_ebooks"),
        content: String::from(
            "of course, as you probably already know, people",
        ),
        reply: false,
        retweet: false,
    }
}
```