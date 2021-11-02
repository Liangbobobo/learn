## Wildcard pattern: _
The wildcard pattern (an underscore symbol) matches any value.   
It is used to ignore values when they don't matter.   
Inside other patterns it matches a single data field (as opposed to the .. which matches the remaining fields). Unlike identifier patterns, it does not copy, move or borrow the value it matches.   

The wildcard pattern is always irrefutable.   

```
let (a, _) = (10, x);   // the x is always matched by _

// ignore a function/closure param
let real_part = |a: f64, _: f64| { a };

// ignore a field from a struct
let RGBA{r: red, g: green, b: blue, a: _} = color;

// accept any Some, with any value
if let Some(_) = x {}
```

