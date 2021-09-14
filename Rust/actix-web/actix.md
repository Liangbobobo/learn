https://actix.rs/docs/
#  Actix
focusses primarily on the Actix Web framework，Now, actix-web is largely unrelated to the actor framework and is built using a different system.    
For information about the actor framework called Actix，At this time, the use of actix is only required for WebSocket endpoints.       


An application developed with actix-web will expose an HTTP server contained within a native executable.使用 actix-web 开发的应用程序将公开包含在本机可执行文件中的 HTTP 服务器。     
You can either put this  behind another HTTP server like nginx or serve it up as-is    
Even in the complete absence of another HTTP server actix-web is powerful enough to provide HTTP/1 and HTTP/2 support as well as TLS (HTTPS). 

## Installing Rust


## Writing an Application
actix-web provides various primitives to build web servers and applications with Rust. It provides routing, middleware, pre-processing of requests, post-processing of responses, etc.   
All actix-web servers are built around the App instance：        
It is used for registering routes for resources and middleware.    
It also stores application state shared across all handlers within the same scope.

An application’s scope acts as a namespace for all routes, i.e. all routes for a specific application scope have the same url path prefix路径前缀.     
 



