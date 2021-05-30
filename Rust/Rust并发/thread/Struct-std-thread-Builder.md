https://doc.rust-lang.org/std/thread/struct.Builder.html#method.spawn

# Struct std::thread::Builder
Declaration:   
pub struct Builder { /* fields omitted */ }

## 用途：  
Thread factory, which can be used in order to configure the properties of a new thread.Methods can be chained on it in order to configure it.

The two configurations available are:  
1. name: specifies an associated name for the thread  
2. stack_size: specifies the desired stack size for the thread

## 返回值
The spawn method will take ownership of the builder and create an io::Result to the thread handle with the given configuration.

The thread::spawn free function uses a Builder with default configuration and unwraps its return value.

You may want to use spawn instead of thread::spawn, when you want to recover from a failure to launch a thread, indeed the free function will panic where the Builder method will return a io::Result.   
```
use std::thread;

let builder = thread::Builder::new();

let handler = builder.spawn(|| {
    // thread code
}).unwrap();

handler.join().unwrap();
```



### Type Definition std::io::Result
``` type Result<T> = Result<T, Error>; ```
A specialized Result type for I/O operations.   
This type is broadly used across std::io for any operation which may produce an error.   
This typedef is generally used to avoid writing out io::Error directly and is otherwise a direct mapping to Result.   

While usual Rust style is to import types directly, aliases of Result often are not, to make it easier to distinguish between them. Result is generally assumed to be std::result::Result, and so users of this alias will generally use io::Result instead of shadowing the prelude’s import of std::result::Result.

Examples
```
use std::io;

fn get_string() -> io::Result<String> {
    let mut buffer = String::new();

    io::stdin().read_line(&mut buffer)?;

    Ok(buffer)
}
```



## Implementations
impl Builder   
1. pub fn new() -> Builder   
Generates the base configuration for spawning a thread, from which configuration methods can be chained.

```
use std::thread;

let builder = thread::Builder::new()
                              .name("foo".into())
                              .stack_size(32 * 1024);

let handler = builder.spawn(|| {
    // thread code
}).unwrap();

handler.join().unwrap();
```

2. pub fn name(self, name: String) -> Builder   
Names the thread-to-be. Currently the name is used for identification only in panic messages.

The name must not contain null bytes (\0).   
```
use std::thread;

let builder = thread::Builder::new()
    .name("foo".into());

let handler = builder.spawn(|| {
    assert_eq!(thread::current().name(), Some("foo"))
}).unwrap();

handler.join().unwrap();
```

3. pub fn stack_size(self, size: usize) -> Builder   
Sets the size of the stack (in bytes) for the new thread.   
The actual stack size may be greater than this value if the platform specifies a minimal stack size.   
```
use std::thread;

let builder = thread::Builder::new().stack_size(32 * 1024);
```
4.
``` 
pub fn spawn<F, T>(self, f: F) -> Result<JoinHandle<T>> where
    F: FnOnce() -> T,
    F: Send + 'static,
    T: Send + 'static, 

```
Spawns a new thread by taking ownership of the Builder, and returns an io::Result to its JoinHandle.  

The spawned thread may outlive the caller (unless the caller thread is the main thread; the whole process is terminated when the main thread finishes). The join handle can be used to block on termination of the child thread, including recovering its panics.

Errors:   
Unlike the spawn free function, this method yields an io::Result to capture any failure to create the thread at the OS level.

Panics:   
Panics if a thread name was set and it contained null bytes.

```
use std::thread;

let builder = thread::Builder::new();

let handler = builder.spawn(|| {
    // thread code
}).unwrap();

handler.join().unwrap();
```


## Auto Trait Implementations
impl RefUnwindSafe for Builder   
impl Send for Builder   
impl Sync for Builder  
impl Unpin for Builder  
impl UnwindSafe for Builder  


## Blanket Implementations