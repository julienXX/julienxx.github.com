---
title:  "Using Resque with Rust"
date:   2015-10-29 14:30:00
description: "Enqueing and performing Resque jobs with the Rust programming language."
---

![Resque workers of the World, unite!](http://i.imgbox.com/D8eBMjnd.jpg)


*Disclaimer: I'm not a Rust expert by any means so please tell me if there is something I can improve in the examples below. I'm not going to explain how Rust works. If you're not familiar with it I highly encourage you to read [the Rust Book](https://doc.rust-lang.org/book/)*

[Resque](https://github.com/blog/542-introducing-resque) is a popular solution in the Ruby world to process background jobs. The great thing with it is the fact it uses Redis as a backend, making it easy to share jobs with workers written in other languages.

In this article we'll see how we can write a fully functional Resque worker in Rust. This will allow us to either use Resque entirely with Rust, enqueue a job in Ruby and perform it with Rust or vice versa.

The full code for the following examples can be found [here](https://github.com/julienXX/rust-resque-example).

## How does Resque work?

Resque jobs are enqueued in a Redis list using the `RPUSH` command.
A Resque job is represented internally with a JSON string containing two keys, one for the job class name and one for its arguments (or payload in Resque terms).

```json
{
    'class': 'JobClass',
    'args': [ arg1, arg2 ]
}
```

Available queues are defined in a `resque:queues` `Redis SET` whose entries are the queue name like `some_queue_name`.

Enqueueing a job is done by issueing a `RPUSH` command with a payload to some queue.

Knowing that let's enqueue our first job from Rust.

## Enqueing a Resque job

Let's pretend we need to enqueue a Resque job in order to send a confirmation email.

First we need to start a new project with:

```
λ cargo new rust-worker --bin
```

First we will need to add some crates.
In your Cargo.toml add:

```toml
[dependencies]
rustc-serialize = "0.3.16"

[dependencies.redis]
git = "https://github.com/mitsuhiko/redis-rs.git"
tag = "0.5.1"
```

Next let's use those crates. In our main.rs:

```rust
extern crate redis;
extern crate rustc_serialize;

use redis::Commands;
use rustc_serialize::Encodable;
use rustc_serialize::json;
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

    // Add our queue in resque:queues Set
    try!(conn.sadd("resque:queues", "rust_test_queue"));
    // Push our job in the resque:queue:rust_test_queue list
    try!(conn.rpush("resque:queue:rust_test_queue", json_job));

    println!("Enqueued job: {:?}", job);

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
        Err(_) => { /* handle failure here */ }
    }
}
```

![Resque screenshot](http://i.imgbox.com/GmeAjSnN.png)

Great! We successfully enqueued a job that can be performed through Resque.

Now onto the perform part.

## Performing a Resque job

A Resque worker will try to `reserve` a job by polling a queue with the `LPOP` command until it gets something to perform.

Let's implement that.

We need a reserve function that will check if a job is present in a queue:

```rust
fn reserve() -> redis::RedisResult<()> {
    println!("--: Checking rust_test_queue");

    // Connect to a local Redis
    let client = try!(redis::Client::open("redis://127.0.0.1/"));
    let conn = try!(client.get_connection());

    // Check if a job is present in the queue
    let res = conn.lpop("resque:queue:rust_test_queue").unwrap();

    // Perform the job or return
    match res {
        Some(job) => perform(job),
        None => return Ok(()),
    }
}
```

We also need to have a function that waits a few seconds to mimic Resque behaviour:

```rust
fn wait_a_bit() {
    println!("--: Sleeping for 5.0 seconds");
    std::thread::sleep_ms(5000);
}
```

In order to perform our job, we need to decode the JSON String retrieved from the Resque queue and then do something useful like sending an email in our case.

```rust
fn perform(json_job: String) -> redis::RedisResult<()> {
    println!("Found job: {:?}", json_job);

    // Decode JSON
    let job: Job = json::decode(&*json_job).unwrap();

    // Send our email with something like:
    // send_email(job.args.first());
    // not implemented here.

    Ok(())
}
```

Our Job struct must derive the `RustcDecodable` trait to be decodable. I'm decoding `&*json_job` since `json::decode` expects a `&str` and not a `String`.

Now let's update our `main()` function to enqueue a job, perform it and wait for other jobs to come.

```rust
fn main() {
    let job: Job = Job { class: "SignupEmail".to_owned(),
                         args: vec!["user@example.com".to_owned()] };

    enqueue(job).unwrap();

    loop {
        reserve().unwrap();
        wait_a_bit();
    }
}
```

Time to try our worker.

```console
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

Our implementation still needs to be Resque web compatible (display workers, failed jobs...). It also needs to be deployed alongside our Ruby workers but that's for another article :)

*Thanks a lot to the reviewers: Flavien, Marc, Steve & Yohan.*
