# The Perils危险 Of Ownership Based Resource Management (OBRM)
OBRM (AKA RAII: Resource Acquisition Is Initialization) is something you'll interact with a lot in Rust. Especially if you use the standard library.   

Roughly speaking the pattern is as follows: to acquire a resource, you create an object that manages it. To release the resource, you simply destroy the object, and it cleans up the resource for you. The most common "resource" this pattern manages is simply memory. Box, Rc, and basically everything in std::collections is a convenience to enable correctly managing memory. This is particularly important in Rust because we have no pervasive GC to rely on for memory management. 

Which is the point, really: Rust is about control. However we are not limited to just memory. Pretty much every other system resource like a thread, file, or socket is exposed through this kind of API.

# Constructors
There is exactly one way to create an instance of a user-defined type: name it, and initialize all its fields at once  
Every other way you make an instance of a type is just calling a totally vanilla普通 function that does some stuff and eventually bottoms out降级为 to The One True Constructor.