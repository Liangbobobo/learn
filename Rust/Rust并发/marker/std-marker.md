# Module std::marker
Primitive traits and types representing basic properties of types.    
Rust types can be classified in various useful ways according to their intrinsic properties. These classifications are represented as traits.

# Macros
Copy	Derive macro generating an impl of the trait Copy.  

# Structs
PhantomData ： 	   
Zero-sized type used to mark things that “act like” they own a T.

PhantomPinned：   	
A marker type which does not implement Unpin.   



# Traits
Copy--Types whose values can be duplicated simply by copying bits.  
Send--Types that can be transferred across thread boundaries.  
Sized--Types with a constant size known at compile time.   
Sync---Types for which it is safe to share references between threads.   
Unpin---Types that can be safely moved after being pinned.   


## Trait std::marker::Send   
Declaration:   
```
pub unsafe auto trait Send { }
```

Types that can be transferred转移 across thread boundaries.      
This trait is automatically implemented when the compiler determines it’s appropriate.   
An example of a non-Send type is the reference-counting pointer rc::Rc. If two threads attempt to clone Rcs that point to the same reference-counted value, they might try to update the reference count at the same time, which is undefined behavior because Rc doesn’t use atomic operations. Its cousin sync::Arc does use atomic operations (incurring some overhead) and thus is Send.

## Trait std::marker::Sync
Declaration:   
```
 #[lang = "sync"]
pub unsafe auto trait Sync { } 
```
Types for which it is safe to share references between threads.   
This trait is automatically implemented when the compiler determines it’s appropriate.   
The precise definition is: a type T is Sync if and only if &T is Send. 准确的定义是，当且仅当&T是Sync时，类型T才是Sync   
In other words, if there is no possibility of undefined behavior (including data races) when passing &T references between threads.   


As one would expect正如人们所期望的, primitive types like u8 and f64 are all Sync, and so are simple aggregate types containing them, like tuples, structs and enums.   
More examples of basic Sync types include “immutable” types like &T, and those with simple inherited mutability, such as ```Box<T>, Vec<T>``` and most other collection types. (Generic parameters need to be Sync for their container to be Sync.)

A somewhat surprising consequence of the definition is that &mut T is Sync (if T is Sync) even though it seems like that might provide unsynchronized mutation.     
The trick诀窍 is that a mutable reference behind a shared reference (that is, & &mut T) becomes read-only, as if it were a & &T. Hence there is no risk of a data race.   

### not Sync
Types that are not Sync are those that have “interior mutability” in a non-thread-safe form，such as Cell and RefCell.    
These types allow for mutation of their contents even through an immutable, shared reference.    
For example the set method on ```Cell<T>``` takes &self, so it requires only a shared reference ```&Cell<T>```. The method performs no synchronization, thus Cell cannot be Sync.


Another example of a non-Sync type is the reference-counting pointer Rc. Given any reference``` &Rc<T>```, you can clone a new``` Rc<T>,``` modifying the reference counts in a non-atomic way.

### 线程内部可变性
For cases when one does need thread-safe interior mutability, Rust provides atomic data types, as well as explicit locking via sync::Mutex and sync::RwLock.    
These types ensure that any mutation cannot cause data races, hence the types are Sync. Likewise, sync::Arc provides a thread-safe analogue类似 of Rc.   
Any types with interior mutability must also use the cell::UnsafeCell wrapper around the value(s) which can be mutated through a shared reference. Failing to doing this is undefined behavior. For example, transmute-ing from &T to &mut T is invalid.

# Send and Sync：   
Not everything obeys inherited mutability, though.    
Some types allow you to have multiple aliases of a location in memory while mutating it.    
Unless these types use synchronization to manage this access, they are absolutely not thread-safe. Rust captures this through the Send and Sync traits.    
1. A type is Send if it is safe to send it to another thread.
2. A type is Sync if it is safe to share between threads (T is Sync if and only if &T is Send).   

Send and Sync are fundamental to Rust's concurrency story. As such, a substantial重大的 amount of special tooling exists to make them work right.     
First and foremost首要, they're unsafe traits. ：   
This means that they are unsafe to implement, and other unsafe code can assume that they are correctly implemented.     
Since they're marker traits (they have no associated items like methods), correctly implemented simply means that they have the intrinsic固有的 properties an implementor should have.    


Send and Sync are also automatically derived traits.    
This means that, unlike every other trait, if a type is composed entirely of Send or Sync types, then it is Send or Sync.    
Almost all primitives are Send and Sync, and as a consequence pretty much all types you'll ever interact with are Send and Sync.   



Major exceptions include:   
1. raw pointers are neither Send nor Sync (because they have no safety guards).   
2. UnsafeCell isn't Sync (and therefore Cell and RefCell aren't).   
3. Rc isn't Send or Sync (because the refcount is shared and unsynchronized).   

Rc and UnsafeCell are very fundamentally not thread-safe:    
they enable unsynchronized shared mutable state.   
However raw pointers are, strictly speaking, marked as thread-unsafe as more of a lint.    
Doing anything useful with a raw pointer requires dereferencing it, which is already unsafe. In that sense, one could argue that it would be "fine" for them to be marked as thread safe.

However it's important that they aren't thread-safe to prevent types that contain them from being automatically marked as thread-safe.       
These types have non-trivial untracked ownership, and it's unlikely that their author was necessarily thinking hard about thread safety. In the case of Rc, we have a nice example of a type that contains a *mut that is definitely not thread-safe.   

Types that aren't automatically derived can simply implement them if desired:   
```

struct MyBox(*mut u8);

unsafe impl Send for MyBox {}
unsafe impl Sync for MyBox {}
```


In the incredibly rare case that a type is inappropriately automatically derived to be Send or Sync, then one can also unimplement Send and Sync:   
```
#![feature(negative_impls)]

// I have some magic semantics for some synchronization primitive!
struct SpecialThreadToken(u8);

impl !Send for SpecialThreadToken {}
impl !Sync for SpecialThreadToken {}
```

Note that in and of itself it is impossible to incorrectly derive Send and Sync. Only types that are ascribed special meaning by other unsafe code can possibly cause trouble by being incorrectly Send or Sync.   

Most uses of raw pointers should be encapsulated behind a sufficient abstraction that Send and Sync can be derived. For instance all of Rust's standard collections are Send and Sync (when they contain Send and Sync types) in spite of their pervasive use of raw pointers to manage allocations and complex ownership. Similarly, most iterators into these collections are Send and Sync because they largely behave like an & or &mut into the collection.