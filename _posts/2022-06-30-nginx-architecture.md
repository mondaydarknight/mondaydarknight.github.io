---
layout: post
title: Nginx architecutre
description: NGINX is a powerful web server and uses a non-threaded, event-driven architecture that enables it to outperform Apache if configured correctly. It can also do other important things, such as load balancing, HTTP caching, or be used as a reverse proxy. Dig dive Nginx web server architecture.
location: Taiwan
tags:
- HTTP
- Network
---

## Nginx architecture

### Previously
Apache, has its roots in the beginning of the 1990s, it was architecutred to matched the existing hardware and operating systems. By the beginning of 2000s, it was obvious that the standalone web server model could not be easily replicated to satisfy the needs of growing web services. Although Apache provide a solid foundation for future development, it was architecture to spawn a copy of itself for each new connection, which was not suitable for nonlinear scalability of a website.
However, nothing comes without a price and the downside to having such a rich and universal combination of tools in a single piece of software is less scalability because of increased CPU and memory usage per connection.
In order to solve C10K problem  by Danial Kegel, Nginx was written with an event-based architecture in mind - one which is much more suitable for nonlinear scalability in both the number of simultaneous connections and requests per second. It doesn’t follow Apache style of spawning new processes or threads for each web page request, that causes that event as load increases, memory and CPU usage remain manageable.
Handling high concurrency with high performance and efficiency has always been the key benefit of deploying Nginx. However, there’re now even more interesting benefits.

Over the course of the development, Nginx has added the following functionalities.
- Integrated with applications through the use of FastCGI, uswgi, SCGI protocols
- With distributed memory object caching systems like memcached
- Reverse proxy with load balling and caching.

| |	Apache |	Nginx |
|-|--------|--------|
| Overview | Open source cross platform web server | Used as reverse proxy, load balancer, mail proxy for HTTP caching |
| Architecture | Process driven approach creates a new thread for each request, multi-threaded | Event-driven approach handles multiple request within one thread, lightweight structure and much faster architecture |
| Performance | Serves static content using the file-based method, which is an overhead for the interpreter. <br>Process dynamic content within the server | Static content can be served in a straight-forward manner and the interpreter will only be contacted when needed <br> It doesn’t process dynamic content and passes to an external processor for execution |
| OS support | Supports all Unix OS including Linus and BSD, it fully supports MS - Windows | Supports almost all Unix OS and Windows partially |
| Distributed/centralized configuration | Allows additional configuration on a per-directory basis via .htaccess files | Doesn’t allow additional configuration |
| Security | Great security |	Better security with the smaller codebase |
| Flexibility | Supports customization of web server through dynamic modules | Not flexible enough to support dynamic modules and loading |
| Feature | modules	60 official dynamically loadable modules that can be turned on/off | 3rd party core modules |

### Overview of Nginx architecture
In contrast to the traditional process/thread architecture, involve handling each connection with a separate process or thread, and blocking on network or input/output operations. Depending on the application, it can be very inefficient in terms of memory and CPU consumption. Spawning a separate process or thread retries preparation of a new runtime environment, including allocation of heap and stack memory, and creation of a new execution context. Additional CPU time is also spent creating these items, which can eventually lead to poor performance due to thread thrashing on excessive context switching.
- modular
- event-driven
- asynchronous
- single-threaded
- non-blocking architecture

Nginx uses multiplexing and event notifications heavily, and dedicated specific tasks to separate processes. Connections are processed in a highly efficient run loop in limited number of single-threaded processes called worker. Within a worker Nginx can handle many thousands of concurrent connections and requests per second. 
**Nginx can handle thousands of connection (requests) within one processing thread.**

### Core structure
Nginx worker code includes the core and functional modules, the core is responsible for maintaining a tight run-loop and executing appropriate sections of modules code on each stage of request processing. Modules constitute most of the presentation and application layer functionality that read from and write to the network and storage, transform content, do outbound filtering apply server-side include actions and pass the requests to the upstream servers when proxying is activated.
NGINX modular architecture generally allows developers coextend the set of web server features without modifying Nginx core, modules come in slightly different incarnations, namely core modules, event modules, phase handlers, protocols, variable handlers, filters, upstreams and load balancers. However, Nginx doesn’t support `dynamically loaded modules` that are compiled along with the core at build stage.
While handling a variety of actions associated with accepting, processing managing network connections and content retrieval, Nginx uses event notification mechanism and a number of disk I/O performance enhancements like kqueue, epoll and event ports.

Nginx doesn’t spawn a process or thread for every connection, instead, worker processes accept new requests from a shared listen docker and execute a highly efficient run-loop inside each worker to process thousands of connections per worker. 
Upon startup, an initial set of listening sockets is created, workers then continuously accept, read from and write to the sockets while processing HTTP requests and responses.

The run-loop is the most complicated part of the nginx code, it includes comprehensive inner calls and relies heavily on the idea of asynchronous task handing, asynchronous operations are implemented through modularity, event notifications, extensive use of callback functions and fine-tuned timers.
Because Nginx cannot fork a process or thread per connection, memory usage is very conservative and extremely efficient in the vast majority of cases. Nginx `conserves CPU cycles` as well be cause there’s no ongoing create-destroy pattern for processes or threads.

Nginx spawns several workers to handle connections, it scales well across multiple cores, generally, a separate worker per core allows full utilization of multicore architectures, and prevents thread thrashing and lock-ups.
There’s no resource starvation and resource controlling mechanisms are isolated within single-threaded worker processes. This model also allows more scalability across physical storage devices, facilities more disk utilization and avoids blocking on disk I/O.

The master process is responsible for the following tasks.
- Reading and validating configuration
- Creating, binding and closing sockets
- Starting, terminating and maintaining the configured number worker processes
- Reconfiguring without service interruption 
- Controlling non-stop binary upgrades (starting new binary and rolling back if necessary)
- Re-opening log files
- Compiling embedded perl scripts

#### Cache loader
Cache loader process is responsible for checking the on-disk cache items and populating Nginx’s in-memory database with cache metadata, essentially, the cache loader prepares Nginx instances to work with files already stored on disk in a specially allocated directory structure. It traverses the directories her cache content metadata and update the revenant entries in shared memory and then exits when everything is clean and ready for use.

#### Cache Manager
Cache manager is mostly responsible for cache expiration and invalidation, it stays in memory during normal Nginx operation and it restarted by the master process in the case of failure.

Nginx modules contains core and functional modules, such as http and mail that provide an additional level of abstraction between the core and lower-level components, in these modules, the handling of sequence of events associated with a respective application layer protocol like HTTP, SMTP or IMAP.
The functional modules can be divided into event modules, phase handlers, output filter variable handlers, protocols, upstreams and load balancers.
Event modules provide a particular OS-dependent event notification mechanism like `kqueue` or `epoll`.

A typical HTTP request processing cycle looks like the following
1. Client sends HTTP request
2. Nginx core chooses the appropriate phase handler based on the configured location matching the request
3. If configured to do so, a load balancer picks an upstream server for proxying
4. Phase handlers does its job and passes each output buffer to the first filter
5. First filter passes the output to the second filter
6. Second filter passes the output to third and so on
7. Final response is sent to the client

Inside the worker, the sequence of actions leading to the run-loop where response is generated looks like the following
1. Begin ngx_worker_process_cycle
2. Process events with OS specific mechanisms (such as poll or kqueue)
3. Accept events and dispatch the revenant actions
4. Process / proxy request header and body
5. Generate response content (header, body) and stream it to the client
6. Finalize the request
7. Re-initialize the timer and events

When the HTTP request header is read, Nginx does a lookup of Thea associated virtual server configuration if the virtual is found, the request goes through six phases
1. Server rewrite phase 
2. Location phase 
3. Location rewrite phase
4. Access control phase
5. try_files phase 
6. Log phase
7. Distribute the load across different upstream servers 
8. Health checks

There’re also a couple of tother interesting modules which provide an additional set of variable for use in the configuration file, while the variables. in Nginx are created and update across different modules, there’re two modules that are entirely dedicated to variables geo and map.
- geo: a module is used to facilitate tracking of clients based on their IP addressses.
- map: allows for the creation of variables from other variables, essentially providing the ability to do flexible mapping s of hostnames and other run-time variables.

Memory allocation mechanism implemented inside a single Nginx, inspired by Apache, a high-level description of Nginx memory management would be the following. 
For each connection, the necessary memory buffers are dynamically allocated, linked and used for storing and manipulating the header and body of the request and the response., freed upon connection release.

> Nginx tries to avoid copying data in memory as much as possible and most of the data is passed along by pointer values, not by cling memcpy.

When the response is generated by a module, the retrieved content is put in a memory buffer which is then added to a buffer chain link, subsequent processing works with this buffer china link as well. Buffer chains are quite complicated in Nginx because there’re several processing scenarios which differ depending on module type.

> Note that memory buffer allocated for the entire life of a connection, thus for long-lived connections some extra memory is kept, at the same time , on an idle keep alive connection , Nginx spends just 550 bytes of memory, it can be optimized for future of releases of Nginx would be reuse and share memory buffers for long-lived connections.

The task of managing allocation is done by Nginx pool allocator, shared memory areas are used to accept mutex, cache metadata,  SSL cache and the information associated with bandwidth policing and management (limits).
In order to organize complex data structure, Nginx also provides a red-black tree implementation that are sued to keep cache metadata in shared memory, track non-regex location definition and for a couple of other tasks.

### Further reading
- [Nginx architecture concept](https://www.aosabook.org/en/nginx.html)

