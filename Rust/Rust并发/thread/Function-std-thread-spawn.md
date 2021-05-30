https://doc.rust-lang.org/std/thread/fn.spawn.html

# Function std::thread::spawn
Declaration:   
```
pub fn spawn<F, T>(f: F) -> JoinHandle<T> where
    F: FnOnce() -> T,
    F: Send + 'static,
    T: Send + 'static, 
```

Spawns a new thread, returning a JoinHandle for it. 

The join handle will implicitly隐含 detach分离 the child thread upon being dropped.    
In this case, the child thread may outlive the parent (unless the parent thread is the main thread; the whole process is terminated when the main thread finishes).     
Additionally, the join handle provides a join method that can be used to join the child thread. If the child thread panics, join will return an Err containing the argument given to panic!. 

This will create a thread using default parameters of Builder, if you want to specify the stack size or the name of the thread, use this API instead.


As you can see in the signature签名 of spawn there are two constraints on both the closure given to spawn and its return value, let’s explain them:   
1. The 'static constraint means that the closure and its return value must have a lifetime of the whole program execution. The reason for this is that threads can detach and outlive the lifetime they have been created in. Indeed if the thread, and by extension its return value, can outlive their caller, we need to make sure that they will be valid afterwards, and since we can’t know when it will return we need to have them valid as long as possible, that is until the end of the program, hence the 'static lifetime.
2. The Send constraint is because the closure will need to be passed by value from the thread where it is spawned to the new thread. Its return value will need to be passed from the new thread to the thread where it is joined. As a reminder, the Send marker trait expresses that it is safe to be passed from thread to thread. Sync expresses that it is safe to have a reference be passed from thread to thread.


## Panics
Panics if the OS fails to create a thread; use Builder::spawn to recover from such errors.

Examples：  
```
use std::thread;

let handler = thread::spawn(|| {
    // thread code
});

handler.join().unwrap();
```
As mentioned in the module documentation, threads are usually made to communicate using channels, here is how it usually looks.

This example also shows how to use move, in order to give ownership of values to a thread.
```
use std::thread;
use std::sync::mpsc::channel;

let (tx, rx) = channel();

let sender = thread::spawn(move || {
    tx.send("Hello, thread".to_owned())
        .expect("Unable to send on channel");
});

let receiver = thread::spawn(move || {
    let value = rx.recv().expect("Unable to receive from channel");
    println!("{}", value);
});

sender.join().expect("The sender thread has panicked");
receiver.join().expect("The receiver thread has panicked");
```

A thread can also return a value through its JoinHandle, you can use this to make asynchronous computations (futures might be more appropriate though).   
```

use std::thread;

let computation = thread::spawn(|| {
    // Some expensive computation.
    42
});

let result = computation.join().unwrap();
println!("{}", result);
```


