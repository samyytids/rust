### HTTP and TCP
Both http and tcp are request-response protocols. A client initiates a request and the server sends the response. 
#### HTTP
*hypertext transfer protocol*
HTTP is the higher level of the two, dealing with the actual contents of the request and response. 
#### TCP
*Transmission control protocol*
TCP is the lower level of the two, detailing how information is gets from one server to the other but has no information about what that information is. 

### The actual project
1. [[single_threaded_webserver_example#Creating the project|Creating the project]]
2. [[single_threaded_webserver_example#Listening for TCP connections|Listening for TCP connections]]
3. [[single_threaded_webserver_example#Reading the request]]
4. [[single_threaded_webserver_example#Writing a response|Writing a response]]
5. [[single_threaded_webserver_example#Returning an HTML response|Returning an HTML response]]
6. [[single_threaded_webserver_example#A simple dynamic response|A simple dynamic response]]

#### Creating the project
Obviously as always we need to create the project using [[cargo]].
```shell
cargo new test_webserver
```

#### Listening for TCP connections
First we need to bring in the module that lets us listen to ports. 

##### Modules
```rust
use std::net::TcpListener;
```

##### Binding to a port
First we need to create tell our webserver to listen to a particular port. In this case we are using the local port (hence 127.0.0.1) 7878. The reason for choosing port 7878 is that it doesn't typically accept HTTP on this port so it is unlikely that we will run into any conflicts on this port. 
Bind in this case is very similar to new in that it will return an instance of TcpListener that listens to this specific port. 
We use unwrap because there is potential for the binding to fail (indicated by bind returning a result<T, E>).
We are using plain unwrap here because we are going for a simple example and if we aren't looking to deal with handling various issues that could occur when listening to a specific port.
```rust
let listener = TcpListener::bind("127.0.0.1:7878").unwrap();
```

##### Listening to that port
The incoming method returns an iterator containing a series of streams (in particular TcpStreams) these represent an open connection between the client and the server. A connection is the name given to the request response cycle, the client makes a request, the server generates a response and then closes the connection. As such we are going to be listening to this port to see what is being sent to the server and then respond based on what request we receive. 
Once again for simplicity at the moment we are simply unwrapping the stream to see if we got a non-error causing response and otherwise we panic. 
If we go to a web browser and connect to this port and hit refresh we should see a print statement for each refresh. 
```rust
for stream in listener.incoming() {
	let stream = stream.unwrap();

	println!("Connection established");
}
```

#### Reading the request
Now we need to actual start doing something with the request we are getting. 
##### Modules
```rust
// Importing everything from io::prelude and just io::BufReader as well as just
// net::TcpListener and net::TcpStream.
use std::{
	io::{prelude::*, BufReader},
	net::{TcpListener, TcpStream},
}
```

##### Handling the connection
Now that we have a way of accessing the requests made to a port we need to actually be able to do something with them. As is our requests are effectively being unheard by the server. 
We will do this using the following function:
```rust
// This function takes in a TcpStream and makes some adjustments to it and 
// nothing.
fn handle_connection(mut stream: TcpStream);
```
###### Using BufReader
The first part of this function makes use of std::io::BufReader.
This line wraps a mutable reference to the stream we are examining with the BufReader type which manages calls to std::io::Read trait method for us.
```rust
let buf_reader = Bufreader::new(&mut stream);
```

We then process this into an http request with the following line of code.
Now to break this down.
1. The list method is provided by the BufReader wrapper and splits the data in the stream based on new line bytes. Think python .split("\n"). 
2. Map then applies applies the closure it is provided to each element in the iterable produced by the lines method. We are just using unwrap here because this isn't going to be used for anything important. This result in our code panicking if provided non-UTF-8 characters. We use unwrap here once again because lines returns an iterator over Result<T, E> types. 
3. Take_while passes the request line into the closure it is provided. This simply checks if the line is empty, this signifies the end of the request because requests are terminated by two newline bytes. So, a blank string would imply 2 newline bytes thus suggesting we have reached the end of the request. I assume the back end of this take_while is effectively equivalent to doing a while loop that loops until line == "". 
4. The collect method then packages all of the lines that we have processed into the vector http_request
The print format here effectively gives us pretty printing.

```rust
let http_request Vec<_> = buf_reader
	.lines()
	.map(|result| result.unwrap())
	.take_while(|line| !line.is_empty())
	.collect();
println!("Request: {:#?}", http_request);
```

###### Breaking down the request string
```shell
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
    Finished dev [unoptimized + debuginfo] target(s) in 0.42s
     Running `target/debug/hello`
Request: [
    "GET / HTTP/1.1",
    "Host: 127.0.0.1:7878",
    "User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:99.0) Gecko/20100101 Firefox/99.0",
    "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8",
    "Accept-Language: en-US,en;q=0.5",
    "Accept-Encoding: gzip, deflate, br",
    "DNT: 1",
    "Connection: keep-alive",
    "Upgrade-Insecure-Requests: 1",
    "Sec-Fetch-Dest: document",
    "Sec-Fetch-Mode: navigate",
    "Sec-Fetch-Site: none",
    "Sec-Fetch-User: ?1",
    "Cache-Control: max-age=0",
]
```

1. The request line, this is the first line within the Request array. This contains the type of request being made, in this case GET or POST.
	1. GET is a request from the client to the server asking for information.
	2. POST is a request from the client for the server to accept information from the client. 
	This is then followed by the URI *uniform resource identifier* this is very similar to a URL *uniform resource locator*. This effectively says where the information the client is trying to request is located. 
	Finally we then detail the HTTP version the client is using in this case HTTP 1.1. This line also ends with a CRLF sequence *carriage retrun and line feed* these are typewriter terms, which we can think of as \\r\\n. This is basically saying split here, new information ahead. 

2. The remainder are request headers which detail information like the host port. GET requests have no body (I assume because they aren't providing any information to place in a body).




#### Writing a response
Now that we know what the user is requesting, we need to provide a response. 
Responses are provided in the following format:
```
HTTP-version Status-Code Reason-Phrase CRLF
headers CRLF
message-body
```

The status code provides a summary of the response being provided and the reason phrase provides extra information such as why this response type is what was received. 

A very simple response would look like so.
```
HTTP/1.1 200 OK\r\n\r\n
```
This response basically says, yep you connected alright. Here's no information. 

Now to actually do this in rust.
Here we are providing a hard coded response of no information beyond yep you got a response and it was good. 
The write_all() method takes in a reference to a slice of u8, or bytes information and returns a Result<T, E> once again, we are handling this result with an unwrap for the sake of ease. 
```rust
fn handle_connection(mut stream: TcpStream) {
	// --snip--
	let response = "HTTP/1.1 200 OK\r\n\r\n";
	stream.write_all(response.as_bytes())).unwrap();
}
```



#### Returning an HTML response
Right, so far we have not returned any responses the user can really see beyond now at least not returning an error!
In order to return and HTML file we need to load in an HTML file (this is a very simple response since this is hardcoded).
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>Hello!</title>
  </head>
  <body>
    <h1>Hello!</h1>
    <p>Hi from Rust</p>
  </body>
</html>
```

##### Modules
To use this HTML file we need to add he fs module to our code.
```rust
use std::fs;
```

##### Actually adding the html content to our response
1. Since we are using pure unwraps we don't need to handle anything other than a 200 response since anything else will lead to a crash. 
2. We then use the fs::read_to_string associated function to read in the content of the hello.html (which I assume based on the lack of file path is locally stored in the same file as the main.rs file?). We then use unwrap for the same reason as already mentioned too many times. 
3. We then find the length of the contents file.
4. This is then all formatted into a new response string using the format! macro.
5. Finally we once again write the response into the stream as bytes and use the unwrap to panic if we've fucked up.

```rust
fn handle_connection(mut stream: TcpStream) {
	//--snip--
	let status_line = "HTTP/1.1 200 OK";
	let contents = fs::read_to_string("hello.html").unrwap();
	let length = contents.len();
	let response = format!(
		"{status_line}\r\nContent-length: {length}\r\n\r\n{contents}"
	);
	stream.write_all(response.as_bytes()).unwrap();
}
```

This solution is incredibly bare bones and completely ignores the request being sent. IE no matter what you include in the GET request this is what you will get, no matter the URI and no matter the parameters in said URI. 

#### A simple dynamic response
Now so far we have provided a catchall response that gives the same thing no matter what is contained in the request.  We are effectively just going to use the URI to either provide a 200 or 404 response. 
To do this we need to check the first line of the request which will require a restructuring of our handle_connection function.
1. Now instead of processing the entire request into the http_request variable we are are calling the next method once to load in the request line. 
2. We then run an if statement checking if the request_line is equal to a specific string. IF it is we provide our default response.
3. Otherwise we load error 404.html instead of hello.html and set our response information to the error code 404 and a NOT FOUND response methods.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>Hello!</title>
  </head>
  <body>
    <h1>Oops!</h1>
    <p>Sorry, I don't know what you're asking for.</p>
  </body>
</html>

```

```rust
fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&mut stream);
    let request_line = buf_reader.lines().next().unwrap().unwrap();

    if request_line == "GET / HTTP/1.1" {
        let status_line = "HTTP/1.1 200 OK";
        let contents = fs::read_to_string("hello.html").unwrap();
        let length = contents.len();

        let response = format!(
            "{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}"
        );

        stream.write_all(response.as_bytes()).unwrap();
    } else {
		let status_line = "HTTP/1.1 404 NOT FOUND";
        let contents = fs::read_to_string("404.html").unwrap();
        let length = contents.len();

        let response = format!(
            "{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}"
        );

        stream.write_all(response.as_bytes()).unwrap();
    }
}
```

With a little bit of refactoring we can reduce the amount of code repetition we have.
This way we define both of the string immediately and then execute the remaining common code based on the result of the let if. Which uses [[patterns#Destructuring structs and tuples|patterns and destructuring]] to set both of our status_line and filename variables based on the value we get for request_line.  
```rust
fn handle_connection(mut stream: TcpStream) {
    // --snip--

    let (status_line, filename) = if request_line == "GET / HTTP/1.1" {
        ("HTTP/1.1 200 OK", "hello.html")
    } else {
        ("HTTP/1.1 404 NOT FOUND", "404.html")
    };

    let contents = fs::read_to_string(filename).unwrap();
    let length = contents.len();

    let response =
        format!("{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}");

    stream.write_all(response.as_bytes()).unwrap();
}
```