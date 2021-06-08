https://doc.rust-lang.org/std/task/index.html  
# Module std::task   
Types and Traits for working with asynchronous tasks.  

## Macros
ready  	Experimental Extracts the successful type of a ```Poll<T>```.

## Structs
>Context----The Context of an asynchronous task.  
Waker----A Waker is a handle for waking up a task by notifying its executor that it is ready to be run.  
RawWaker----A RawWaker allows the implementor of a task executor to create a Waker which provides customized wakeup behavior.    
RawWakerVTable----A virtual function pointer table (vtable) that specifies the behavior of a RawWaker.   


### Struct std::task::Context
``` pub struct Context<'a> { /* fields omitted */ } ```   
The Context of an asynchronous task.    
Currently, Context only serves to provide access to a &Waker which can be used to wake the current task.   

Implementations:   
impl<'a> Context<'a>    

pub fn from_waker(waker: &'a Waker) -> Context<'a>    
Create a new Context from a &Waker.   

pub fn waker(&self) -> &'a Waker    
Returns a reference to the Waker for the current task.


### Struct std::task::Waker
A Waker is a handle for waking up a task by notifying its executor that it is ready to be run.    
This handle encapsulates a RawWaker instance, which defines the executor-specific wakeup behavior.   
Implements Clone, Send, and Sync.   

Implementations:   
impl Waker   
pub fn wake(self)  
Wake up the task associated with this Waker.   

pub fn wake_by_ref(&self)     
Wake up the task associated with this Waker without consuming the Waker.       
This is similar to wake, but may be slightly less efficient in the case where an owned Waker is available. This method should be preferred to calling waker.clone().wake().   

pub fn will_wake(&self, other: &Waker) -> bool    
Returns true if this Waker and another Waker have awoken the same task.    
This function works on a best-effort basis, and may return false even when the Wakers would awaken the same task. However, if this function returns true, it is guaranteed that the Wakers will awaken the same task.    
This function is primarily used for optimization purposes.   

pub unsafe fn from_raw(waker: RawWaker) -> Waker    
Creates a new Waker from RawWaker.   
The behavior of the returned Waker is undefined if the contract defined in RawWaker’s and RawWakerVTable’s documentation is not upheld. Therefore this method is unsafe.    

### Struct std::task::RawWaker
A RawWaker allows the implementor of a task executor to create a Waker which provides customized wakeup behavior.    
It consists of a data pointer and a virtual function pointer table (vtable) that customizes the behavior of the RawWaker.    

Implementations:   
impl RawWaker    
pub const fn new(data: *const (), vtable: &'static RawWakerVTable) -> RawWaker    
Creates a new RawWaker from the provided data pointer and vtable.   
The data pointer can be used to store arbitrary data as required by the executor. This could be e.g. a type-erased pointer to an Arc that is associated with the task. The value of this pointer will get passed to all functions that are part of the vtable as the first parameter.   

The vtable customizes the behavior of a Waker which gets created from a RawWaker. For each operation on the Waker, the associated function in the vtable of the underlying RawWaker will be called.   

### Struct std::task::RawWakerVTable
A virtual function pointer table (vtable) that specifies the behavior of a RawWaker.   

The pointer passed to all functions inside the vtable is the data pointer from the enclosing RawWaker object.    
The functions inside this struct are only intended to be called on the data pointer of a properly constructed RawWaker object from inside the RawWaker implementation. Calling one of the contained functions using any other data pointer will cause undefined behavior.

Implementations:    
impl RawWakerVTable   
``` 
pub const fn new(
    clone: unsafe fn(*const ()) -> RawWaker,
    wake: unsafe fn(*const ()),
    wake_by_ref: unsafe fn(*const ()),
    drop: unsafe fn(*const ())
) -> RawWakerVTable 
```  

Creates a new RawWakerVTable from the provided clone, wake, wake_by_ref, and drop functions.    

clone:   
This function will be called when the RawWaker gets cloned, e.g. when the Waker in which the RawWaker is stored gets cloned.   

The implementation of this function must retain all resources that are required for this additional instance of a RawWaker and associated task. Calling wake on the resulting RawWaker should result in a wakeup of the same task that would have been awoken by the original RawWaker.

wake:   
This function will be called when wake is called on the Waker. It must wake up the task associated with this RawWaker.   

The implementation of this function must make sure to release any resources that are associated with this instance of a RawWaker and associated task.

wake_by_ref:   
This function will be called when wake_by_ref is called on the Waker. It must wake up the task associated with this RawWaker.   

This function is similar to wake, but must not consume the provided data pointer.

drop:   
This function gets called when a RawWaker gets dropped.   
The implementation of this function must make sure to release any resources that are associated with this instance of a RawWaker and associated task.


## Enums
Poll:      	
Indicates whether a value is available or if the current task has been scheduled to receive a wakeup instead.


## Traits
Wake:         
The implementation of waking a task on an executor.    

### Trait std::task::Wake
```
pub trait Wake {
    pub fn wake(self: Arc<Self>);

    pub fn wake_by_ref(self: &Arc<Self>) { ... }
}

```

The implementation of waking a task on an executor.

This trait can be used to create a Waker. An executor can define an implementation of this trait, and use that to construct a Waker to pass to the tasks that are executed on that executor.

This trait is a memory-safe and ergonomic alternative to constructing a RawWaker. It supports the common executor design in which the data used to wake up a task is stored in an Arc. Some executors (especially those for embedded systems) cannot use this API, which is why RawWaker exists as an alternative for those systems.

## Examples
A basic block_on function that takes a future and runs it to completion on the current thread.   

Note: This example trades correctness for simplicity. In order to prevent deadlocks, production-grade implementations will also need to handle intermediate calls to thread::unpark as well as nested invocations.

```

use std::future::Future;
use std::sync::Arc;
use std::task::{Context, Poll, Wake};
use std::thread::{self, Thread};

/// A waker that wakes up the current thread when called.
struct ThreadWaker(Thread);

impl Wake for ThreadWaker {
    fn wake(self: Arc<Self>) {
        self.0.unpark();
    }
}

/// Run a future to completion on the current thread.
fn block_on<T>(fut: impl Future<Output = T>) -> T {
    // Pin the future so it can be polled.
    let mut fut = Box::pin(fut);

    // Create a new context to be passed to the future.
    let t = thread::current();
    let waker = Arc::new(ThreadWaker(t)).into();
    let mut cx = Context::from_waker(&waker);

    // Run the future to completion.
    loop {
        match fut.as_mut().poll(&mut cx) {
            Poll::Ready(res) => return res,
            Poll::Pending => thread::park(),
        }
    }
}

block_on(async {
    println!("Hi from inside a future!");
});
```


## Required methods
``` pub fn wake(self: Arc<Self>) ```   
Wake this task.   

```pub fn wake_by_ref(self: &Arc<Self>) ```   
Wake this task without consuming the waker.   
If an executor supports a cheaper way to wake without consuming the waker, it should override this method. By default, it clones the Arc and calls wake on the clone.
