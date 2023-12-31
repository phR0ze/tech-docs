# Concurrency
<img align="left" width="48" height="48" src="../../../../../data/images/logo_256x256.png">

[Concurrent programming](https://doc.rust-lang.org/book/ch16-00-concurrency.html) is where different 
parts of a program execute independently while parallel programming is where different parts of a 
program execute at the same time. So an application that runs in parallel can be said to be 
concurrent but the same is not necessarily true in reverse.

### Quick links
* [Runtime](#runtime)
* [Standard runtime](#standard-runtime)
* [Standard Threading](#standard-threading)
* [move closure](#move-closure)
* [Message passing](#message-passing)
* [Sync and Send traits](#sync-and-send-traits)
* [Blocking vs non-blocking](#blocking-vs-non-blocking)

## Runtime
Rust defines `runtime` support as the code that is included in every binary to manage aspects of the 
language. Every non-assembly language includes a runtime of some sort, and in some cases can be quite 
large (i.e. Golang) providing advanced functionality such as garbage collection and thread management 
(i.e. `green threads`).  Rust explicitely chose to not include a large runtime with advanced features 
to keep the binary small and make any runtime inclusion a conscious decision for developers.

### Standard runtime
The standard Rust runtime included in every binary is as small as possible providing no advanced 
threading controls and no garbage collection keeping Rust on par with C++ for performance.

* Tiny performant runtime
* `1:1` default thread implementation rather than `green threads`

### Standard threading
Rust's threading implementation simply wraps the underlying primitives keeping the runtime 
implementation as thin as possible. In Rust all threads will terminate when the main thread exists if 
you don't specifically wait for the spawned threads.

```rust
use std::thread;
use std::time::Duration;

fn main() {
    thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }
}
```

## move closure
The `move` closure is often used with `thread::spawn` because it allows you to use data from one 
thread in another thread. By using the `move` keyword before the thread's paramater list you force 
the thread to take ownership for the values it uses in the environment.

```rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(move || {
        println!("Here's a vector: {:?}", v);
    });

    handle.join().unwrap();
}
```

## Message passing
Golang's creators say `do not communicate by sharing memory` i.e. they prefer message-passing. The 
reason they are so adamant about it is that message-passing captures the same single owner concept as 
Rust's default stance which is very helpful in concurrant programming. However Rust has taken this 
further enforcing clear ownership rules through out its design. `Channels` are a modern concept 
popularized by Golang that Rust also implements. Channels have two halves: a producer and a consumer. 
The producer half is the upstream where you put items that flow down to the consumer end. The channel 
is said to be `closed` if either the producer or consumer is dropped. Rust allows for multiple 
producers but only a single consumer with the `std::sync::mpsc` implementation.

We can move the transmitting end into a thread providing a means of having multiple workers sending 
messages back to the main thread.
```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap();
    });

    let received = rx.recv().unwrap();
    println!("Got: {}", received);
}
```

The call in the main thread to `recv()` will block until a messages is received. Alternatively there 
is a `try_recv()` that is non-blocking. Additionally you can use a channel in a for loop to treat it 
like an iterator which will block internally calling `recv()` and complete the loop when the channel 
is closed. Simply `tx.clone()` the transmitter a.k.a. producer side to get a copy for each thread to 
work with in a multiple producer situation.

## Shared state
Shared memory, unlike message-passing, doesn't have a single owner. Multiple threads can access the 
same memory. In these cases you have to use mutual exclusion techniques to guard the use of the 
memory to only a single thread at a time. 

```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];
    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();

            *num += 1;
        });
        handles.push(handle);
    }
    for handle in handles {
        handle.join().unwrap();
    }
    println!("Result: {}", *counter.lock().unwrap());
}
```

The call to `lock()` is blocking but will fail if another thread that is holding the lock panicked 
and no one would ever be able to get the lock so using `unwrap()` is appropriate. In order to share a 
value between threads you need to wrap it in an `Atomic Reference Counter Arc<T>` and use a `Mutex`

## Sync and Send traits
`Sync` and `Send` indicate that a particular type is safe to use in a concurrent situation.

### Send trait
The `Send` trait indicates that ownership of values of the type implementing `Send` can be 
transferred between threads. Almost every Rust type is `Send`, but there are some exceptions 
including `Rc<T>`. The `Sync` trait indicates that it is safe for the type implementing `Sync` to be 
referenced from multiple threads. Types that are made up of `Send` and `Sync` based types are 
automatically also `Send` and `Sync`. We don't have to implement the traits and infact are not 
allowed to without using `Unsafe` code.

## Blocking vs non-blocking
The naive way to write an application that works on many things at the same time is to spawn a new 
thread for every task. This will lead to unsustainable amounts of overhead as the number of tasks 
becomes large. Improving on this concept is to have a pool of worker threads that are reused and pull 
work from a queue.

<!-- 
vim: ts=2:sw=2:sts=2
-->
