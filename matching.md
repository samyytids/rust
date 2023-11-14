Match functions are very similar to switch cases in cpp. 
You have a series of constraints and the value will fall through into the first constraint that results with true. Matches need to be exhaustive. If you don't account for all possible patterns you will hit a compile error.

```rust
enum Coin {
	Penny, 
	Nickel,
	Dime,
	Quarter,
}

// These all effectively return the value associated with each coin you can
// however add further functionality to the match cases. 
fn vaue_in_cents(coin: Coin) -> u8 {
	match coin {
		Coin::Penny => 1,
		Coin::Nickel => 5,
		Coin::Dime => 10,
		Coin::Quarter => 25,
	}
}

fn value_in_cents_extra(coin: Coin) -> u8 {
	match coin {
		Coin::Penny => {
			println!("Lucky penny!");
			1
		}
		Coin::Nickel => {
			println!("Huh, not actually made of nickel");
			5
		}
		Coin::Dime => {
			println!("Wow, what a pretty piece");
			10
		}
		Coin::Quarter => {
			println!("Kinda ironic to call a full circle a quarter.");
			25
		}
	}
}


```

### Matching with nested enums
[[enums#nested enums|See here for information about nested enums]].
The below example is using the fact that quarters used to have different designs depending on which state they were minted/used in. But, none of the other coins received this treatment. 
```rust
#[derive(Debug)] // so we can inspect the state in a minute
enum UsState {
    Alabama,
    Alaska,
    // --snip--
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        // This effecively checks for each coin type and if said coin is a
        // quarter it executes code that varies depending on the state value
        // stored within the coin's state value.
        Coin::Quarter(state) => {
            println!("State quarter from {:?}!", state);
            25
        }
    }
}

```

### Matching with optional types
[[option|See here for information about optional types]].
We can use optional types within match blocks in order to have specific functionality dependent on whether the optional variable contains a value or not. 
```rust
fn plus_one(x: Option<i8>) -> Option<i8> {
	match x {
		None => None,
		// in this case i stands for a sort of wildcard whereby it will bind
		// to whatever specific value has been fed to the function.
		// If I wanted hardcoded behaviour based on whether the supplied value
		// was equal to some value I would need to use magic numbers.----------------------------
		Some(i) => Some(i+1),
	}
}

fn main() {
	let five = Some(5);
	let six = plus_one(five);
	let none = plus_one(None);
}
```

### Catch-all patterns
We have two options when wanting to have a pattern that catches all possible remaining values (I say remaining because catch alls should be used after the specific arms, otherwise the specific arms after the catch all are redundant).

#### catch-all with value
```rust
let dice_roll = 9;
match dice_roll {
	3 => add_fancy_hat(),
	7 => remove_fancy_hat(),
	// because we have used a named variable as our catch-all we pass the value
	// supplied to the match block to the function after the =>
	other => move_player(other),
}

fn add_fancy_hat {/*foo*/}
fn remove_fancy_hat {/*bar*/}
fn move_player(num_spaces: u8) {/*foobar*/}
```

#### catch-all with no value
```rust
let dice_roll = 9;
match dice_roll {
	3 => add_fancy_hat(),
	7 => remove_fancy_hat(),
	// because we have used an _ instead of a named variable we have a catch-all
	// that does not make use of the value supplied to the match block.
	__ => reroll(),
}

fn add_fancy_hat {/*foo*/}
fn remove_fancy_hat {/*bar*/}
fn reroll() {/*foobar*/}
```

#### match case with no implementation
If you want nothing to happen if a specific pattern occurs you can do the following
```rust
// --snip--
match dice_roll {
	// --snip--
	// This says do nothing if not picked up by one of the above arms.
	_ => (),
}
```