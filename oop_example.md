Here we are creating an example of the state design pattern whereby the action of an object is impacted by the state it is in. With a [[is_rust_oop#Rust's pseudo inheritance system.|lack of inheritance]] in rust we need to instead make use of the [[traits|trait]] system.

### Example structure
1. A post starts off as an empty draft. 
2. It can then be given text and set to needs to be reviewed.
3. The post can then be reviewed and published.
4. When queried for content we should only receive a non-empty string from a completed post. 

```rust
// Pretend this really exists.
use blog::Post;

fn main() {
	let mut post = Post::new();

	post.add_text("My wife is lovely!");
	// Post hasn't even been set to be reviewed yet, so shouldn't be able to get
	// the content from the post yet.
	assert_eq!("", post.content());

	post.request_review();
	// Although we have asked for a review we haven't been updated yet so still
	// no content.
	assert_eq!("", post.content());

	post.approve();
	// The post has now been approved so it should return the content now.
	assert_eq!("My wife is lovely!", post.content());
}
```


### The actual code!
#### Defining some of our structs.
```rust
struct Post {
	// Instantiating a state object, remember if we just used a trait bound 
	// generic the type within here would have to be static. IE we could only
	// ever have a draft post.
	state: Option<Box<dyn State>>,
	content: String,
}

impl Post {
	pub fn new() -> Post {
		Post {
			state: Some(Box::new(Draft {})),
			content: String::new(),
		}
	}
}

// This is the trait State, but we have provided it with a default implementation
// that does nothing.
trait State {}

// This is currently a blank draft struct.
struct Draft {}

impl State for Draft {}
```

#### Adding text.
```rust
imple Post {
	// mutable reference to self so that the function can make changes to the 
	// struct. 
	// This doesn't have any need to consider state. 
	fn add_text(&mut self, text: &str) {
		self.content.push_str(text);
	}
}
```

#### Ensuring the draft has no content. 
```rust
impl Post {
	// We are providing a base implementation of the getter for the content
	// attribute, which returns a blank string. 
	pub fn content(&self) &str {
		""
	}
}
```

#### Requesting a review.

```rust
impl Post {
	// mutable reference again since we are changing the post. 
	pub fn request_review(&mut self) {
		// Think of this as a single state match statement. We are checking if 
		// the Post instance has a value in state and if so we are setting its
		// value to the response gained from calling its request_review() method.
		// .take() takes the value out of a Some wrapper and leaves a None in its
		// place. 
		if let Some(s) = self.state.take() {
			self.state = Some(s.request_review())
		}
	}
}

trait State {
	// this is creating a State trait which has the request_review function that
	// has no current implementation but defines the input and output of the 
	// function. 
	fn request_review(self: Box<self>) -> Box<dyn State>;
}

// Once again this is a blank draft.
// Blank because all we need it for is the type that it embodies. 
struct Draft {}

impl State for Draft {
	// When we call request review on a Draft post we need it to be moved to a 
	// PendingReview post. Hence why we are returning a Box::new(PendingReview).
	// What this does is when we call request_review on a Post, we take the
	// contents of its state variable. When we call request_review on it and we
	// have a Draft in the state this is called, upgrading our post to the new 
	// state. 
	fn request_review(self: Box<self>) -> Box<dyn State> {
		Box::new(PendingReview {})
	}
}

// Blank for the same reasoning as the Draft struct.
struct PendingReview {}

impl State for PendingReview {
	// Meanwhile in this instance if we call request_review on a Post that's 
	// state is already pending review then we don't need to change the state
	// so we can put this exact state back in the Post's state variable. 
	fn request_review(self: Box<self>) -> Box<dyn State> {
		self
	}
}
```

#### Implementing approval. 

```rust
impl Post {
	pub fn approve(&mut self) {
		if let Some(s) = self.state.take() {
			self.state = Some(s.approve())
		}
	}
}

trait State {
	// No default implementation.
	fn approve(self: Box<self>) -> Box<dyn State>;
}

impl State for PendingReview {
	fn approve(self: Box<self>) -> Box<dyn State> {
		Box::new(Published {})
	}
}

// Once again only necessary for the state.
struct Published {}

impl State for Published {
	// This has already been published, so calling request_review on it should
	// effectively do nothing.
	fn request_review(self: Box<self>) -> Box<dyn State> {
		self
	}
	// Same deal.
	fn approve(self: Box<self>) -> Box<dyn State> {
		self
	}
}
```

#### Making the content work properly.

```rust
trait State {
	// We need to add the lifetime here because the content only lasts as long 
	// as the post that contains it does. 
	// We are also providing a default implementation because this saves us from
	// writing the same implementation of returning an empty string for both 
	// draft and under review posts. 
	// We take a reference to the post that the state is in because obviously
	// the state doesn't containt the content.
	// We also pass a reference to the post because if we took ownership the post
	// would go out of scope as soon as the content function executed. 
	fn content<'a> (&self, post: &'a Post) -> &'a str {
		""
	}
}

impl State for Published {
	// Only the published state needs to return the content of the post. 
	fn content<'a> (&self, post: &'a Post) -> &'a str {
		&post.content
	}
}
```

### Why not just use an enum?
Perfectly valid option, but the downside to using an enum is that the areas of your code that access the post need to have a way to determine the state based on the enum, whereas this way the post can handle that logic internally leading to less code reproduction.

### Trade offs with this approach.
Easy to extend the functionality of a post, for example you only need to add a new state and an implementation for its State trait's functions in order to provide this functionality to all scenarios that the Post struct is used. EG you can easily add a reject method which takes a pending review to a draft. 

However, this means that we have coupling, draft -> review -> published. If we wanted to add an intermediary we would need to adjust both review and published to accommodate the new state. 

We also have no guarantee that we have exhaustively handled all scenarios until we hit run time bugs. 

### A rustier alternative


Instead of having a struct with different states we use a series of extra structs that contain the minimum functionality that they need, the goal of this is to make it impossible to have scenarios where we use the wrong method on a struct. 

#### Post and DraftPost
```rust
// all our structs contain content as this ensures that we can access that 
// information but the state is handled entirely by what struct is within a 
// particular variable. 
pub struct Post {
    content: String,
}

pub struct DraftPost {
    content: String,
}

// Post by default creates a draft post, so we can only start with a draft post.
impl Post {
    pub fn new() -> DraftPost {
        DraftPost {
            content: String::new(),
        }
    }

	// only the end post type can return its content.
    pub fn content(&self) -> &str {
        &self.content
    }
}

// DraftPost doesn't have a constructor as it is created by a DraftPost. 
impl DraftPost {
	// Draft posts can only add text.
	// If we write a piece of code that tries to accecss the content method of a 
	// DraftPost we will get a compile error. 
    pub fn add_text(&mut self, text: &str) {
        self.content.push_str(text);
    }
}
```

#### Implementing transitions
```rust
impl DraftPost {
    // --snip--
    // The draft post needs a way to move from a draft to a pending review. 
    pub fn request_review(self) -> PendingReviewPost {
        PendingReviewPost {
            content: self.content,
        }
    }
}

// Once again no reason to keep track of state. 
pub struct PendingReviewPost {
    content: String,
}

impl PendingReviewPost {
	// Pending review posts have no reason to adjust their content, they just 
	// need to be able to move from a pending post to a Post. 
	// Once again if our code tries to use the functionality of another type
	// of post we will get a compile error. 
    pub fn approve(self) -> Post {
        Post {
            content: self.content,
        }
    }
}
```

#### Stringing it all together
```rust
use blog::Post;

fn main() {
	// We create a mutable post because we need to be able to add text to a Post.
    let mut post = Post::new();

    post.add_text("I ate a salad for lunch today");

	// Immutable because we simply shadow the definition rather than adjusting 
	// the post itself. 
    let post = post.request_review();

	// Same deal.
    let post = post.approve();

    assert_eq!("I ate a salad for lunch today", post.content());
}
```

#### Main positives
The main positive here is that it is impossible to compile this code with any states that aren't valid. 