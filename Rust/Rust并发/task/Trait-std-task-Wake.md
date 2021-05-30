# Trait std::task::Wake
Declaration:  
```
pub trait Wake {
    pub fn wake(self: Arc<Self>);

    pub fn wake_by_ref(self: &Arc<Self>) { ... }
}
```

作用：   
The implementation of waking a task on an executor.  
This trait can be used to create a Waker.   
An executor can define an implementation of this trait, and use that to construct a Waker to pass to the tasks that are executed on that executor.

This trait is a memory-safe and ergonomic alternative to constructing a RawWaker.    
It supports the common executor design in which the data used to wake up a task is stored in an Arc.    
Some executors (especially those for embedded嵌入式 systems) cannot use this API, which is why RawWaker exists as an alternative选择 for those systems.


## Examples  

