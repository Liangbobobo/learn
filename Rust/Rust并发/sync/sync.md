# std::sync； Useful synchronization primitives.
## The need for synchronization 同步操作的需要  
Conceptually, a Rust program is a series of operations which will be executed on a computer.   
The timeline of events happening in the program is consistent with the order of the operations in the code.

但是compiler及hardware，可能会对code进行优化，改变其执行的顺序。（详见）

Note that thanks to Rust’s safety guarantees, accessing global (static) variables requires unsafe code, assuming we don’t use any of the synchronization primitives in this module.

## Out-of-order execution 
Instructions can execute in a different order from the one we define, due to various reasons:    
1. The compiler reordering instructions：   
If the compiler can issue an instruction at an earlier point, it will try to do so. For example, it might hoist memory loads at the top of a code block, so that the CPU can start prefetching the values from memory.   
In single-threaded scenarios, this can cause issues when writing signal handlers or certain kinds of low-level code. Use compiler fences to prevent this reordering.

2. A single processor executing instructions out-of-order:    
Modern CPUs are capable of superscalar execution, i.e., multiple instructions might be executing at the same time, even though the machine code describes a sequential process.   
This kind of reordering is handled transparently透明的 by the CPU.  

3. A multiprocessor system executing multiple hardware threads at the same time:   
 In multi-threaded scenarios, you can use two kinds of primitives to deal with synchronization:   
 memory fences to ensure memory accesses are made visible to other CPUs in the right order.   
 atomic operations to ensure simultaneous access to the same memory location doesn’t lead to undefined behavior.

 ## Higher-level synchronization objects
 Most of the low-level synchronization primitives are quite error-prone易于 and inconvenient to use, which is why the standard library also exposes some higher-level synchronization objects.   

These abstractions can be built out of lower-level primitives. For efficiency, the sync objects in the standard library are usually implemented with help from the operating system’s kernel, which is able to reschedule the threads while they are blocked on acquiring a lock.

The following is an overview of the available synchronization objects:   
1. Arc:   
 Atomically Reference-Counted pointer, which can be used in multithreaded environments to prolong延长 the lifetime of some data until all the threads have finished using it.

2. Barrier:   
Ensures multiple threads will wait for each other to reach a point in the program, before continuing execution all together.

3. Condvar:    
Condition Variable, providing the ability to block a thread while waiting for an event to occur.

4. mpsc:    
Multi-producer, single-consumer queues, used for message-based communication. Can provide a lightweight inter-thread synchronisation mechanism, at the cost of some extra memory.

5. Mutex:    
Mutual Exclusion mechanism, which ensures that at most one thread at a time is able to access some data.

6. Once:   
Used for thread-safe, one-time initialization of a global variable.

7. RwLock:  
 Provides a mutual exclusion mechanism which allows multiple readers at the same time, while allowing only one writer at a time. In some cases, this can be more efficient than a mutex.