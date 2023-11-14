### Loop
This is a specific implementation of while True.
These loops will iterate until they are broken. 
```rust
fn main() {
	let mut x = 0;
	loop {
		println!("Loop iteration: {x}");
		x += 1;
		if x > 100 {
			// You can add functionality to the break line
			// break x * 1000;
			break;
		}
	}
}
```

You can provide names to loops in order to keep track of which loop we are specifically talking about. 
```rust
fn main() {
    let mut count = 0;
    // 'name_of_loop gives the loop an alias so we can call breaks on specific
    // loops.
    'counting_up: loop {
        println!("count = {count}");
        let mut remaining = 10;

        loop {
            println!("remaining = {remaining}");
            if remaining == 9 {
                break;
            }
            if count == 2 {
	            // This means that when count reaches 2 we specifically break
	            // the counting up loop rather than the loop we are actually
	            // currently in.
	            // This basically means once we do break from this loop we won't 
	            // need to return to the start of the counting up loop.
                break 'counting_up;
            }
            remaining -= 1;
        }

        count += 1;
    }
    println!("End count = {count}");
}
```

### While loops
#### Syntax
Once again this is a halfway house between python and cpp
```rust
fn main() {
	let mut number = 3;
	while number < 10 {
		println!("{number}");
		number += 1;
	}
	println!("Done");
}
```

### For loops
#### Syntax
I bet you can't guess what I am about to say

This is a for loop for iterating through a collection.
```rust
fn main() {
	let a = [1,2,3,4,5];
	for element in a {
		println!("{element}");
	}
}
```

This is a for loop for iterating through a range of values.
```rust
fn main() {
	for number in (1..4).rev() {
		println!("{number}");
	}
	println!("BOOM!");
}
```
