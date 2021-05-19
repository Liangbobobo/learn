https://doc.rust-lang.org/std/sync/atomic/fn.compiler_fence.html  
# Function std::sync::atomic::compiler_fence  ---A compiler memory fence use in a single-thread.
pub fn compiler_fence(order: Ordering)   

compiler_fence does not emit any machine code, but restricts the kinds of memory re-ordering the compiler is allowed to do.   
Specifically, depending on the given Ordering semantics, the compiler may be disallowed from moving reads or writes from before or after the call to the other side of the call to compiler_fence. 

Note that it does not prevent the hardware from doing such re-ordering. 

This is not a problem in a single-threaded, execution context, but when other threads may modify memory at the same time, stronger synchronization primitives such as fence are required.

The re-ordering prevented by the different ordering semantics are:  
1.  SeqCst, no re-ordering of reads and writes across this point is allowed. 
2.  Release, preceding reads and writes cannot be moved past subsequent writes.前面的读写不能在之后的写入后移动
3.  Acquire, subsequent reads and writes cannot be moved ahead of preceding reads.后续的读写不能在随后的读取之前移动  
4.  AcqRel, both of the above rules are enforced.


compiler_fence is generally only useful for preventing a thread from racing with itself. That is, if a given thread is executing one piece of code, and is then interrupted, and starts executing code elsewhere (while still in the same thread, and conceptually still on the same core).    
In traditional programs, this can only occur when a signal handler is registered.在传统程序中，这只能在注册信号处理程序时发生

Panics if order is Relaxed.




eg:   
Without compiler_fence, the assert_eq! in following code is not guaranteed to succeed, despite everything happening in a single thread. To see why, remember that the compiler is free to swap the stores to IMPORTANT_VARIABLE and IS_READ since they are both Ordering::Relaxed. If it does, and the signal handler is invoked right after IS_READY is updated, then the signal handler will see IS_READY=1, but IMPORTANT_VARIABLE=0. Using a compiler_fence remedies this situation.
```
use std::sync::atomic::{AtomicBool, AtomicUsize};
use std::sync::atomic::Ordering;
use std::sync::atomic::compiler_fence;

static IMPORTANT_VARIABLE: AtomicUsize = AtomicUsize::new(0);
static IS_READY: AtomicBool = AtomicBool::new(false);

fn main() {
    IMPORTANT_VARIABLE.store(42, Ordering::Relaxed);
    // prevent earlier writes from being moved beyond this point
    compiler_fence(Ordering::Release);
    IS_READY.store(true, Ordering::Relaxed);
}

fn signal_handler() {
    if IS_READY.load(Ordering::Relaxed) {
        assert_eq!(IMPORTANT_VARIABLE.load(Ordering::Relaxed), 42);
    }
}
```






