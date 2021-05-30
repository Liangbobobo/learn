# Struct std::thread::JoinHandle
Declaration:   
``` pub struct JoinHandle<T>(_); ```

An owned permission to join on a thread (block on its termination).   
A JoinHandle detaches the associated thread when it is dropped, which means that there is no longer any handle to thread and no way to join on it.    
Due to platform restrictions, it is not possible to Clone this handle: the ability to join a thread is a uniquely-owned permission.    

## 构造：   
This struct is created by the thread::spawn function and the thread::Builder::spawn method.

1. Creation from thread::spawn:  
```

use std::thread;

let join_handle: thread::JoinHandle<_> = thread::spawn(|| {
    // some work here
});
```

2. Creation from thread::Builder::spawn:   
```
use std::thread;

let builder = thread::Builder::new();

let join_handle: thread::JoinHandle<_> = builder.spawn(|| {
    // some work here
}).unwrap();
```

Child being detached and outliving its parent:   
```

use std::thread;
use std::time::Duration;

let original_thread = thread::spawn(|| {
    let _detached_thread = thread::spawn(|| {
        // Here we sleep to make sure that the first thread returns before.
        thread::sleep(Duration::from_millis(10));
        // This will be called, even though the JoinHandle is dropped.
        println!("♫ Still alive ♫");
    });
});

original_thread.join().expect("The thread being joined has panicked");
println!("Original thread is joined.");

// We make sure that the new thread has time to run, before the main
// thread returns.

thread::sleep(Duration::from_millis(1000));
```

## Implementations
```
 impl<T> JoinHandle<T>    
 Extracts a handle to the underlying thread.

 use std::thread;

let builder = thread::Builder::new();

let join_handle: thread::JoinHandle<_> = builder.spawn(|| {
    // some work here
}).unwrap();

let thread = join_handle.thread();
println!("thread id: {:?}", thread.id());  
 
 ```


``` pub fn join(self) -> Result<T> ```
Waits for the associated thread to finish.
In terms of atomic memory orderings, the completion of the associated thread synchronizes with this function returning. In other words, all operations performed by that thread are ordered before all operations that happen after join returns.


If the child thread panics, Err is returned with the parameter given to panic!.    
Panics:    
This function may panic on some platforms if a thread attempts to join itself or otherwise may create a deadlock with joining threads.

```
use std::thread;

let builder = thread::Builder::new();

let join_handle: thread::JoinHandle<_> = builder.spawn(|| {
    // some work here
}).unwrap();
join_handle.join().expect("Couldn't join on the associated thread");

```

Trait Implementations:   




