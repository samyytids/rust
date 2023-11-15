Panics are unrecoverable errors, as such these guarantee a crash.
These can either be caused by rust. EG accessing an element outside of an array, or they can be manually called by the users code. 
By default panicking manually unwinds and cleans up memory, but you can change this behaviour and push those responsibilities onto the operating system.
```rust
[profile.release]
panic = 'abort'
```

### Calling panic
```rust
fn main() {
	// panic is a macro included in the prelude.
	panic!("Force crash");
}
```

