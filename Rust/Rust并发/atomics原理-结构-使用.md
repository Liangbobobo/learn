# Atomics


## Compiler Reordering
Compilers fundamentally(根本上) want to be able to do all sorts of（各种种类） complicated transformations（复杂的改变） to reduce data dependencies and eliminate（排除） dead code. In particular, they may radically（根本上） change the actual order of events, or make events never occur! If we write something like：   
```
x = 1;
y = 3;
x = 2;
```
The compiler may conclude that it would be best if your program did  
```
x = 2;
y = 3;
```
This has inverted the order of events and completely eliminated one event.From a single-threaded perspective this is completely unobservable: after all the statements have executed we are in exactly the same state.    
But if our program is multi-threaded, we may have been relying on x to actually be assigned(分配) to 1 before y was assigned. 

We would like the compiler to be able to make these kinds of optimizations, because they can seriously improve performance. On the other hand, we'd also like to be able to depend on our program doing the thing we said.


## Hardware Reordering
On the other hand, even if the compiler totally understood what we wanted and respected our wishes, our hardware might instead get us in trouble.    
Trouble comes from CPUs in the form of memory hierarchies（层次结构）. 

There is indeed a global shared memory space somewhere in your hardware, but from the perspective of each CPU core it is so very far away and so very slow.    
Each CPU would rather 更乐于 work with its local cache of the data and only go through all the anguish of talking to shared memory only when it doesn't actually have that memory in cache.   
After all, that's the whole point of the cache, right? If every read from the cache had to run back to shared memory to double check that it hadn't changed, what would the point be?   

The end result is that the hardware doesn't guarantee that events that occur in some order on one thread, occur in the same order有些时候 on another thread. To guarantee this, we must issue special instructions to the CPU telling it to be a bit less smart.


For instance, say we convince the compiler to emit this logic:  
```
initial state: x = 0, y = 1

THREAD 1        THREAD2
y = 3;          if x == 1 {
x = 1;              y *= 2;
                         }
```
Ideally this program has 2 possible final states:   

    y = 3: (thread 2 did the check before thread 1 completed)
    y = 6: (thread 2 did the check after thread 1 completed)


However there's a third potential state that the hardware enables:   
y = 2: (thread 2 saw x = 1, but not y = 3, and then overwrote y = 3)

It's worth noting that different kinds of CPU provide different guarantees.    
It is common to separate hardware into two categories: strongly-ordered and weakly-ordered. Most notably x86/64 provides strong ordering guarantees, while ARM provides weak ordering guarantees. This has two consequences for concurrent programming:   
1. Asking for stronger guarantees on strongly-ordered hardware may be cheap or even free because they already provide strong guarantees unconditionally无条件的. Weaker guarantees may only yield performance wins on weakly-ordered hardware.   
2. Asking for guarantees that are too weak on strongly-ordered hardware is more likely to happen to work, even though your program is strictly incorrect. If possible, concurrent algorithms should be tested on weakly-ordered hardware.

## Data Accesses
The C++ memory model attempts to bridge the gap by allowing us to talk about the causality因果关系 of our program.    
Generally, this is by establishing a happens before relationship between parts of the program and the threads that are running them. This gives the hardware and compiler room to optimize the program more aggressively where a strict happens-before relationship isn't established, but forces them to be more careful where one is established. The way we communicate these relationships are through data accesses and atomic accesses.


Data accesses are the bread-and-butter of the programming world. They are fundamentally unsynchronized and compilers are free to aggressively optimize them.In particular, data accesses are free to be reordered by the compiler on the assumption that the program is single-threaded. The hardware is also free to propagate the changes made in data accesses to other threads as lazily and inconsistently不一致 as it wants. Most critically, data accesses are how data races happen.Data accesses are very friendly to the hardware and compiler, but as we've seen they offer awful semantics to try to write synchronized code with. Actually, that's too weak.

It is literally impossible to write correct synchronized code using only data accesses.   
Atomic accesses are how we tell the hardware and compiler that our program is multi-threaded.    
Each atomic access can be marked with an ordering that specifies what kind of relationship it establishes with other accesses.    

 In practice, this boils down 归结起来是 to telling the compiler and hardware certain things they can't do. For the compiler, this largely revolves围绕 around re-ordering of instructions指示. For the hardware, this largely revolves around how writes are propagated to other threads. The set of orderings Rust exposes are:  
 1. Sequentially Consistent保持 (SeqCst)  顺序一致
 2. Release
 3. Acquire
 4. Relaxed

 ### Sequentially Consistent
 Intuitively直观上, a sequentially consistent operation cannot be reordered:    
 all accesses on one thread that happen before and after a SeqCst access stay before and after it.   
 A data-race-free program that uses only sequentially consistent atomics and data accesses has the very nice property that there is a single global execution of the program's instructions that all threads agree on.   
 This execution is also particularly nice to reason about: it's just an interleaving交织 of each thread's线程的 individual executions.   
 This does not hold if you start using the weaker atomic orderings.如果您开始使用较弱的原子排序，则不会保持。   

 The relative 相对的developer-friendliness of sequential consistency doesn't come for free. Even on strongly-ordered platforms sequential consistency involves涉及 emitting memory fences 发送内存栅栏.  

 In practice, sequential consistency is rarely necessary for program correctness. However sequential consistency is definitely确实 the right choice if you're not confident about the other memory orders. Having your program run a bit slower than it needs to is certainly better than it running incorrectly! It's also mechanically trivial 机械琐碎 to downgrade atomic降级原子操作 operations to have a weaker consistency later on. Just change SeqCst to Relaxed and you're done! Of course, proving that this transformation is correct is a whole other matter.


### Acquire-Release
Acquire and Release are largely intended to be paired. 获得和释放在很大程度上打算配对  
Their names hint暗示 at their use case: they're perfectly suited for acquiring and releasing locks, and ensuring that critical sections don't overlap关键部分不交叠.

Intuitively, an acquire access ensures that every access after it stays after it.   
However operations that occur before an acquire are free to be reordered to occur after it但是，在获取之前发生的操作可以自由地重新排序以在它之后发生   
 Similarly, a release access ensures that every access before it stays before it. However operations that occur after a release are free to be reordered to occur before it.同样，释放访问可确保在其之前留下的每次访问。但是，释放后发生的操作可以自由地重新排序以在其之前发生。  

 When thread A releases a location in memory and then thread B subsequently acquires the same location in memory, causality is established. Every write (including non-atomic and relaxed atomic writes) that happened before A's release will be observed by B after its acquisition. However no causality is established with any other threads. Similarly, no causality is established if A and B access different locations in memory.

 Basic use of release-acquire is therefore simple: you acquire a location of memory to begin the critical section, and then release that location to end it. For instance, a simple spinlock might look like:  
 ```
use std::sync::Arc;
use std::sync::atomic::{AtomicBool, Ordering};
use std::thread;

fn main() {
    let lock = Arc::new(AtomicBool::new(false)); // value answers "am I locked?"

    // ... distribute lock to threads somehow ...

    // Try to acquire the lock by setting it to true
    while lock.compare_and_swap(false, true, Ordering::Acquire) { }
    // broke out of the loop, so we successfully acquired the lock!

    // ... scary data accesses ...

    // ok we're done, release the lock
    lock.store(false, Ordering::Release);
}
 ```

 On strongly-ordered platforms most accesses have release or acquire semantics, making release and acquire often totally free. This is not the case on weakly-ordered platforms.

 ## Relaxed
 Relaxed accesses are the absolute weakest.They can be freely re-ordered and provide no happens-before relationship.    
 Still, relaxed operations are still atomic. That is, they don't count as data accesses and any read-modify-write operations done to them occur atomically.    
 Relaxed operations are appropriate for things that you definitely want to happen, but don't particularly otherwise care about. For instance, incrementing a counter can be safely done by multiple threads using a relaxed fetch_add if you're not using the counter to synchronize any other accesses.

 There's rarely a benefit in making an operation relaxed on strongly-ordered platforms, since they usually provide release-acquire semantics anyway. However relaxed operations can be cheaper on weakly-ordered platforms.   





 # Module std::sync::atomic
 ## Atomic types
 Atomic types provide primitive原始的 shared-memory communication between threads, and are the building blocks of other concurrent types.   
 including：    
 THe integer type which can be safely shared between threads.   
 > AtomicI8、AtomicI16、AtomicI32、AtomicI64、AtomicIsize、AtomicU8、AtomicU16、AtomicU32、AtomicU64
 	
A boolean type which can be safely shared between threads.:   
AtomicBool

A raw pointer type which can be safely shared between threads.:   
AtomicPtr

## Each method takes an Ordering which represents the strength of the memory barrier for that operation.
Enums:   
Ordering	  Atomic memory orderings

Atomic variables are safe to share between threads (they implement Sync) but they do not themselves provide the mechanism机制 for sharing and follow the threading model of Rust线程模型.    
The most common way to share an atomic variable is to put it into an Arc (an atomically-reference-counted shared pointer).

Atomic types may be stored in static variables, initialized using the constant initializers like AtomicBool::new. Atomic statics are often used for lazy global initialization.  






# Atomics Struct std::sync::Arc
A thread-safe reference-counting pointer. ‘Arc’ stands for ‘Atomically Reference Counted’.   
The type ```Arc<T>``` provides shared ownership of a value of type T, allocated in the heap.    
Invoking clone on Arc produces a new Arc instance, which points to the same allocation on the heap as the source Arc, while increasing a reference count.   
When the last Arc pointer to a given allocation is destroyed, the value stored in that allocation (often referred to as “inner value”) is also dropped.   

Shared references in Rust disallow mutation by default, and Arc is no exception:    
you cannot generally obtain a mutable reference to something inside an Arc通常不能在arc内获得可变的引用. If you need to mutate through an Arc, use Mutex, RwLock, or one of the Atomic types.

## Thread Safety
Unlike ```Rc<T>, Arc<T> ```uses atomic operations for its reference counting.   
This means that it is thread-safe.   
The disadvantage is that atomic operations are more expensive than ordinary memory accesses.   

```Arc<T>``` will implement Send and Sync as long as只要 the T implements Send and Sync.   
