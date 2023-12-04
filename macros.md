Macros are basically code that write code. By creating a macro (depending on the kind of macro) you can write code that implements features for you. Think \#\[derive(debug)] we have used before. This macro implements the debug trait on a class for you. 
There are 3 different types of macros:
1. Custom \#derive macros.
2. Attribute like macros
3. Function like macros

### Declarative macros
These are the most common type of macros `marco_rules!` macros. An example of this is the vec! macro. The benefit of using a macro rather than an initializer in this case is that because macros are dynamic they can effectively take any number of arguments. 
```rust
// Note this implementation is simplified because vec! actually has some inbuilt
// memory optimization that we are not using here. 

// macro_export says that whenever this crate is brought into scope the macro 
// should also be brought into scope. 
#[macro_export]
macro_rules! vec {
	// What this basically boils down to is a regex style pattern.
	// & says that we are expecting a variable declaration in the macro. 
	// &x:expr says we are looking for any rust expression and we then give that
	// expression the name x. 
	// , after the $() says we are looking for a literall comma separator and *
	// says that we are looking for 0 or more of this pattern. 
	// To use a python analogy.
	// test_list = list("1,2,3,4,5".split(","))
	// We are looking for some kind of expression, split by commas and we are 
	// expecting any number of this expression.
	// The code after the => is the "code" that we execute. 
	// In this case we are creating a vector than then for every $x  we push back
	// a value into that vector.
	( $( $x:expr ), *) => {
		{
			let mut temp_vec = Vec::new();
			$(
				temp_vec.push($x);
			)
			temp_vec
		}
	};
}
```

### Procedural macros for generating code from attributes
While declarative macros take code, match it to a pattern and then write more code based on that pattern, procedural macros take code operate on that code and produce some code as an output. 
Procedural macros as they are now need to be stored in a separate crate.

```rust
use proc_macro;

#[some_attribute]
pub fn some_name(input: TokenStream) -> TokenStream {

}
```

#### Creating a custom derive macro
For this example we are going to create a custom derive macro that prints the name of a data type when it is called. These only work for [[structs]] and [[enums]].

```rust
use hello_macro::HelloMacro;
use hello_macro_derive::HelloMacro;

#[derive(HelloMacro)]
struct Pancakes;

// As is this would require me to write an implementation for this trait for 
// every single type that I want to use it for which would be incredibly tedious.
impl HelloMacro for Pancakes {
    fn hello_macro() {
        println!("Hello, Macro! My name is Pancakes!");
    }
}

fn main() {
	// The intention here is to have this print the datatypes name.
    Pancakes::hello_macro();
}
```

When we want to create a procedural macro we need to create it in its own separate crate. 
```shell
$ cargo new hello_macro_derive --lib
```

Once this separate crate has been created we need to add some dependencies and settings in the cargo.toml file.
```toml
[lib]
proc-macro = true

[dependencies]
syn = "1.0"
quote = "1.0"
```

We can now get to the meat and potatoes of creating the custom derive macro. 
A common practice is to separate the macro into 2 parts. The derive function which executes the implementation function. Almost all proc_macros that you see will be defined this way. 
```rust
// This won't compule because we have not defined the impl_hello_macro function
// yet.

use proc_macro::TokenStream;
// This then turns the syn parsed datastructure back into rust code. 
use quote::quote;
// This crate parses rust code into a datastructre that we can query in order for
// the macro to generate procedural code. 
use syn;

// The name specified in here will set the name needed to be called in the 
// #[derive(xyz)] decorator.
// As such this can be applied to a something using #[derive(HelloMacro)]
#[proc_macro_derive(HelloMacro)]
pub fn hello_macro_derive(input: TokenStream) -> TokenStream {
    // Construct a representation of Rust code as a syntax tree
    // that we can manipulate
    let ast = syn::parse(input).unwrap();

    // Build the trait implementation
    impl_hello_macro(&ast)
}
```

If we apply this macro the the Pancakes struct we will get the following datastructure from the syn::parse() associated function.
```rust
DeriveInput {
    // --snip--
	// Identity
    ident: Ident {
        ident: "Pancakes",
        span: #0 bytes(95..103)
    },
	// The actual data
    data: Struct(
        DataStruct {
            struct_token: Struct,
            fields: Unit,
            semi_token: Some(
                Semi
            )
        }
    )
}
```

Now we need to create the actual impl_hello_macro function.
```rust
// We need to pass the syn parsed data structure first.
fn impl_hello_macro(ast: &syn::DeriveInput) -> TokenStream {
	// All we are interested in here is the identity Pancakes.
    let name = &ast.ident;
    // We then create the function.
    let gen = quote! {
	    // # syntax here specifies this is using a variable named name from the 
	    // outer scope. 
        impl HelloMacro for #name {
            fn hello_macro() {
	            // stringify! converts this into a rust String.
                println!("Hello, Macro! My name is {}!", stringify!(#name));
            }
        }
    };
    // We can think of this as the export that actually exports the created 
    // implementation. 
    gen.into()
}
```

#### Attribute like macros
These work for more than just structs and enums and instead of generating code for the derive attribute they create their own attributes. 
We are going to create an example that we use to annotate functions that use a web app framework. 
```rust
#[route(GET, "/")]
fn index() {
	// -- snip --
}
```

This macro would be defined as a proc_macro.
So here is our actual definition of the route attribute. 
```rust
#[proc_mac_attribute]
// This creates 2 arguments, the attrs is the GET, "/" and the other is the 
// TokenStream for the actual function the macro is attached to, in this case the
// index function.
pub fn route(attr: TokenSteam, item: TokenStream) -> TokenStream {

}
```

Beyond this they function identically to proc_macros

#### Function like macros
These are used similarly to macro_rules macros but instead of using a match like syntax to generate rust code these take in a TokenStream and execute code on it. This example shows a hypothetical macro that would take in a SQL query and check that it is syntactically correct. 
These are more flexible than standard functions.
```rust
#[proc_macro]
pub fn sql(input: TokenStream) -> TokenStream {
}
let sql = sql!(SELECT * FROM posts WHERE id=1);
```