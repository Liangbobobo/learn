https://doc.rust-lang.org/std/cell/index.html

# Module std::cell--Shareable mutable containers.共享可变容器
Rust memory safety is based on this rule: Given an object T, it is only possible to have one of the following:    
1. Having several immutable references (&T) to the object (also known as aliasing).      
2. Having one mutable reference (&mut T) to the object (also known as mutability).    
This is enforced by the Rust compiler. However, there are situations where this rule is not flexible enough. Sometimes it is required to have multiple references to an object and yet mutate it.   


Shareable mutable containers exist to permit mutability in a controlled manner, even in the presence of aliasing.    
Both ```Cell<T>``` and ```RefCell<T>``` allow doing this in a single-threaded way.    
However, neither ```Cell<T> ```nor ```RefCell<T>``` are thread safe (they do not implement Sync).    
If you need to do aliasing and mutation between multiple threads it is possible to use ```Mutex<T>```, ```RwLock<T> ```or atomic types.

Values of the ```Cell<T> and RefCell<T> ```types may be mutated through shared references (i.e. the common &T type), whereas most Rust types can only be mutated through unique (&mut T) references.    
We say that ```Cell<T> and RefCell<T> ```provide ‘interior mutability’, in contrast with typical Rust types that exhibit ‘inherited mutability’. 

Cell types come in two flavors:   
```Cell<T> and RefCell<T>. ```
```Cell<T>``` implements interior mutability by moving values in and out of the```Cell<T>. ```    
To use references instead of values, one must use the ```RefCell<T> ```type, acquiring a write lock before mutating. ```Cell<T>``` provides methods to retrieve and change the current interior value:    
1. For types that implement Copy, the get method retrieves the current interior value.   
2. For types that implement Default, the take method replaces the current interior value with Default::default() and returns the replaced value.   
3. For all types, the replace method replaces the current interior value and returns the replaced value and the into_inner method consumes the ```Cell<T> ```and returns the interior value. Additionally, the set method replaces the interior value, dropping the replaced value.   


```RefCell<T> ```uses Rust’s lifetimes to implement ‘dynamic borrowing’, a process whereby one can claim temporary暂时, exclusive单独, mutable access to the inner value.    
Borrows for ```RefCell<T>s``` are tracked ‘at runtime’, unlike Rust’s native reference types which are entirely tracked statically, at compile time.     
Because ```RefCell<T>``` borrows are dynamic it is possible to attempt to borrow a value that is already mutably borrowed; when this happens it results in thread panic.  

# When to choose interior mutability
The more common inherited mutability, where one must have unique access to mutate a value, is one of the key language elements that enables Rust to reason strongly about pointer aliasing, statically preventing crash bugs. 