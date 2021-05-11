# The Future Trait
The Future trait is at the center of asynchronous programming in Rust  
A Future is an asynchronous computation that can produce a value (although that value may be empty, e.g. ())   

# Trait std::future::Future
A future represents an asynchronous computation.  
A future is a value that may not have finished computing yet. This kind of “asynchronous value” makes it possible for a thread to continue doing useful work while it waits for the value to become available.   
Once a future has finished, clients should not poll it again.  
When using a future, you generally won’t call poll directly, but instead .await the value. 

## The poll method  
 
The core method of future, poll, attempts to resolve the future into a final value.   
This method does not block if the value is not ready. Instead, the current task is scheduled to be woken up when it’s possible to make further progress by polling again.   
The context passed to the poll method can provide a Waker, which is a handle for waking up the current task.   

## Associated Types
type Output：The type of value produced on completion.（完成后产生的值类型。）

## Required methods
```pub fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>```

This function returns:   
Poll::Pending if the future is not ready yet   
Poll::Ready(val) with the result val of this future if it finished successfully.   



# Future 用到的标准库
## Struct std::task::Context
The Context of an asynchronous task.  
Currently, Context only serves to provide access to a &Waker which can be used to wake the current task.（目前context，只服务于提供了访问一个用于唤醒当前任务的&waker）


##  Struct std::task::Waker   
A Waker is a handle for waking up a task by notifying its executor that it is ready to be run.   
