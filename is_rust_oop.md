
#### OOP characteristics

- [x] Objects that contain both behaviour and data. 
	Yes rust has structs!
- [x]  Encapsulation that hides implementation details.
	Yes rust's structs have public and private methods!
- [ ] Inheritance.
	Nope rust doesn't have inheritance, but...

### Rust's pseudo inheritance system.
One of the main uses of inheritance is to be able to use types in place of other types, EG in cpp having an argument of a function be a base class will allow the use of child classes, but you can only call methods that are available in the base class. 

Rust despite not having inheritance can allow this behaviour using the `dyn` keyword which allows you to pass [[trait_objects|trait objects]] as arguments.  These allow for greater flexibility compared to generic types, as they allow for different types to be used within the same function/data rather than requiring the same generic type to be used.

```rust
// In this simplified example of a generic the generic vector can have any type
// in it but once we have input a type into the generic all generics must be the
// same type.
let vector_generic = vec![1,2,3];
let vector_generic = vec!['c','d','e'];

// In this simplified example of a trait object we can see that so long as they
// all implement the required trait and are wrapped in a box pointer we are all 
// good. 
let vector_trait_object = vec!['c', 1, 'd'];
```