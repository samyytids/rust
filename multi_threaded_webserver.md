A downside to our [[single_threaded_webserver_example|single threaded webserver]] is that it has to process each request one at a time. So, a very small request will be held back by someone making a very large request. We are now going to add multi-threading to this server in order to allow multiple requests to be handled at once. 

### Simulating a slow request
We are going to simulate a slow request by using thread sleep. 

#### Modules
Most of these come from our previous single threaded webserver, but the thread and time::duration are explained here (concurrency).
```rust
use std::{
	fs,
	io::{prelude::*, BufReader},
	net::{TcpListener, TcpStream},
	thread,
	time::Duration,
};
```

#### Implementing our slow request
After fetching our request from the port local port 7878 and fetching the request line from that response we do the following:
1. Use a match statement on a slice of the entirety of the request line and match it against 3 [[patterns#matching literals|patterns]]. 
	1. Our valid response to the default URI, which triggers a 200 and the [[single_threaded_webserver_example#Actually adding the html content to our response|hello.html]] content.
	2. Our sleep response to the URI /sleep, which does the same but after waiting for 5 seconds.
	3. Our 404 error response which is triggered with a catchall _.

2. We then process the response as normal.
```rust
fn handle_connection(mut stream: TcpStream) {
    // --snip--

    let (status_line, filename) = match &request_line[..] {
        "GET / HTTP/1.1" => ("HTTP/1.1 200 OK", "hello.html"),
        "GET /sleep HTTP/1.1" => {
            thread::sleep(Duration::from_secs(5));
            ("HTTP/1.1 200 OK", "hello.html")
        }
        _ => ("HTTP/1.1 404 NOT FOUND", "404.html"),
    };

    // --snip--
}
```

As it is with our single threaded implementation whenever the slow request is called all other requests must wait for it to execute even though we have the resources available to load many other requests while the sleep request is being processed.

### Using a thread pool to improve performance.
A thread pool is a group of spawned threads that are waiting and ready to receive a request, this way we don't have to wait for the thread to be spawned as and when they are needed. Another benefit to this is that we are protected from DoS attacks (think ddos but without the distributed part), since we aren't dynamically creating threads they can't completely shut down our server by overloading it with 274329387409243872094380927438 requests. 
Each of our threads will have a queue to hold requests that are made before the thread has finished processing, this way we can handle N requests at a time where N is the number of threads. 

Code structuring suggestions, set up the client interface first to guide the design of the rest of the system. AKA write a theoretical API and then create the backend to fit said API rather than creating a backend and trying to get it to fit an API later on. 

#### A non-thread pool implementation

Here we are spawning the thread as and when a stream is received and using a closure to execute the handle_connection function on the stream. 
```rust
fn main() {
	let listener = TcpListener::bind("127.0.0.1:7878").unwrap();

	for stream in listener.incoming() {
		let stream = stream.unwrap();

		thread::spawn(|| {
			handle_connection(stream);
		})
	}
}
```

#### A thread pool implementation

The issue with this code is that we don't have a ThreadPool type, so we are going to need to create one. 
```rust
fn main() {
	let listener = TcpListener::bind("127.0.0.1:7878").unwrap();
	let pool = ThreadPool::new(4);

	for stream in listener.incoming() {
		let stream = stream.unwrap();

		pool.execute(|| {
			handle_connection(stream);
		})
	}
}
```

##### Creating the ThreadPool type using compiler driven development

We'll first create a lib.rs so that we can create this type outside of our main.rs. 
```shell
touch lib.rs
```

We then need to create a public struct called ThreadPool so that it can be brought into our main.rs. As is we have no real implementation for this class so when we impost it with use and run cargo check we will get a new compiler error telling use what we need to add to get the code to compile.
```rust
pub struct ThreadPool;
```

```rust
use hello::ThreadPool;
```

```shell
$ cargo check
    Checking hello v0.1.0 (file:///projects/hello)
error[E0599]: no function or associated item named `new` found for struct `ThreadPool` in the current scope
  --> src/main.rs:12:28
   |
12 |     let pool = ThreadPool::new(4);
   |                            ^^^ function or associated item not found in `ThreadPool`

For more information about this error, try `rustc --explain E0599`.
error: could not compile `hello` due to previous error
```

As we can see from the above we have no implementation for new within our ThreadPool struct so we have to start there first. Let's start with a bare bones initializer. We have chosen usize here because we have no need for negative numbers of threads for obvious reasons. We run cargo check again to see what we are missing. 

```rust
impl ThreadPool {
	pub fn new(size: usize) -> ThreadPool {
		ThreadPool
	}
}
```

```shell
$ cargo check
    Checking hello v0.1.0 (file:///projects/hello)
error[E0599]: no method named `execute` found for struct `ThreadPool` in the current scope
  --> src/main.rs:17:14
   |
17 |         pool.execute(|| {
   |              ^^^^^^^ method not found in `ThreadPool`

For more information about this error, try `rustc --explain E0599`.
error: could not compile `hello` due to previous error
```

This now tells us that we haven't implemented the method execute for our ThreadPool class. We need to decide what it is that we are going to do to get this to work, we know that we are using a similar implementation as the thread::spawn function so we can use that as inspiration. Checking the source code for thread::spawn we see that the closure that they take has the FnOnce trait and is Send with a 'static lifetime. FnOnce makes sense because it needs to capture the information from execute and each request sent to a thread will only need to be executed once. Send makes sense because we are using a multi-threaded webserver and the static lifetime makes sense because we have no idea when the thread will finish running.

```rust
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
    where
        F: FnOnce() -> T,
        F: Send + 'static,
        T: Send + 'static,
```

Our implementation will look like this.
We have FnOnce() because our closure takes no arguments.
Now an obvious caveat is that at the moment despite compilling (spoiler) our code doesn't actually do anything with our requests as yet so we will get the errors we got way at the beginning of the single threaded webserver.  This is where we would start writing unit tests to make sure the code compiles as well as having the functionality that we want it to have, not just that it compiles at all. 
```rust
impl ThreadPool {
	pub fn execute<F>(&self, f: F)
	where
		f: FnOnce() + Send + 'static,
	{
	}
}
```

```shell
$ cargo check
    Checking hello v0.1.0 (file:///projects/hello)
    Finished dev [unoptimized + debuginfo] target(s) in 0.24s
```

#### Creating the threads and making sure we don't accept bad stuff. 
The first place to start logically is now creating the ability to actually create our ThreadPool, there's no point creating an execute method for a ThreadPool that doesn't exist.  We've added some documentation to this so that when we call cargo doc --open we can see the documentation looking all nice and pretty. 
By adding an assert that size needs to be greater than 0 instead of returning a Result<ThreadPool, Error> we are saying that we want a thread count of 0 to be an unrecoverable situation. 

```rust
impl ThreadPool {
	/// Create a new threadpool.
	/// 
	/// The size is the number of threads in the pool.
	///
	/// # Panics
	///
	/// The `new` function will panic if the size is zero.
	pub fn new(size: usize) -> ThreadPool {
		assert!(size > 0);

		ThreadPool
	}
}
```

#### Creating space for the threads
We can now validate that we have a valid number of threads, we need to actually figure out where to store them. 
1. We actually need somewhere in the struct to hold onto the threads. We do that by creating a threads vector that contains a JoinHandle<()>
2. The reason we chose this is because the spawn function from threads also returns a JoinHandle<T,> but since our implementation takes a closure that returns nothing we can instead use the unit type () in-place of T. 
3. . We set a cap to the number of threads the vector can accepect by creating a blank vector of maximum size using the Vec::with_capacity() associated function.
4. We then use for loop on a range from 0 to size, but use the _ for the iterator value because we don't need it. But we still don't actually spawn anything as of yet. 
5. This will compile but still won't actually do antyhing.

```rust
pub struct ThreadPool {
    threads: Vec<thread::JoinHandle<()>>,
}

impl ThreadPool {
    // --snip--
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        let mut threads = Vec::with_capacity(size);

        for _ in 0..size {
            // create some threads and store them in the vector
        }

        ThreadPool { threads }
    }
    // --snip--
}
```

#### Using a worker struct to send code from the ThreadPool to a Thread.

The default thread::spawn() expects some code (a closure) to execute on the thread, but we want the threads to sit there and wait until we provide them with a request to execute. For this task we will create a *worker* (common term). These workers will each hold onto a JoinHandle and have a method which takes a closure and feeds it to the thread. Each worker will also have an id so that we can know which one is which. 

Our new implementation looks like so:

1. Define a worker struct that has the id and holds the JoinHandle we need to hold the thread. 
2. Change the ThreadPool so that it actually holds a vector of workers.
3. Populate the limited size vector of workers and provide each worker an id. 
4. Add a initializer for the worker that creates the thread and provides it some base code to run.
```rust
use std::thread;

pub struct ThreadPool {
    workers: Vec<Worker>,
}

impl ThreadPool {
    // --snip--
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        let mut workers = Vec::with_capacity(size);

		// Note we are now using the iterator to generate an id for out threads.
        for id in 0..size {
            workers.push(Worker::new(id));
        }

        ThreadPool { workers }
    }
    // --snip--
}

struct Worker {
    id: usize,
    thread: thread::JoinHandle<()>,
}

impl Worker {
    fn new(id: usize) -> Worker {
        let thread = thread::spawn(|| {});

        Worker { id, thread }
    }
}
```

But we are still not actually processing requests in any meaningful way (by which I mean at all).

#### Sending code to the workers via channels

```rust
use std::{sync::mpsc, thread};

pub struct ThreadPool {
    workers: Vec<Worker>,
    sender: mpsc::Sender<Job>,
}

struct Job;

impl ThreadPool {
    // --snip--
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        let (sender, receiver) = mpsc::channel();

        let mut workers = Vec::with_capacity(size);

        for id in 0..size {
            workers.push(Worker::new(id));
        }

        ThreadPool { workers, sender }
    }
    // --snip--
}

```

The issue here is that as is this won't work because we are trying to use multiple consumers on a single producer. We need to bring in the mutex. 

```rust
use std::{
    sync::{mpsc, Arc, Mutex},
    thread,
};
// --snip--

impl ThreadPool {
    // --snip--
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        let (sender, receiver) = mpsc::channel();

        let receiver = Arc::new(Mutex::new(receiver));

        let mut workers = Vec::with_capacity(size);

        for id in 0..size {
            workers.push(Worker::new(id, Arc::clone(&receiver)));
        }

        ThreadPool { workers, sender }
    }

    // --snip--
}

// --snip--

impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        // --snip--
    }
}

```

Now that we are providing each worker with a receiver we can provide the execute method a definition.

#### Implementing execute

For this we are passing a trait object containing the trait restrictions we ascertained would be appropriate for our closure. 
We can now package that closure inside a trait object and sent it to our Threads. 

```rust
// --snip--

type Job = Box<dyn FnOnce() + Send + 'static>;

impl ThreadPool {
    // --snip--

    pub fn execute<F>(&self, f: F)
    where
        F: FnOnce() + Send + 'static,
    {
        let job = Box::new(f);

        self.sender.send(job).unwrap();
    }
}

// --snip--

```

We've also had to update the initializer for our worker. It now takes an arc pointer to a mutex of a receiver of type Job. With an id of type usize which is assigned by the for loop. 
We also print which worker has started executing a job. 
When we create the worker we get the lock from, unwrap it to panic if we failed to get the lock and then call recv to receive the job. 

```rust
// --snip--

impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        let thread = thread::spawn(move || loop {
            let job = receiver.lock().unwrap().recv().unwrap();

            println!("Worker {id} got a job; executing.");

            job();
        });

        Worker { id, thread }
    }
}
```

WE HAVE A WORKING MULTI-THREADED SERVER.
#### Other options
We can also use the fork/join model, single threaded async I/O or multi-threaded async I/O. 

