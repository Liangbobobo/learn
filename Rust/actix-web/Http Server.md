官方文档介绍：https://actix.rs/docs/server/   
# server

# The HTTP Server
The HttpServer type ：   
is responsible for serving HTTP requests.

HttpServer accepts an application factory as a parameter, and the application factory must have Send + Sync boundaries. More about that in the multi-threading section（?）.   

To bind to a specific socket address, bind() must be used, and it may be called multiple times. To bind ssl socket, bind_openssl() or bind_rustls() should be used. To run the HTTP server, use the HttpServer::run() method.


```
use actix_web::{web, App, HttpResponse, HttpServer};

#[actix_web::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new().route("/", web::get().to(|| HttpResponse::Ok()))
    })
    .bind("127.0.0.1:8080")?
    .run()
    .await
}
```

The run() method returns an instance of the Server type.    
Methods of server type could be used for managing the HTTP server：  
pause() - Pause accepting incoming connections   
resume() - Resume accepting incoming connections   
stop() - Stop incoming connection processing, stop all workers and exit   


## how to start the HTTP server in a separate thread.   
```
use actix_web::{rt::System, web, App, HttpResponse, HttpServer}; //pub use actix_rt as rt;
use std::sync::mpsc;
use std::thread;

#[actix_web::main]
async fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let sys = System::new("http-server");

        let srv = HttpServer::new(|| {
            App::new().route("/", web::get().to(|| HttpResponse::Ok()))
        })
        .bind("127.0.0.1:8080")?
        .shutdown_timeout(60) // <- Set shutdown timeout to 60 seconds
        .run();

        let _ = tx.send(srv);
        sys.run()
    });

    let srv = rx.recv().unwrap();

    // pause accepting new connections
    srv.pause().await;
    // resume accepting new connections
    srv.resume().await;
    // stop server
    srv.stop(true).await;
}
```