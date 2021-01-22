---
layout: post
title:  "The Transport Layer"
date:   2021-1-20 00:00:00 -0500
tags: [networking]
---

# Introduction

Next, we need to talk about TCP, which is actually really interesting once you dig down into it. But there's so much to say about TCP that I think the blog will be better off if this post is just getting us introduced to TCP.

TCP is a protocol in the transport layer of the stack. The transport layer is basically in charge of getting packets data to the correct program on another computer. With that definition, it's easy to get confused about what exactly this layer is doing. It is *not* routing packets through the internet. Instead, this layer is in charge of **multiplexing** (the gathering of packets being sent into the internet by different applications and encapsulating these packets in the correct header) and **demultiplexing** (the reserve, essentially taking incoming data packets and making sure they all go to the correct application). This is a subtle but important distinction. Consider if you were Skyping with someone and at the same time you sent them an email (this example actually isn't quite right but it's good enough to explain the distinction). You wouldn't want the recipient's computer sending your email to Skype and your video/audio information to Gmail. These programs wouldn't understand what's being sent to them. The transport layer allows us to ensure that each application receives the correct packets.

Packets at this level are called **segments** although sometimes people will call UDP segments datagrams which is confusing as datagram is the term used in the network layer (UDP is covered later in this post).

# In More Detail

The transport layer multiplexes these segments by detailing which **port** they should be sent to. Ports don't actually represent anything physical, they're just an identifier. So our transport layer protocols know which socket to give the data to thanks to a port number. Port numbers are 16-bit identifiers (0 to 65,535) with ports 0-1023 reserved for well known applications. For example, if you're using HTTP, your computer will try to connect to a server's port 80 (the port number associated with HTTP). There's nothing special about the number 80, anything could be used. Indeed, you'll occasionally see HTTP hosted on port 8080. The point is that port 80 is well known for being the HTTP port; if your computer tried to connect to a server on port 80 but they were serving HTTP pages from port 55,555, how would you learn this? This will prevent you from connecting. That's why we have some standard port numbers for common protocols.


# UDP

The **user datagram protocol (UDP)** is one of the transport layer protocols. It's a very simple protocol. UDP doesn't really provide any services beyond the ability to multiplex messages (unlike TCP which we'll see in the next blog post). The only feature UDP provides in addition to this is a checksum to check for message errors. A UDP segment only has 8 bytes of overhead in the header (4 values, all 2 bytes long). These four fields are: source port number, destination port number, a length field specifying how large the segment is (header included), and the checksum. While UDP doesn't contain many guarantees, we could build many of them into the application itself, although this is by no means a trivial process.

What this means is that UDP messages can get lost or corrupted once they are sent, there is no guarantee that they will arrive at the destination. This clearly isn't acceptable for something like HTTP (imagine loading a webpage and occasionally parts of it just wouldn't load). But at the same time, UDP is often faster than TCP (we'll talk about this in the next post) so there are some situations where UDP is useful. Consider using an application like Skype: it doesn't matter if every single frame of the video arrives (we won't notice occasionally tiny glitches) but at the same time we need packet deliver to be as fast as possible (we don't want to big of a delay or else it gets hard to have a conversation). In that case, UDP is a good choice. UDP is also used for DNS and a few other things although TCP is far more prevalent.

It's also worth noting that UDP segments addressed to the same IP and port number will arrive at the same socket even if they come from different sources. TCP however routes packets from different sources to different sockets. The implications of this are interesting and possibly worthy of a future blog post but not right now. Hopefully, this gives us enough of a background to tackle TCP in the next post. Some of this may seem a little incomplete or confusing but that should all clear up once we have a chance to cover TCP.
