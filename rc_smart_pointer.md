Rc (reference counting) pointers in rust are analogous to shared poiners in cpp. Shared pointers allow for multiple pointers to share ownership of a value. The value also doesn't go out of scope until all pointers to the value have gone out of scope. 
Not Rc provides immutable references if we wanted to maintain mutability at some levels we would need to use the [[refcell_smart_pointer|RefCell pointer]] in conjunction with Rc pointers. 

### Use case
Rc pointers are useful when you have multiple parts of your programme need to have the ability to read a piece of heap allocated data, but you don't know which bit will be the last to be in scope. 

### Example
Say we want to create a datastructure that represents the following data:
![Two lists that share ownership of a third list](https://doc.rust-lang.org/book/img/trpl15-03.svg)
If we tried to achieve this using a [[box_smart_pointer|box pointer]] we would get a compilation error because b would take ownership of a, this leaving a invalid. When c then tries to take ownership of a, it is no longer there and as such we get the compile error. 
#### Box pointers
```rust
enum List {
    Cons(i32, Box<List>),
    Nil,
}

use crate::List::{Cons, Nil};

fn main() {
    let a = Cons(5, Box::new(Cons(10, Box::new(Nil))));
    // A moves to b leaving A empty.
    let b = Cons(3, Box::new(a));
    // A is no longer valid, so we get an error. 
    let c = Cons(4, Box::new(a));
}
```

#### Rc pointers
```rust
enum List {
    Cons(i32, Rc<List>),
    Nil,
}

use crate::List::{Cons, Nil};
use std::rc::Rc;

fn main() {
	// Note if we want to share a with others, it itself needs to be an Rc
	// pointer.
    let a = Rc::new(Cons(5, Rc::new(Cons(10, Rc::new(Nil)))));
    // we provide a reference to a to both b and c. A maintains ownership of 
    // itself. 
    // the reference count for a is incremeneted twice (once for b and again for
    // c). This means that a will stay in scope until all of its references go
    // out of scope. 
    // We use Rc::clone instead of a.clone() because Rc::clone doesn't 
    // necessitate a deep copy, unlike a, making our code more efficient. 
    let b = Cons(3, Rc::clone(&a));
    let c = Cons(4, Rc::clone(&a));
}
```

This next section provides logic of how in this context (slightly altered) the reference counting system works. 
```rust
fn main() {
	// Note if we want to share a with others, it itself needs to be an Rc
	// pointer.
    let a = Rc::new(Cons(5, Rc::new(Cons(10, Rc::new(Nil)))));
    println!("count after creating a = {}", Rc::strong_count(&a));
    // 1
    let b = Cons(3, Rc::clone(&a));
    println!("count after creating b = {}", Rc::strong_count(&a));
    // 2
    {
        let c = Cons(4, Rc::clone(&a));
        // 3
        println!("count after creating c = {}", Rc::strong_count(&a));
    } // c has exited its scope so now 2
    println!("count after c goes out of scope = {}", Rc::strong_count(&a));
} // a and b have left scope so now 0; a can be invalidated.
```
