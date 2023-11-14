if let allows you to use the functionality of a match statement while only have to use 1 case.

### Syntax
```rust
fn main() {
	let config_max = some(3u8);
	// If the pattern matches then we execute the code in the if let block
	// otherwise nothing happens.
	if let Some(max) = config_max {
		println!("The maximum is configured to be {}", max);
	}
	// You can include an else.
	else {
		println!("The maximum has not been configured");
	}
}
```
