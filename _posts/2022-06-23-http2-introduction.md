## Introduction HTTP2

### Introduction
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
