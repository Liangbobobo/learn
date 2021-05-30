# Module std::thread
## The threading model
An executing Rust program consists of a collection of native OS threads, each with their own stack and local state.   
Threads can be named, and provide some built-in support for low-level synchronization.

## 线程间通信
Communication between threads can be done through channels, Rust’s message-passing types, along with other forms of thread synchronization and shared-memory data structures.

In particular, types that are guaranteed to be threadsafe are easily shared between threads using the atomically-reference-counted container, Arc.

## panic 
Fatal logic errors in Rust cause thread panic, during which a thread will unwind the stack, running destructors and freeing owned resources.    
While not meant as a ‘try/catch’ mechanism, panics in Rust can nonetheless be caught (unless compiling with panic=abort) with catch_unwind and recovered from, or alternatively be resumed with resume_unwind.     
If the panic is not caught the thread will exit, but the panic may optionally be detected from a different thread with join. If the main thread panics without the panic being caught, the application will exit with a non-zero exit code.

## 生命周期及依赖性
When the main thread of a Rust program terminates, the entire program shuts down, even if other threads are still running. However, this module provides convenient facilities for automatically waiting for the termination of a child thread (i.e., join).

## 用法 Spawning a thread
A new thread can be spawned using the thread::spawn function:  
```
use std::thread;

thread::spawn(move || {
    // some work here
});

In this example, the spawned thread is “detached” from the current thread. 
This means that it can outlive活得更长 its parent 
(the thread that spawned it), 
unless this parent is the main thread.
```

The parent thread can also wait on the completion of the child thread; a call to spawn produces a JoinHandle, which provides a join method for waiting:   
```
use std::thread;

let child = thread::spawn(move || {
    // some work here
});
// some work here
let res = child.join();
```
The join method returns a thread::Result containing Ok of the final value produced by the child thread, or Err of the value given to a call to panic! if the child panicked.

## Configuring threads
A new thread can be configured before it is spawned via the Builder type, which currently allows you to set the name and stack size for the child thread:  
```
use std::thread;

thread::Builder::new().name("child1".to_string()).spawn(move || {
    println!("Hello, world!");
});
```


## The Thread type
Threads are represented via the Thread type, which you can get in one of two ways:   
1. By spawning a new thread, e.g., using the thread::spawn function, and calling thread on the JoinHandle.    
2. By requesting the current thread, using the thread::current function.   

The thread::current function is available even for threads not spawned by the APIs of this module.



## Thread-local storage
This module also provides an implementation of thread-local storage for Rust programs.    
Thread-local storage is a method of storing data into a global variable that each thread in the program will have its own copy of.   
Threads do not share this data, so accesses do not need to be synchronized.   

A thread-local key owns the value it contains and will destroy the value when the thread exits.   
It is created with the thread_local! macro and can contain any value that is 'static (no borrowed pointers).    
It provides an accessor function, with, that yields a shared reference to the value to the specified closure. Thread-local keys allow only shared access to values, as there would be no way to guarantee uniqueness if mutable borrows were allowed.   
Most values will want to make use of some form of interior mutability through the Cell or RefCell types.