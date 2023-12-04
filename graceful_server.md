The big problem with [[single_threaded_webserver_example|both]] of our [[multi_threaded_webserver|servers]] implementations is that they both hard [[panic]] if anything goes wrong, for example if I use ctrl+c to close down the server every thread stops even if they are in the middle of doing something. To get around this we should implement the [[drop]] trait to call join so that all threads end before the close occurs.

### Implementing the drop trait

This current implementation will not compile because the join method takes ownership of the thread and we since we don't have ownership of the JoinHandle within the Worker so we can't provide join with ownership. If we want to do that we need to wrap the Worker's thread with an Option as this will allow us to use the take method.
```rust
impl Drop for ThreadPool {
    fn drop(&mut self) {
        for worker in &mut self.workers {
            println!("Shutting down worker {}", worker.id);

            worker.thread.join().unwrap();
        }
    }
}

```

After making this change the code still won't compile. In part because we haven't updated the initializer for the Worker struct as this still passes a raw JoinHandle which we now need to wrap in Some. We also need to use an if let with the .take() function because the take doesn't guarantee to return a JoinHandle it could return a None. 
```rust
struct Worker {
    id: usize,
    thread: Option<thread::JoinHandle<()>>,
}
```

We now have code that compiles however it doesn't behave how we want it to. The issue being that our server has a for loop that will forever be looking for requests and as such we will end up blocked on that. To do that we need to make the ThreadPool drop the sender so that it can no longer listen for new requests once the current requests have been shut down.
```rust
impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        // --snip--

        Worker {
            id,
            thread: Some(thread),
        }
    }
}

impl Drop for ThreadPool {
    fn drop(&mut self) {
        for worker in &mut self.workers {
            println!("Shutting down worker {}", worker.id);

            if let Some(thread) = worker.thread.take() {
                thread.join().unwrap();
            }
        }
    }
}
```

Here we drop the sender manually at the start of the drop trait so that the for loop can end. This results in all of our workers no longer receiving messages which causes them to error out. To make this more graceful we need to provide a way for the worker to handle this and shutdown nicely. 
```rust 
pub struct ThreadPool {
    workers: Vec<Worker>,
    sender: Option<mpsc::Sender<Job>>,
}
// --snip--
impl ThreadPool {
    pub fn new(size: usize) -> ThreadPool {
        // --snip--

        ThreadPool {
            workers,
            sender: Some(sender),
        }
    }

    pub fn execute<F>(&self, f: F)
    where
        F: FnOnce() + Send + 'static,
    {
        let job = Box::new(f);

        self.sender.as_ref().unwrap().send(job).unwrap();
    }
}

impl Drop for ThreadPool {
    fn drop(&mut self) {
        drop(self.sender.take());

        for worker in &mut self.workers {
            println!("Shutting down worker {}", worker.id);

            if let Some(thread) = worker.thread.take() {
                thread.join().unwrap();
            }
        }
    }
}
```

We do this by implementing a match statement that runs the job if they receive an Ok and breaks otherwise. 
```rust
impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        let thread = thread::spawn(move || loop {
            let message = receiver.lock().unwrap().recv();

            match message {
                Ok(job) => {
                    println!("Worker {id} got a job; executing.");

                    job();
                }
                Err(_) => {
                    println!("Worker {id} disconnected; shutting down.");
                    break;
                }
            }
        });

        Worker {
            id,
            thread: Some(thread),
        }
    }
}
```

To test if this works properly we will force the pool to take 2 requests and then shutdown.
We do this by using take(2) on the iterator to force it to only make 2 iterations. 
```rust
fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();
    let pool = ThreadPool::new(4);

    for stream in listener.incoming().take(2) {
        let stream = stream.unwrap();

        pool.execute(|| {
            handle_connection(stream);
        });
    }

    println!("Shutting down.");
}
```

### The total project
#### main.rs
```rust
// Brining inall of the necessary modules. can be abbreviated a bit
use hello::ThreadPool;
use std::fs;
use std::io::prelude::*;
use std::net::TcpListener;
use std::net::TcpStream;
use std::thread;
use std::time::Duration;

// I think this is how it could be done, but I think it is less redable. 

use hello::ThreadPool;
use std {
	fs,
	io::prelude:::*,
	net {
		TcpListener,
		TcpStream,
	},
	thread,
	time::Duration
};

fn main() {
	// We start listening on the local port 7878, unwrap leads to a panic if we
	// get an error while trying to bind to that port.
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();

	// We now create a threadpool of 4 threads.
    let pool = ThreadPool::new(4);

	// As a hypothetical we force the server to shut down gracefully after 
	// getting 2 requests.
    for stream in listener.incoming().take(2) {
	    // We unwrap the stream, one again panicking if we get an error.
        let stream = stream.unwrap();

		// We now execute code in one of the threads in our pool by passing it a
		// closure containing our handle_connection function.
        pool.execute(|| {
            handle_connection(stream);
        });
    }

	// A simple print used to show the server has shut down.
    println!("Shutting down.");
}

fn handle_connection(mut stream: TcpStream) {
	// I believe this creates a buffer which is an array of 1024 0s. 
	// This provides a way to store out stream into something. I assume 1024 is
	// just a bytes object that's way larger than we are going to need. 
    let mut buffer = [0; 1024];
    // Our stream then reads this buffer and panicks if it gets an error.
    stream.read(&mut buffer).unwrap();

	// This provides a bytes version of our GET string.
    let get = b"GET / HTTP/1.1\r\n";
	// This provides a bytes version of our GET and sleep string.
    let sleep = b"GET /sleep HTTP/1.1\r\n";

	// We now check if our buffer starts with the get bytes. 
	// if so we return a status of 200 and the content from hello.html.
	// if it's sleep we sleep for 5 seconds and then return the same. 
	// otherwise we just return the 404 error code and the 404.html.
    let (status_line, filename) = if buffer.starts_with(get) {
        ("HTTP/1.1 200 OK", "hello.html")
    } else if buffer.starts_with(sleep) {
        thread::sleep(Duration::from_secs(5));
        ("HTTP/1.1 200 OK", "hello.html")
    } else {
        ("HTTP/1.1 404 NOT FOUND", "404.html")
    };

	// We then load the content from the html as a string and use unwrap to panic
	// on error. 
    let contents = fs::read_to_string(filename).unwrap();

	// We then use the format! macro to load the content and status into a string
	// that can be passed as our response. 
    let response = format!(
        "{}\r\nContent-Length: {}\r\n\r\n{}",
        status_line,
        contents.len(),
        contents
    );

	// Writes the stream (I think this effectively posts it?)
    stream.write_all(response.as_bytes()).unwrap();
    // We then flush the stream?
    stream.flush().unwrap();
}
```

#### lib.rs
```rust
// Imports
use std::{
    sync::{mpsc, Arc, Mutex},
    thread,
};

// Defining our threadpool struct
pub struct ThreadPool {
	// This contains a vector of our workers and an Option wrapped mspc::Sender
	// Wrapped in an option so that when it receives nothing we can use the Err
	// value to close gracefully.
    workers: Vec<Worker>,
    sender: Option<mpsc::Sender<Job>>,
}
// Creating a type alias of job instead of having to provide the trait object for
// our closure over and over again. 
type Job = Box<dyn FnOnce() + Send + 'static>;

impl ThreadPool {
	// doc strings
    /// Create a new ThreadPool.
    ///
    /// The size is the number of threads in the pool.
    ///
    /// # Panics
    ///
    /// The `new` function will panic if the size is zero.
    // How we initialize a ThreadPool.
    pub fn new(size: usize) -> ThreadPool {
	    // We start by crashing the programme if we get 0 threads in our pool.
        assert!(size > 0);

		// We create our sender and receiver so that we can send jobs to our 
		// workers. 
        let (sender, receiver) = mpsc::channel();

		// Because we are going to be using 1 sender from this thread but 
		// multiple receivers we need to pass the receiver in a thread safe 
		// version of the Rc pointer... But this also needs to be mutable so we
		// need to pass wrap it further in a mutex. 
        let receiver = Arc::new(Mutex::new(receiver));

		// We then create a vector of workers with a maximum size set from 
		// outside. Currently empty.
        let mut workers = Vec::with_capacity(size);

		// We now populate the worker with an id based on the iterator and also
		// pass it a Arc to the receiver. 
        for id in 0..size {
            workers.push(Worker::new(id, Arc::clone(&receiver)));
        }

		// Finally we return the TreadPool and wrap the sender in a Some so that
		// we can drop it later and use that to set off the close. 
		// We have to do this manually as otherwise we will forever be waiting 
		// as this will sit in a for loop forever listening for new jobs until
		// we force it to stop by dropping it. 
        ThreadPool {
            workers,
            sender: Some(sender),
        }
    }

	// Implementing the execute function.
    pub fn execute<F>(&self, f: F)
    // Specifying that it needs to take a trait bound type that needs to 
    // implement FnOnce, Send AND have a static lifetime. Static because we don't
    // know how long the thread will last. 
    where
        F: FnOnce() + Send + 'static,
    {
	    // We are wrapping the closure in a box so we can pass it as a trait obj.
        let job = Box::new(f);

		// We are now sending the job to a worker. 
		// Which one it is sent to is determined by which one has the lock.
        self.sender.as_ref().unwrap().send(job).unwrap();
    }
}

// Implementing the drop trait so that the ThreadPool can close gracefully.
impl Drop for ThreadPool {
	// Obviously needs to be a method as well as a mut method so that we can 
	// make changes (drop) the sender. 
    fn drop(&mut self) {
	    // Dropping the sender for the reasons mentioned earlier.
        drop(self.sender.take());

		// Telling each worker to stop. We do this by taking the thread from the
		// worker. This is possible because the worker has its thread wrapped in
		// an option.
        for worker in &mut self.workers {
            println!("Shutting down worker {}", worker.id);
			// By taking it and calling join on in it we are basically making 
			// everything now merge into this thread and as such stopping them
			// from being able to take on more jobs.
            if let Some(thread) = worker.thread.take() {
                thread.join().unwrap();
            }
        }
    }
}

// Implementing our worker struct.
struct Worker {
	// Just an arbitrary id.
    id: usize,
    // Thread JoinHandle which basically translates to being a thread, wrapped in
    // an option for the reasons previously mentioned.
    thread: Option<thread::JoinHandle<()>>,
}

// Implementing the worker initializer. 
impl Worker {
	// Already been through why these types.
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
	    // We are moving the receiver into this closure so that it can take
		// take ownership of it so that it can take the lock and receive jobs.
		// This is all in a loop so that the loop code in the loop will execute 
		// constantly (which is why the thread doesn't close).
		// We pass this thread jobs via the pointer to the receiver that it has 
		// received.
        let thread = thread::spawn(move || loop {
	        // recieving the job.
            let message = receiver.lock().unwrap().recv();

			// This dictates when the worker shuts down, if we no longer have a
			// message we won't get an Okay and as such we will enter the Err arm
			// this arm breaks out of the loop and as such shuts down the thread.
            match message {
                Ok(job) => {
                    println!("Worker {id} got a job; executing.");

                    job();
                }
                Err(_) => {
                    println!("Worker {id} disconnected; shutting down.");
                    break;
                }
            }
        });

		// Here we return the worker to the ThreadPool. Reminder this can execute
		// while the above is running because this is on a different thread. 
        Worker {
            id,
            thread: Some(thread),
        }
    }
}
```