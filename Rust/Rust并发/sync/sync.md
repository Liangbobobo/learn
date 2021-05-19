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