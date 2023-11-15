### Compiling a project step by step

#### Root file
The compiler will look for the root file. 
- main.rs for a binary crate
- library.rs for a library crate. 

#### Checking for modules
In your root rile you should declare modules. using the syntax:
```rust
mod module_name;
mod module_name {
	//foo bar
}
```
With the initial syntax rust's compiler will check the following locations in order:
1. ./module_name.rs
2. ./module_name/mod.rs
Where the base directory is the directory of your root file.

#### Checking for sub-modules
In your modules you should declare sub-modules. using the syntax:
```rust
mod sub_module_name;
mod sub_module_name {
	//foo bar
}
```
With the initial syntax rust's compiler will check the following locations in order:
1. ./sub_module_name.rs
2. ./module_name/sub_module_name.rs
Where the base directory is the directory of your module file.

#### Accessing custom modules
Once your modules are part of your crate they can be accessed using this syntax:
```rust
crate::module::sub_module::sub_sub_module...;
self::module::sub_module::...;
super::module::sub_module::...;
```
you can also use the super keyword in order to use a relative import starting from the parent folder and the self keyword to start from the current folder. 
#### Private vs public code
Modules are by default created as private with respect to their parent modules.
If you want to make a module public declare it using this syntax:
```rust
pub mod module_name;
```
To make items within the module itself public as well they must also be declared as public using the pub keyword.

#### Shorthand via use
Much like in cpp you can use use to reduce having to write the entire module path everytime you use a function from a module.

#### Best practices
Rust best practices uses the library.rs to manage module trees and then the main.rs makes use of that code via use statements and only has access to the public aspects of the code. 

#### Public and private declarations of structs and enums
Rust allows for objects inside of public modules to be private (this is the default). When making an object public, it's members are still private by default so the members of each class must be manually assigned to be public. 
However, enums are either fully private or fully public.
```rust
pub mod module {
	// in its current form this struct would be uninstantiable by a parent
	// module. Because we would be unable to set a value for the private
	// member. To get aronud this we would need to provide a public function that
	// can be used to instantiate a Struct object.
	pub struct Struct {
		private_struct_member: u8,
		pub public_struct_member: u8,
	}
}
```