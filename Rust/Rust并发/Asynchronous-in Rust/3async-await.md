# async/.await
async/.await are special pieces of Rust syntax that make it possible to yield control of the current thread rather than blocking, allowing other code to make progress while waiting on an operation to complete.

There are two main ways to use async:async fn and async blocks.     
Each returns a value that implements the Future trait      
```
// `foo()` returns a type that implements `Future<Output = u8>`.
// `foo().await` will result in a value of type `u8`.
async fn foo() -> u8 { 5 }

fn bar() -> impl Future<Output = u8> {
    // This `async` block results in a type that implements
    // `Future<Output = u8>`.
    async {
        let x: u8 = foo().await;
        x + 5
    }
}
async bodies and other futures are lazy: 
they do nothing until they are run. 

The most common way to run a Future is to .await it. 
When .await is called on a Future, it will attempt to run it to completion. 
 If the Future is blocked, it will yield control of the current thread. 
 When more progress can be made, the Future will be picked up by the executor 
 and will resume running, allowing the .await to resolve.
```

## async Lifetimes
Unlike traditional functions, async fns which take references or other non-'static arguments return a Future which is bounded by the lifetime of the arguments:   
```
// This function:
async fn foo(x: &u8) -> u8 { *x }

// Is equivalent to this function:
fn foo_expanded<'a>(x: &'a u8) -> impl Future<Output = u8> + 'a {
    async move { *x }
}
```
This means that the future returned from an async fn must be .awaited while its non-'static arguments are still valid.    
 In the common case of .awaiting the future immediately after calling the function (as in foo(&x).await) this is not an issue. However, if storing the future or sending it over to another task or thread, this may be an issue.

 One common workaround for turning an async fn with references-as-arguments into a 'static future is to bundle the arguments with the call to the async fn inside an async block:   
 ```
fn bad() -> impl Future<Output = u8> {
    let x = 5;
    borrow_x(&x) // ERROR: `x` does not live long enough
}

fn good() -> impl Future<Output = u8> {
    async {
        let x = 5;
        borrow_x(&x).await
    }
}

By moving the argument into the async block, 
we extend its lifetime to match that of the Future returned from the call to good.
 ```

## async move
async blocks and closures allow the move keyword, much like normal closures.    
An async move block will take ownership of the variables it references, allowing it to outlive the current scope,   
but giving up the ability to share those variables with other code: