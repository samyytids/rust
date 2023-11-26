Iterators allow you to iterate without creating a for loop. 
Performance wise they are identical to standard for loops, they are just more concise. 

When you create an iterator and store it in a variable you don't actually call the iterator, that needs to be done separately. 
```rust
fn main() {
	let vector = vec![1,2,3];
	// This stores an iterator over the above vector
	// Unlike in the next section this isn't mutable because the for loop takes
	// ownership of the iterator and makes it mutable behind the scenes. 
	let vector_iterator = vector.iter();

	for val in vector_iterator {
		println!("Vector contains: {}", val);
	}
}
```

All iterators implement a train Iterator, this is defined in the [[Contents#Std|std library]] this trait is defined as follows.
```rust
pub trait Iterator {
	// This defines an associated type, which we will talk about later. 
	// All we need to know for now is that this effectively means that in order
	// to define an iterator for an iterable we need to define the type that is
	// returned by next as an associated type. 
	// What next effectively does is when it is called it returns an Option, 
	// which contains an element of the iterable (wrapped in Some) or None once
	// it is finished. 
	type Item;
	fn next(&mut self) -> Option<Self::Item>;
}
#[test]
fn iterator_demonstration() {
	let v1 = vec![1, 2, 3];

	// The reason this needs to be mutable is that once we use the next method
	// we alter the contents of the v1_iter. 
	// Basically the internals need to keep track of where inside the iterator
	// we are at, if this were immuatable this would need to be done externally.
	let mut v1_iter = v1.iter();

	// These all assert to true and highlight the way in which the returning of 
	// the next method works. It returns a Some(wrapped reference) to an element
	// within the iterable or None. 
	assert_eq!(v1_iter.next(), Some(&1));
	assert_eq!(v1_iter.next(), Some(&2));
	assert_eq!(v1_iter.next(), Some(&3));
	assert_eq!(v1_iter.next(), None);
}
```

### Consuming adaptors 
The inbuilt iterator functions from STD can be split into categories, these consuming adaptors call the next method and as such alter the state of the iterator and "consume" the elements within the iterator when they are called. This is why we must define the next trait for our iterators. 
#### Example
```rust
// This is an example of the sum method.
#[test]
fn main() { 
	let v1 = vec![1,2,3];
	// Note this isn't mutable as such we can infer that sum takes ownership and 
	// as such we cannot use this iterator once the sum method has been called. 
	let v1_iterator = v1.iter();
	let total: i32 = v1_iterator.sum();
	assert_eq!(total, 6)
}
```


### Iterator adaptors
These are adaptors that create other iterators. These do not consume there iterator they instead make some change to the initial iterator and return it. These make use of mutable references to the elements of the itterable. 

#### Example
##### Map
```rust
fn main() { 
	let v1 = vec![1,2,3];
	let v1_iterator = v1.iter();
	// Note the use of a closure here. 
	// What this does is iterate through the v1_iterator and + 1 to each element.
	// It then returns the iterator over the new elements. BUT, remember that 
	// iterators do not do anything unless they are called and as such we need to
	// call the .collect() method in order to actually "activate" the iterator 
	// and store its values in a vector. 
	let v2: Vec<_> = v1_iterator.map(|x| x + 1).collect();
}
```

You can  chain numerous iterator adaptors to perform complex logic, but you will need to call a consuming adaptor at some point in order to actually have said iterators do anything. 

##### Filter
```rust
struct Shoe {
	size: u32,
	style: String,
}

fn shoe_in_size(shoes: Vec<Shoe>, shoe_size: u32) -> Vec<Shoe> {
	// Filter returns a boolean value based on the condition within the closure
	// you provide. This boolean decides whether a value should be part of the 
	// new iterator once it is called. .collect() then collects the items based 
	// on the alterations made by the adaptors used on the initial iterator. 
	// Also, never forget that omission of ; means a return. 
	// Into iter creates an iterator that takes ownership of the vector, I assume
	// this is necessary so that we can return the vector without it going out of
	// scope at the end of the function and thus becoming invalid. 
	shoes.into_iter().filter(|s| s.size == shoe_size).collect()
}
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn filters_by_size() {
        let shoes = vec![
            Shoe {
                size: 10,
                style: String::from("sneaker"),
            },
            Shoe {
                size: 13,
                style: String::from("sandal"),
            },
            Shoe {
                size: 10,
                style: String::from("boot"),
            },
        ];

        let in_my_size = shoes_in_size(shoes, 10);

        assert_eq!(
            in_my_size,
            vec![
                Shoe {
                    size: 10,
                    style: String::from("sneaker")
                },
                Shoe {
                    size: 10,
                    style: String::from("boot")
                },
            ]
        );
    }
}

```


### Generic definitions
To make our definitions of functions more generic we can use Iterators with specific associated types rather than specific datastructures containing specific types. 

#### Example
```rust
// Pretend we can implementing a config struct
impl Config {
	pub fn build(
		// What this effecitvely does is instead of saying &[String] which
		// implies a reference to an array of strings we can instead say a we can
		// take anything that has an implementation of the trait Iterator that 
		// has elements of the String type.
		mut args: impl Iterator<Item = String>, 
	) ->  Result<Config, &'static str> {
		// --snip--
	}
}
```

### Ownership
Iterators do not take ownership of the elements they return/iterate through, they simply provide (immutable/mutable) references to these elements. 
Iterators however, can require their ownership to be transferred to some of their methods leaving them invalid after their implementation. [[iterators#Consuming adaptors#Example|See this example]].

