Boxes simply allow for the ability to store data on the [[stack_and_heap#Heap|heap]] and keep only a reference to that data on the [[stack_and_heap#Stack|stack]]. 

### Use cases
1. When you have a type of unknown size but the context you want to use it in requires fixed size.
2. When you want to be able to transfer ownership of a large amount of data without having to clone it. Instead of copying your super complex class, you can just move the pointer to the other entity.
3. When you want to own a value but the only thing you care about is if it implements a specific trait and not what type it actually is. 


#### Storing data on the heap
```rust
fn main() {
	let x = Box::new(5);
	println!("this is your number {}", x);
}
```

#### Variable size data as fixed data
This requires some context. 
Cons list is a datatype with similar functionality to a linked list. Instead of having (a, b) (b, c) (c, d) (d, e) (e, None) you have (a, (b, (c, (d, (e, None))))). An issue with this datatype is that it requires a Cons struct which contains a Cons struct, which may contain another Cons struct... ... ... As such this type is recursive and the compiler doesn't like that since this object is of infinite size. 

```rust
// We are going to use an enum for this data structure.
enum List {
	Cons(i32, List),
	Nil,
}

enum List {
	Cons(i32, Box<List>),
	Nil,
}

fn main() {
	// This will not compile as is.
	let list = Cons(1, Cons(2, Cons(3, Nil)));
	// This will compile.
	// The reason this will compile is that now instead of the recursive part
	// of this datastructure containing another item of said datastructure that 
	// may or may not contain another... we have short cut this to containing a 
	// Box pointer which no matter how large the contents is the Box pointer will
	// always be of fixed size, as such we have fixed the infinite size issue. 
	let list = Cons(1, Box::new(Cons(2, Box::new(Cons(3, Nil)))));
}
```

### Performance
There is no performance overhead associated with a box pointer, aside from the performance loss from being on the heap. Think the analogy of running around to pick up data on the heap compared to the contiguous nature of the stack. 