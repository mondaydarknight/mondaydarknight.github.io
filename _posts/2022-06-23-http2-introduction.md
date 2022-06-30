---
layout: post
title: Introduction HTTP/2
description: Define HTTP/2, why it was needed, what are its advantages, why could it be disadvantageous, alongside the key differences between HTTP/2 and HTTP/1.1
location: Taiwan
tags:
- HTTP
- Network
---

## Introduction HTTP2

### Summary
HTTP (Hypertext Transfer Protocol) is an application protocol that has been standard for communication between clients and servers. 
While its initial version 1.1 is still the most extensively implemented protocol but has disadvantages, thats' why HTTP/2 came to being.
List thev main differences between HTTP/1.1 and HTTP/2.

### HTTP/2
HTTP/2 began as the SPDY protocol, developed primarily at Google with the intention of reducing web page load latency by using techniques such as compression, multiplexing, and prioritization. it introduces a new binary framing layer that is not backward compatible with previous HTTP/1.x servers and clients (that's why version increment to HTTP/2).
By addressing some of the well-known performance limitations of HTTP/1.1 as follows.
- Target a 50% reduction in page load time (PLT).
- Avoid the need for any changes to content by website authors.
- Minimize deployment complexity, and avoid changes in network infrastructure.
- Develop this new protocol in partnership with the open-source community.
- Gather real performance data to (in)validate the experimental protocol.

### Binary framing layer
In HTTP/2, the binary framing layrer encodes requests/responses and cuts them up into smaller packets of information, greatly increasing the flexibility of data transfer.
As opposed to HTTP/1.1, which msut make use of multiple TCP connections to lessen the effect of HOL blocking, HTTP/2 establishes a single connection object between the two machines, there're multiple streams that each stream consist of mutiple messages of data within the connection then split into smaller units called frames.

![HTTP/2 Binary framing layer](https://user-images.githubusercontent.com/32161174/176599625-99c9a47f-1928-4029-aa85-5e378e6dc3a5.svg)


### Multiplexing
The communication channel consists of a bunch of binary-encoded frames, each tagged to a particular stream that allow the connection to interleave these frames and reassemble at the other end, the interleaved requests and respnoses can run in parallel without blocking the messages that called multiplexing.
Multiplexing resolves the [head-of-line blocking](https://en.wikipedia.org/wiki/Head-of-line_blocking) by guaranteeing that no message has to wait for another finish.


### Header compression
A common method of optimizing web applications is to use compression algorithms to reduce the size of HTTP messages that travel between the client and the server. 

**HTTP/1.1**
Programs like gzip have long been used to compress the data sent in HTTP messages, especially to describe the size of css, javascript files. However, the message is always sent as plain text and adds anywhere from 500-800 bytes of overhead per transfer, although each header is quite small, the burden of this uncompressed data weighs heavior on the connection as more requests are made. Additionally, the use of cookie can sometimes make headers much larger, increasing the need for some kind of compression.

**HTTP/2**
In HTTP/2, it uses the binary framing layer to exhibit greater control over finer detail. While compressing headers, HTTP/2 can split headers from their data, result in a `header frame` and a `data frame`, and comress via the specific program [HPACK](https://datatracker.ietf.org/doc/html/draft-ietf-httpbis-header-compression-12).
Since this algorithm can encode the header metadata using Huffman code which reduces their individual transfer size, thereby greatly decreasing its size. Additionally, HPACK can keep track of previously conveyed metadata fields and further compress them according to a dynamically altered index shared between the client and server.

![HTTP/2 server push](https://user-images.githubusercontent.com/32161174/176600729-867f65b8-86a0-42b7-b558-cdb047ac6135.png)

As one further optimziation, HPACK compression context consists of a static and dynamic table, the static table is defined in the specification and provides a list of common HTTP header fields that all connections are likely to use; the dynamic table is initally empty and is updated based on exchanged values within a partticular connection.

### Stream prioritization
Once an HTTP message can be split into many individual frames, the order in which the frames are interleaved and delivered both by the client and server becomes a critical performance consideration, to faciliate this, HTTP/2 standard allwos each stream to have an associated weight and dependency.
- Each stream may be assigned an integer weight between 1 and 256.
- Each stream may be given an explicit dependency on another stream.
The combination of stream dependencies and weights allows the client to construct and communicate a `proirtization tree` that express how it would prefer to receive responses.
The server can use this inforamtion to prioritize stream processing by controlling the allocation of CPU, memory and other resources. Allocation of bandwidth to ensure optimal delivery of high-priority responses to the clinet.
As a developer, you can set the weights in your requests based on your needs. For example, you may assign a lower priority for loading an image with high resolution after providing a thumbnail image on the web page. By providing this facility of weight assignemnt, HTTP/2 enables developers to gain better control over web page rendering. The protocol also allows the client to change dependencies and reallocate weights at runtime in response to user interaction. 

![HTTP/2 stream prioritization](https://user-images.githubusercontent.com/32161174/176600845-0fbdae74-1df5-4190-aab4-d6ec410c00f0.svg)

### Flow control
In any TCP connection between two machines, both the client and the server have a certain amount of buffer space available to hold incoming requests that have not yet been processed. These buffers offer flexibility to account for numerous or particularly large requests, in addition, to uneven speeds of downstraem and upstream connections.
There're situations that take a large amount of buffer size, for example, the server may be pushing a large amount of data at a pace that the client application is not able to cope with due to a limited buffer size or a lower bandwidth. Likewise, when a client uploads a huge image or a video to a server, the server buffer may overflow, causing some additional packets to be lost.

In order to avoid buffer overflow, a flow control mechanism must prevent the sender from overwhelming the receiver with data. HTTP/2 allows the client and the server to implement their own flow controls, rather than relying on the transport layer, the application layer communicates the available buffer space, allowing the client and server to set `receive window` on the level of the multiplexed streams. This fine-scale flow control can be modified after the initial connection via a `WINDOW_UPDATE` frame.

![HTTP/2 flow control](https://user-images.githubusercontent.com/32161174/176600497-96a19239-dfc1-49d4-8fcc-1d92fda6f7fa.png)

### Server push
Since HTTP/2 enables multiple concurrent responses to a client initial GET request, a server can send a client along with the requested HTML page, providing the resource before the client asks for it, the process is called `server push`. This way HTTP/2 connection can accomplish the same goal of resource inlining while maintaining the separation between the pushed resource and the document. This means that the client can decide to cache or decline the pushed resource separate from the main HTML document, fixing the major drawback of resource inlining.

The process begins when the server sends a `PUSH_PROMISE` frame to inform the client that it's going to push a resource. This frame includes only the header of the message, and allows the client know ahead of time which resource the server will push. If it already has the resource cached, the client can decline the push by sending a `RST_STREAM` frame in response. The `PUSH_PROMISE` frame also saves the client from sending a duplicate request to the server.

![HTTP/2 server push](https://user-images.githubusercontent.com/32161174/176600620-4592770d-ea45-4ed2-b1ea-b4f78c0fb16c.png)

### Further reading
- [HTTP/2 standard specification](https://httpwg.org/http2-spec/draft-ietf-httpbis-http2bis.html#ConnectionSpecific)
- [HTTP/1.1 vs HTT/2 What's the difference](https://www.digitalocean.com/community/tutorials/http-1-1-vs-http-2-what-s-the-difference#http-2-advantages-of-the-binary-framing-layer)
- [Introduction to HTT/2](https://web.dev/performance-http2/#header-compression)
- [Advancement from HTTP/1 to HTTP/2](https://www.wallarm.com/what/what-is-http-2-and-how-is-it-different-from-http-1)
