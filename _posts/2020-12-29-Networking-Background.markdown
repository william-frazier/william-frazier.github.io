---
layout: post
title:  "Networking Background"
subtitle: A brief introduction to networking
date:   2020-12-29 15:12:00 -0500
comments: true
tags: [networking]
gh-badge: [follow]
gh-repo: william-frazier
topic: networking
---
I want to write a series of posts about how modern networks operate. Most of this
information is coming from a graduate course I took at Boston University and the
textbook [Computer Networking: A Top-Down Approach][textbook-link]. I'm not going
to claim to be an expert on any of this and also the textbook is a few years out
of date so some information may be a little bit dated. But with that, let's get
started!

# Some Background
I will use KB to mean 2<sup>10</sup> *bytes*. Meanwhile I'll use Mbps to mean 10<sup>6</sup> *bits* per second. That is, memory is base 2 and bytes but bit rate is base 10 and bits.

Although people use bandwidth to mean bit rate, this isn't correct. Bandwidth is more specifically related to the frequencies a link can carry. We really won't talk about bandwidth other than in the following term.

The **bandwidth delay product (BxD)** is the bit rate times the round trip delay (notice that bandwidth is misused). It can also be the one-way time occasionally. As this increases, it becomes harder to manage a channel. Suppose we have a 100ms delay and a 45Mbps link, there 4,500,000 bits in the air before we get a response. That means about 550KB of data need to be sent off into the ether before we get a reply which could easily be "server not found."

# Network Models

In the OSI model:

The application layer is the user interface.
The presentation layer does things like encryption but also ensure that if your computer encodes information differently, that it is converted to the correct web format and back again. A header in this layer might be something like what encoding we are using.
The session layer controls checkpoints and data synchronization. Basically organizes and synchronizes data exchange. This and the layers above are application specific. You'll often see people combine these first three layers into a single layer.
The transport layer controls multiplexing/demultiplexing; breaking large messages up into smaller packets; and flow, congestion, and error control (e.g., ensuring packets aren't sent faster than the client can receive). Congestion control refers to ensuring messages are sent slow enough to not overwhelm the network while flow control is about ensuring messages are sent slow enough to not overwhelm the receiver. Headers in this layer might contain sequence numbers so the client can reorder the received packets. There is a layer between the transport and network layer called the internetwork sub-layer which connects different networks.
The network layer controls addressing and routing (for a single network). This and the layers below are network dependent.
The data link layer controls at the link level (moving a single jump). The data link contains headers but also trailers which are needed to delineated the end of a message. There is another layer between the data link and physical layer called the MAC sub-layer (medium access control). This sub-layer handles shared links so that we can avoid collisions.
The physical layer converts bits to a physical signal.

We call the process of adding headers as we move through the layers **encapsulation** and the reverse process on the receiver side **decapsulation**.

There is another model called the **TCP/IP** (or **Internet**) **architecture**. There are four layers: application, transport, internet, and network interface. The data units are called: messages, segments, datagrams, and frames. Here, the application layer contains OSI's application, presentation, and session layers; the transport layers are the same; the internet layer contains only the OSI's internetwork sub-layer; and the network interface (or host-to-network) layer contains OSI's network layer (minus the internetwork sub-layer), the data link layer, and the physical layer. This is the de facto standard. Application protocols include telnet, FTP, SMTP, and DNS. The transport protocols are TCP (transmission control protocol) and UDP (user datagram protocol). The internet protocol is IP. And the final layer is composed of networks (initially e.g., ARPANET, LAN, SATNET, Packet Radio). Originally, there was no distinction between the internet and transport layers.

OSI networks can talk to TCP/IP networks and vice-versa but it requires some gatekeeper to translate the data flow. Even though they're both just models of the same thing, they contain different protocols.

The protocol stack is the collection of protocols you're currently using (e.g., Zoom uses a UDP-IP stack).


[textbook-link]: https://www.amazon.com/Computer-Networking-Top-Down-Approach-7th/dp/0133594149#:~:text=Unique%20among%20computer%20networking%20texts,down%20toward%20the%20physical%20layer%2C
