The use keyword allows you to have access to functions and objects from other modules without having to continuously use the long absolute/relative path to get you to each individual object or function you make use of. 
```rust
mod module {
	pub mod sub_module {
		pub fn function() {}
	}
}

use crate::module::sub_module;

fn main() {
	sub_module::function();
}
```

### Using modules outside of modules
Use only grants usage to functions to the module that the use statement is used within.
```rust
mod module {
    pub mod sub_module {
        pub fn function() {}
    }
}

// This use statement is outside of the module that actually makes use of the 
// module that was "imported".
// To fix this I could either remove the module wrapping below, or I could
// move the use statement inside of that module.
use crate::module::sub_module;

mod other_module {
    pub fn eat_at_restaurant() {
        sub_module::function();
    }
}
```

### As
import pandas as pd.
```rust
use std::io::Result as IoResult;
```
This also addresses the problem mentioned in [[use#^24da0f|"best practice"]], now we can distinguish between potential naming conflicts as we have provided a custom name. 

### Using external packages
Think of this as being like pip install scrapy and then using import scrapy.
But instead you whack scrapy in your cargo.toml and then use scrapy.

### Nested paths
Think of this as being similar to from x.y import a,b,c.
```rust
use std::cmp::Ordering;
use std::io;
// or
use std::{cmp::Ordering, io};

// You can also use self in order to import the base module and some others.
use std::io;
use std::io::Write;
// or
use std::io::{self, Write};
```

### Glob operator
from db.models import *
```rust
// this imports all the modules from within std::collections;
use std::collections::*;
```
### Black magic
By using pub use, you effectively bring the module you use into your module in such a way that other modules can now call the module from this module instead (I am explaining this terribly let's look at a code example).
```rust
// nomrally to call this module from somewhere below a module from outside of
// this module would have to call
// this_module::module::sub_module;
mod module {
    pub mod sub_module {
        pub fn function() {}
    }
}

// after calling this, modules outside of this module can now call
// this_module::sub-module;
// The main use for this is kind of obscure for me now and would be more 
// impactful in real world code where numerous people need to interact with the
// code. The idea being that is basically allows multiple different 
// ways of accessing your code, which may be more understandable for different
// teams that have different access to the code base?
pub use module::sub_module;



```

### Best practice

^24da0f

Much like in cpp it is best to keep your namespaces 1 level lower than using functions from other modules directly. This prevents naming conflicts.
EG. If I import the function test_function and my current module also has a function called test_function I will have a naming conflict. 
I should instead use the module that contains the function I want to use as I will then need to call module::test_function() which will prevent the name conflict issue.

