https://doc.rust-lang.org/std/iter/index.html

# Module std::iter
Composable external iteration.  可组合的外部迭代。    
If you’ve found yourself with a collection of some kind, and needed to perform an operation on the elements of said collection, you’ll quickly run into ‘iterators’. Iterators are heavily used in idiomatic Rust code, so it’s worth becoming familiar with them.

Before explaining more, let’s talk about how this module is structured:   
Organization：   
This module is largely organized by type:    
1. Traits are the core portion部分: these traits define what kind of iterators exist and what you can do with them. The methods of these traits are worth putting some extra study time into.   
2. Functions provide some helpful ways to create some basic iterators.   
3. Structs are often the return types of the various methods on this module’s traits. You’ll usually want to look at the method that creates the struct, rather than the struct itself. For more detail about why, see ‘Implementing Iterator’.

## Iterator
The heart and soul of this module is the Iterator trait. The core of Iterator looks like this:   
```
trait Iterator {
    type Item;
    fn next(&mut self) -> Option<Self::Item>;
}
```   
An iterator has a method, next, which when called, returns an ``` Option<Item> ```.     
next will return Some(Item) as long as there are elements, and once they’ve all been exhausted一旦用完, will return None to indicate that iteration is finished.     
Individual iterators may choose to resume恢复 iteration, and so calling next again may or may not eventually start returning Some(Item) again at some point (for example, see TryIter).    

Iterator’s full definition includes a number of other methods as well还, but they are default methods, built on top of next, and so you get them for free.    
Iterators are also composable, and it’s common to chain them together to do more complex forms of processing. See the Adapters section below for more details.

## The three forms of iteration
There are three common methods which can create iterators from a collection:   
1. iter(), which iterates over &T.    
2. iter_mut(), which iterates over &mut T.    
3. into_iter(), which iterates over T.    
Various things in the standard library may implement one or more of the three, where appropriate.

## Implementing Iterator
Creating an iterator of your own involves two steps:    
1. creating a struct to hold the iterator’s state, and then implementing Iterator for that struct.   
2. This is why there are so many structs in this module: there is one for each iterator and iterator adapter.    

Let’s make an iterator named Counter which counts from 1 to 5:   
```
// First, the struct:

/// An iterator which counts from one to five
struct Counter {
    count: usize,
}

// we want our count to start at one, so let's add a new() method to help.
// This isn't strictly necessary, but is convenient. Note that we start
// `count` at zero, we'll see why in `next()`'s implementation below.
impl Counter {
    fn new() -> Counter {
        Counter { count: 0 }
    }
}

// Then, we implement `Iterator` for our `Counter`:

impl Iterator for Counter {
    // we will be counting with usize
    type Item = usize;

    // next() is the only required method
    fn next(&mut self) -> Option<Self::Item> {
        // Increment our count. This is why we started at zero.
        self.count += 1;

        // Check to see if we've finished counting or not.
        if self.count < 6 {
            Some(self.count)
        } else {
            None
        }
    }
}

// And now we can use it!

let mut counter = Counter::new();

assert_eq!(counter.next(), Some(1));
assert_eq!(counter.next(), Some(2));
assert_eq!(counter.next(), Some(3));
assert_eq!(counter.next(), Some(4));
assert_eq!(counter.next(), Some(5));
assert_eq!(counter.next(), None);
``` 
Calling next this way gets repetitive. Rust has a construct which can call next on your iterator, until it reaches None. Let’s go over that next.   

Also note that Iterator provides a default implementation of methods such as nth and fold which call next internally. ?

## for loops and IntoIterator
Rust’s for loop syntax is actually sugar for iterators. Here’s a basic example of for:   
```
let values = vec![1, 2, 3, 4, 5];

for x in values {
    println!("{}", x);
}
```
This will print the numbers one through five, each on their own line. But you’ll notice something here: we never called anything on our vector to produce an iterator. What gives?   

There’s a trait in the standard library for converting something into an iterator: IntoIterator. This trait has one method, into_iter, which converts the thing implementing IntoIterator into an iterator. Let’s take a look at that for loop again, and what the compiler converts it into:   
```
let values = vec![1, 2, 3, 4, 5];

for x in values {
    println!("{}", x);
}
```
Rust de-sugars糖 this into:    
```
let values = vec![1, 2, 3, 4, 5];
{
    let result = match IntoIterator::into_iter(values) {
        mut iter => loop {
            let next;
            match iter.next() {
                Some(val) => next = val,
                None => break,
            };
            let x = next;
            let () = { println!("{}", x); };
        },
    };
    result
}
```
