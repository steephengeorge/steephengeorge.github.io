---
layout: inner
position: center
title: QUIC- A brief history and Adavantages
date: 2023-11-01 14:15:00
categories: quic 
tags: quic Protocols
lead_text: "QUIC is a new protocol designed for web communication, featuring built-in security and high performance. Initially developed by Google to address challenges associated with high-volume, high-throughput TCP traffic, QUIC stands out from prior existing protocols."
project_link: 'http://steephengeorge.github.io/quic/2023/11/01/QUIC-intro-adv-history.html'
---

# QUIC: A brief history and Adavantages 

QUIC is a new protocol designed for web communication, featuring
built-in security and high performance. Initially developed by Google to
address challenges associated with high-volume, high-throughput TCP
traffic, QUIC stands out from prior existing protocols. To understand
its distinctions, let us delve into a bit of history.

### History of Network Protocols

As part of building computer networks, internet scientists invented
packet switching as the foundational method. Packet switching can be
simply defined as follows:

> *\"The routing and transferring of data by means of addressed packets
> so that a channel is occupied during the transmission of the packet
> only, and upon completion of the transmission, the channel is made
> available for the transfer of other traffic.\"*

The experimental packet switching network development identified the
need for higher-level protocols and led to the development of the OSI
(Open Systems Interconnection) model reference architecture. The OSI
model holds a layered architecture and consists of the following layers:

1.  Physical layer

2.  Data layer

3.  Network layer

4.  Transport layer

5.  Session layer

6.  Presentation layer

7.  Application layer

In parallel, internet protocol suites were developed, including the
following and many more:

-   TCP, UDP (Transport layer)

-   FTP, BGP (Application layer)

-   IP(v4 and v6) (Internet layer)

The Application, Presentation, and Session layers of the OSI model were
not developed independently in TCP/IP, which has only the Application
layer above the Transport layer.

In computer science class, you might have learned about UDP (User
Datagram Protocol) as a non-reliable transport protocol and TCP
(Transmission Control Protocol) as a reliable one. This implies that TCP
offers reliability and ordering of data transmission. In the event of
packet loss while transmitting using TCP, the protocol ensures the
recovery of data and maintains the order of packets reaching the
destination. In the case of UDP, if there is packet loss, the protocol
will ignore it and continue without attempting recovery.

### Challenges of TCP

TCP was the backbone of the World Wide Web. However, as the internet
grows, limitations of TCP are exposed.

TCP was silent about security. To address security concerns, SSL (Secure
Socket Layer) was introduced. Over time, multiple versions of SSL were
released, each more secure than the previous one. One upgrade of SSL,
developed by a different group of engineers, was renamed TLS (Transport
Layer Security) to signify ownership change. However, SSL and TLS are
widely used interchangeably. For connection establishment, TCP uses 3
handshakes and 4 handshakes are needed for TLS enablement. This was
causing a long round-trip time for connection establishment.

TCP uses multiplexing/multi-stream. If any stream faces data loss, all
streams are blocked until data is recovered. In a high-traffic scenario,
this mechanism is not considered a good approach.

### Answer to TCP Challenges

QUIC is the solution to major challenges related to TCP. Initially named
as an abbreviation of 'Quick UDP Internet Connections,' QUIC quickly
dropped that abbreviation and adopted QUIC as the name for the new
protocol. In short, QUIC is defined as follows, according to quicwg.org:

> *\"A UDP-based, stream-multiplexing, encrypted transport protocol.\"*

QUIC provides built-in TLS security and guarantees zero or at most 1
round trip to establish the connection. Application data is not blocked
while waiting for the handshake to complete. QUIC exchanges TLS keys in
the initial handshake.

QUIC\'s multiplexing/multi-stream feature is enabled with self-healing
at the stream level. If there is data loss in a stream, QUIC takes care
of it independently without blocking all other streams. QUIC also offers
prioritization and flow control, allowing high-priority data
transmission through priority streams based on application requirements.
In the event of an error in a specific stream, QUIC can restart that
specific stream without affecting all other streams.

Another significant advantage of QUIC over TCP is its ability to
gracefully change networks. For example, when moving from home with
Wi-Fi to cellular networks, QUIC ensures a smooth transition without
service interruption. Similarly, when a vehicle moves across networks
with varying coverage areas, QUIC handles unstable networks and
connection migration gracefully.

It\'s worth mentioning that QUIC provides better congestion control
policies and algorithms compared to TCP.

### Web Versioning

The World Wide Web began as a space filled with static web pages and
information consumers in its initial stage, referred to as web 1.0. The
backbone of web 1.0 is the HTTP (HyperText Transfer Protocol) protocol,
built on top of TCP.

As it transitioned into web 2.0, social media and content creators took
center stage in the web world. Web 2.0 retained the server-client
architecture at its core, emphasizing centralization and the
monetization of data. This brought about its own scalability and
availability challenges, prompting Big Tech companies to develop new
technologies to address these concerns. To tackle these challenges, Big
Tech upgraded HTTP to HTTP2 and enabled bidirectional streams. However,
they quickly recognized the limitations of TCP, and QUIC emerged as one
of the solutions to address those challenges.

Simultaneously, the industry began discussing the concept of a
decentralized web, moving away from the server-client architecture.
Primarily championed by crypto enthusiasts following the success of
Bitcoin, this new paradigm was named web 3.0.

It is intersting to watch and see, will there be a new version for web 
once QUIC is widely accepted across industries.  

### QUIC Adaptation

QUIC adaptation does not require upgrading the existing network
infrastructure, making it easy and quick to implement. QUIC serves as
the base layer of Http3, replacing TCP. Http3 maintains similar
semantics to Http2, using the same request methods, status codes, and
message fields. Major Big Tech companies have embraced Http3 for their
Software as a Service (SaaS) solutions, and currently, all major
browsers support QUIC.
