https://doc.rust-lang.org/nomicon/subtyping.html
# Subtyping and Variance
Subtyping is a relationship between types that allows statically typed languages to be a bit more flexible and permissive.子类型是类型之间的一种关系，它允许静态类型语言更加灵活和宽容。   

```
trait Animal {
    fn snuggle(&self);
    fn eat(&mut self);
}

trait Cat: Animal {
    fn meow(&self);
}

trait Dog: Animal {
    fn bark(&self);
}

fn love(pet: Animal) {
    pet.snuggle();
}

let mr_snuggles: Cat = ...;
love(mr_snuggles);         // ERROR: expected Animal, found Cat
```
This is exactly the problem that subtyping is intended to fix. Because Cats are just Animals and more, we say Cat is a subtype of Animal (because Cats are a subset of all the Animals). Equivalently, we say that Animal is a supertype of Cat.     
###  With subtypes, we can tweak our overly strict static type system with a simple rule: anywhere a value of type T is expected, we will also accept values that are subtypes of T.Or more concretely: anywhere an Animal is expected, a Cat or Dog will also work.




