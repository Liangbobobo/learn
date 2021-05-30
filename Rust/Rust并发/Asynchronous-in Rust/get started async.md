# Asynchronous Programming in Rust
async programming 用于：   
1. web server,
2. a database,  
3. an operating system,

## Async vs other concurrency models（并发模型）： 


In summary, asynchronous programming allows highly performant implementations that are suitable for low-level languages like Rust, while providing most of the ergonomic（人类工程学的） benefits of threads and coroutines.  


## Async in Rust vs other languages
1. Futures are inert（惰性） in Rust and make progress（执行） only when polled. Dropping a future stops it from making further progress.
2. Async is zero-cost in Rust. you can use async without heap allocations and dynamic dispatchwhich is great for performance! This also lets you use async in constrained environments(约束环境), such as embedded systems（嵌入式系统）.
3. No built-in runtime is provided by Rust. Instead, runtimes are provided by community maintained crates.
4. Both single- and multithreaded runtimes are available in Rust, which have different strengths and weaknesses.


## Async vs threads in Rust
The primary alternative to async in Rust is using OS threads, either directly through std::thread or indirectly through a thread pool.   
Migrating from threads to async or vice versa typically requires major refactoring work（重构）, both in terms of implementation and (if you are building a library) any exposed public interfaces. 
1. OS threads are suitable for a small number of tasks   
缺点：   
    since threads come with CPU and memory overhead.   
Spawning and switching between threads is quite expensive as even idle threads consume system resources. A thread pool library can help mitigate some of these costs, but not all.   
优点：  
threads let you reuse existing synchronous code without significant code changes—no particular programming model is required.   
In some operating systems, you can also change the priority（优先） of a thread, which is useful for drivers and other latency sensitive applications.


2. Async  
优点：  
Async provides significantly reduced CPU and memory overhead, especially for workloads with a large amount of IO-bound tasks, such as servers and databases。 All else equal, you can have orders of magnitude（数量级） more tasks than OS threads, because an async runtime uses a small amount of (expensive) threads to handle a large amount of (cheap) tasks.  
缺点：
However, async Rust results in larger binary blobs（binary large object,二进制大对象） due to the state machines generated from async functions and since each executable bundles an async runtime.


On a last note, asynchronous programming is not better than threads, but different. If you don't need async for performance reasons, threads can often be the simpler alternative.

# The State of Asynchronous Rust
Notably, Rust does not let you declare async functions in traits. Instead, you need to use workarounds to achieve the same result, which can be more verbose.

# async/.await Primer
async/.await is Rust's built-in tool for writing asynchronous functions that look like synchronous code. 

async transforms a block of code into a state machine that implements a trait called Future. 

Whereas calling a blocking function in a synchronous method would block the whole thread, blocked Futures will yield control of the thread, allowing other Futures to run.

使用方式：  
1. add some dependencies to the Cargo.toml file:  
```
[dependencies]
futures = "0.3"
```
2.  create an asynchronous function, you can use the async fn syntax:  
```
async fn do_something() { /* ... */ }

 async fn的返回值是一个Future
```

future是惰性的，必须在一个 executor上才会运行：  
```
// `block_on` blocks the current thread until the provided future has run to
// completion. Other executors provide more complex behavior, like scheduling
// multiple futures onto the same thread.
use futures::executor::block_on;

async fn hello_world() {
    println!("hello, world!");
}

fn main() {
    let future = hello_world(); // Nothing is printed
    block_on(future); // `future` is run and "hello, world!" is printed
}
```

## Inside an async fn, you can use .await 
to wait for the completion of another type that implements the Future trait, such as the output of another async fn.  

Unlike block_on, .await doesn't block the current thread, but instead asynchronously waits for the future to complete, allowing other tasks to run if the future is currently unable to make progress.

```
async fn learn_song() -> Song { /* ... */ }
async fn sing_song(song: Song) { /* ... */ }
async fn dance() { /* ... */ }

fn main() {
    let song = block_on(learn_song());
    block_on(sing_song(song));
    block_on(dance());
}

```
we're not giving the best performance possible this way—we're only ever doing one thing at once!   
Clearly we have to learn the song before we can sing it, but it's possible to dance at the same time as learning and singing the song. To do this, we can create two separate async fn which can be run concurrently:
```
async fn learn_and_sing() {
    // Wait until the song has been learned before singing it.
    // We use `.await` here rather than `block_on` to prevent blocking the
    // thread, which makes it possible to `dance` at the same time.
    let song = learn_song().await;
    sing_song(song).await;
}

async fn async_main() {
    let f1 = learn_and_sing();
    let f2 = dance();

    // `join!` is like `.await` but can wait for multiple futures concurrently.
    // If we're temporarily blocked in the `learn_and_sing` future, the `dance`
    // future will take over the current thread. If `dance` becomes blocked,
    // `learn_and_sing` can take back over. If both futures are blocked, then
    // `async_main` is blocked and will yield to the executor.
    futures::join!(f1, f2);
}

fn main() {
    block_on(async_main());
}
```

In this example, learning the song must happen before singing the song, but both learning and singing can happen at the same time as dancing. If we used block_on(learn_song()) rather than learn_song().await in learn_and_sing, the thread wouldn't be able to do anything else while learn_song was running. This would make it impossible to dance at the same time. By .await-ing the learn_song future, we allow other tasks to take over the current thread if learn_song is blocked. This makes it possible to run multiple futures to completion concurrently on the same thread.

