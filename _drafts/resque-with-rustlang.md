---
title:  "Using Resque with Rust"
date:   2015-10-26 14:40:00
description: "Enqueing and performing Resque jobs with the Rust programming language."
---

*Disclaimer: I'm not a Rust expert by any means so please tell me if there is something I can improve in the examples below. I'm not going to explain how Rust works, if you haven't yet you should have a look at [the Rust Book](https://doc.rust-lang.org/book/)*

The full code for the following examples can be found [here](https://github.com/julienXX/rust-resque-example).

[Resque](https://github.com/blog/542-introducing-resque) is a popular solution in the Ruby world to process background jobs. It uses Redis as a backend to store jobs in queues.

What I really like about Resque is that I can process jobs in other languages than Ruby if needed. Let's see how we can write a Resque worker with the Rust programming language.

## How Resque works?

Resque jobs are enqueued in a Redis list using the `RPUSH` command.
A Resque job is represented internally with a JSON string containing two keys, one for the job class name and one for its arguments.

```json
{
    'class': 'JobClass',
    'args': [ arg1, arg2 ]
}
```

Queues are just a Redis SET whose entries looks like `resque:queue_name`.

Knowing that let's enqueue our first job from Rust.

## Enqueing a Resque job

Let's pretend we need to enqueue a Resque job to send an email upon registering our awesome service. Our perform method on the Ruby side look like:

```ruby
class SignupEmail
  @queue = :rust_test_queue

  def self.perform(email)
    ...
  end
end 
```

Let's start a new project with:

```shell
λ cargo new rust-worker --bin
```

First we will need to add some crates.
In your Cargo.toml add:

```toml
[dependencies]
redis = "*"
rustc-serialize = "*"
```

Next let's use those crates. In our main.rs:

```rust
extern crate redis;
extern crate rustc_serialize;

use redis::Commands;
use rustc_serialize::Encodable;
use rustc_serialize::json::{self};
```

For our example we need to define a Job struct with with two fields: a class that will be a String and args that will be a vector of Strings.

```rust
#[derive(RustcEncodable, Debug)]
pub struct Job {
    class: String,
    args: Vec<String>
}
```

This struct needs the `RustcEncodable` trait so that we can encode it in JSON and the `Debug` trait for printing purposes so we'll derive them.

Next, we will define an `enqueue` function that will take a `Job` as input and return a `Redis Result`:

```rust
fn enqueue(job: Job) -> redis::RedisResult<Job> {
    // Connect to a local Redis
    let client = try!(redis::Client::open("redis://127.0.0.1/"));
    let conn = try!(client.get_connection());

    // Encode our Job in JSON
    let json_job = json::encode(&job).unwrap();

    // Add our queue in the resque:queues Redis set
    // let _: () throws away the result, just making sure it does not fail
    let _: () = try!(conn.sadd("resque:queues", "rust_test_queue"));

    // Push our job in the resque:queue:rust_test_queue Redis list
    let _: () = try!(conn.rpush("resque:queue:rust_test_queue", json_job));

    Ok(job)
}
```

Finally we create and enqueue a job in our main function:

```rust
fn main() {
    // Create a Job
    let job: Job = Job { class: "SignupEmail".to_owned(),
                         args: vec!["user@example.com".to_owned()] };

    // Enqueue our job
    match enqueue(job) {
        Ok(job) => println!("Enqueued job: {:?}", job),
        Err(_) => { /* enqueue can't fail right now but you could handle failure here */ }
    }
}
```

![Resque screenshot](https://photos-4.dropbox.com/t/2/AAARR-dh3ArkvMUHf3aVTpX7fSkuvQcF217ailcZ2oPBjg/12/536328/png/32x32/1/1445886000/0/2/Screenshot%202015-10-26%2018.51.07.png/CIjeICABIAIgAyAFIAcoAg/RZH07sQij-UY1efKeVMh9R2wkI4DwboutFj_B2q808o?size_mode=5)

Great we successfully enqueued a job that can be performed through Resque.

Now onto the perform part.

## Performing a Resque job

A Resque worker will try to `reserve` a job by polling a queue with the `LPOP` command until it gets something to perform.

Let's implement that.

We need a reserve function that will check if a job is present in a queue or wait a few seconds:

```rust
fn reserve() -> redis::RedisResult<()> {
    println!("--: Checking rust_test_queue");

    // Connect to a local Redis
    let client = try!(redis::Client::open("redis://127.0.0.1/"));
    let conn = try!(client.get_connection());

    // Removes and returns the first element of the list
    let res = conn.lpop("resque:queue:rust_test_queue").unwrap();

    // Perform the job or wait
    match res {
        Some(job) => perform(job),
        None => Ok(wait_a_bit()),
    }
}
```

We also make a function that waits 5 seconds and then tries to reserve again:

```rust
fn wait_a_bit() {
    println!("--: Sleeping for 5.0 seconds");
    std::thread::sleep_ms(5000);

    let _ = reserve();
}
```

In order to perform a job, we need to decode the JSON String retrieved from the Resque queue.

```rust
fn perform(json_job: String) -> redis::RedisResult<()> {
    let job: Job = json::decode(&*json_job).unwrap();
    println!("Found job: {:?}", job);

    // Do something with our job.

    Ok(())
}
```

Our Job struct must derive the `RustcDecodable` trait to be decodable. I'm decoding `&*json_job` since `json::decode` expects a `&str` and not a `String`.

Now let's update our `main()` function to make a complete flow through Resque.

```rust
fn main() {
    let job: Job = Job { class: "SignupEmail".to_owned(),
                         args: vec!["user@example.com".to_owned()] };

    // I use 'let _ =' because we don't use the return value
    let _ = enqueue(job);

    loop {
        let _ = reserve();
    }
}
```

Time to try our worker.

```shell
λ cargo build
   Compiling rust-resque-example v0.1.0 (file:///Users/julien/Code/rust-resque-example)

λ cargo run
     Running `target/debug/rust-resque-example`
Enqueued job: Job { class: "SignupEmail", args: ["user@example.com"] }
--: Checking rust_test_queue
Found job: Job { class: "SignupEmail", args: ["user@example.com"] }
--: Checking rust_test_queue
--: Sleeping for 5.0 seconds
--: Checking rust_test_queue
--: Sleeping for 5.0 seconds
```

And it works! We successfully enqueued and performed a job in Resque from Rust.

## What's missing?

Our very basic implementation still needs to be Resque web compatible and to handle failed jobs the Resque way. It also needs to be deployed alongside our Ruby workers but that's for another article :)
