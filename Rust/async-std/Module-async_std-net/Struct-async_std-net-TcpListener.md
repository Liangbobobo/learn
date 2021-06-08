# Struct async_std::net::TcpListener
``` pub struct TcpListener { /* fields omitted */ } ```
A TCP socket server, listening for connections.   

After creating a TcpListener by binding it to a socket address, it listens for incoming TCP connections. These can be accepted by awaiting elements from the async stream of incoming connections.

The socket will be closed when the value is dropped.        
The Transmission Control Protocol is specified in    
https://datatracker.ietf.org/doc/html/rfc793  

This type is an async version of std::net::TcpListener.   

Examples:  
```

use async_std::io;
use async_std::net::TcpListener;
use async_std::prelude::*;

let listener = TcpListener::bind("127.0.0.1:8080").await?;
let mut incoming = listener.incoming();

while let Some(stream) = incoming.next().await {
    let stream = stream?;
    let (reader, writer) = &mut (&stream, &stream);
    io::copy(reader, writer).await?;
}
```