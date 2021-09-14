https://doc.rust-lang.org/reference/items/traits.html#object-safety
# Trails
A trait describes an abstract interface that types can implement. This interface consists of associated items, which come in three varieties:   
functions  
types  
constants

All traits define an implicit type parameter Self that refers to "the type that is implementing this interface".    
Traits may also contain additional type parameters. These type parameters, including Self, may be constrained受约束 by other traits and so forth as usual和往常一样.   
