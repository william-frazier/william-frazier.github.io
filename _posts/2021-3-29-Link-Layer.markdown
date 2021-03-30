---
layout: post
title:  "The Link Layer"
date:   2021-3-29 00:00:00 -0500
tags: [networking]
---



# Introduction

I am going to refer to every device on the link layer (hosts, routers, switches, APs, etc.) as **nodes** in order to simplify terminology. Recently, I made a post explaining how a packet crosses the wider internet. But so far, we haven't yet covered how a packet (in this layer called a **frame**) gets from one node to the next; for example, how does data get from my laptop to my WiFi router?

There are a number of different link-layer protocols that all operate a little differently. The protocols also require some sort of way of determining link access. If there is only a single host we don't really need this but when we have multiple hosts all on the same AP for example, we need a way to stop the hosts from communicating at the same time. Some protocols also offer reliable delivery. This is really only used in WiFi because the error rate on wired links is so low that the reliable delivery overhead just isn't worth it. Finally, protocols can offer some sort of error detection and correction. These systems tend to be more complicated than the TCP/IP checksums.

The link layer is generally implemented in hardware, on what's known as the **network adapter** or **network interface card (NIC)**. The core of this adapter is the link-layer controller which handles most of the link-layer responsibilities.


# Error Detection and Correction

In general, the link layer attaches **error detecting and correcting (EDC) bits** to the message. This is generally calculated over the payload and some of the header fields.

The simplest technique we could use is called a **parity bit**. We can use either an even or an odd scheme here. Suppose we use an even scheme, then we simply include a single bit at the end of the message which is a 1 if the number of 1s in the message is odd, or a 0 if the number of 1s is even. When the receiver gets this message, they simply count up the total number of 1s (including the parity bit) and ensure this number is even. This only allows us to catch an odd number of errors as an even number will make the number of 1s stay even. In theory, if we had some link with really low error rates this might be ok because we are expecting at most one bit flip. In reality, bit flips tend to occur in bursts rather than on just one bit so this scheme doesn't really work.

We could make this scheme a bit better by using a two-dimensional version of it. Suppose we broke our data up into strings of *n* bits. Then, we could calculate the parity bit for each row and for each column. If we have a single error, we can not only find it, but correct it (try it yourself to see what I mean). If we have two errors, we can find them but we can't correct them. And if there is a single error in the EDC bits then we can detect this as well. A scheme like this might conceivably be used for something like real time audio because its better to often be able to fix bits and move on, rather than waiting for NAKs and the like to trigger a retransmission.

Now we could make this even stronger and use checksumming like we did with TCP and IP. But in reality, that doesn't provide great error detection, we use it in those protocols mainly because it's really fast. But we can do something more interesting at the link layer because most everything is done by hardware, not software. We instead use something called **cyclic redundancy check (CRC) codes** (sometimes called **polynomial codes**).

To explain this, assume our data is *d* bits long. The sender and receiver will agree on an *r+1* bit pattern called a **generator**, which we will denote *G*. Note that the leftmost bit must be a 1. The sender then appends *r* bits to the data being sent such that if this message is interpreted as an integer, it is divisible by *G* using mod 2 arithmetic. It isn't difficult to find the *r* bits to make this work but I neglect the calculation here. The math works out such that CRC codes can detect burst errors of up to *r* bits in length, can detect longer errors with high probability, and can always detect an even number of errors.


# Broadcast Links

We can have point-to-point links but there also exist broadcast links. Here, a frame is sent to all nodes that are part of this link. This leads to the **multiple access problem**, basically the issue of coordinating when hosts are allowed to send data.

We use **multiple access protocols** to coordinate. There are three main types of these protocols. The first are **channel partitioning protocols**. These protocols use something like TDM or FDM (link [here](https://william-frazier.github.io/2021-01-04-Networking-Terms/) for a refresher) to divide up the channel and allow all nodes to send. An example is **code division multiple access (CDMA)** where each sender has a certain code and all nodes can send at once with the property that the receiver can still find the sender's message, even through a collision. This is pretty complicated and really requires the use of good visuals (which is not my strong suit) so I'm not going to go into detail. That being said, the scheme is pretty interesting so I recommend looking it up. I had trouble finding a good webpage to share so if you have the [textbook](https://www.amazon.com/Computer-Networking-Top-Down-Approach-7th/dp/0133594149#:~:text=Unique%20among%20computer%20networking%20texts,down%20toward%20the%20physical%20layer%2C), I'd start there.

The second type of protocols are **random access protocols**. These protocols allow all nodes to send at the maximum rate and if there is a collision, those nodes wait a random amount of time before retransmitting the frame which collided. Because the delay is random, hopefully the second time the frame will make it through. An example is the **Carrier Sense Multiple Access with Collision Detection (CSMA/CD)** protocol. Carrier sense refers to nodes being able to detect when another node is broadcasting; collision detection is still needed because nodes may think the link is clear because bits being broadcast have yet to reach them. In Ethernet, if there is a collision, the host picks a random number *k* from 1 to *2n-1* where *n* is the number of times that frame has collided (and capped at 10). Then the host waits for however long it would take to transmit *k\*512* bits. This is known as **binary exponential backoff**.

The final type are **taking turns protocols**. An example includes the **polling protocol** where some master node goes around and tells hosts that it's their turn to transmit. This introduces some delay and also fails if the master node goes down. A similar approach is the **token-passing protocol**. Here, there is a token which tells hosts they can transmit. When the host is done, they pass that token on to the next node. This also has the problem that a non-responsive host can bring the whole system down.



# Link Layer Addressing

**MAC addresses** are the equivalent of IP addresses for the link layer. Although they were designed to be permanent, now it is possible to change them in software. Each network adapter gets its own 48 bit MAC address (technically, there can be different sized addresses but wireless and Ethernet use 48 bit). Companies purchase a 24 bit prefix from IEEE and then produce adapters whose MAC addresses begin with that prefix. FF:FF:FF:FF:FF:FF is the broadcast MAC address.

In order to match IP addresses with their MAC addresses, we use the **Address Resolution Protocol (ARP)**. This is very much like DNS with the exception that it only works for hosts on the same subnet (i.e., you can't issue an ARP request for some computer in a different country or even down the block). Every host and router has an ARP table. Each entry in this table is an IP address, the corresponding MAC address, and a TTL (typically 20 minutes). Basically, when a host wants to send a message, they look to see if there is the corresponding entry in their ARP table; if not, the host broadcasts an ARP packet to FF:FF:FF:FF:FF:FF. Every host on the network will check to see if that ARP request is looking for their IP and if so they reply with their MAC address. ARP packets are encapsulated in link layer frames.

It's worth reflecting on why we need a link layer address and an IP address. There are a few good answers to this but the basic explanation is that an IP address is easy to change so that we can move from network to network and have packets reach us. This makes it easy to build routing rules since IP addresses will always stay in the same geographical area. Meanwhile, a MAC address gives us something hardcoded which allows for really fast hardware implementations. This means a computer can quickly drop all of the packets it sees which aren't destined for it.

Suppose a host wants to send a message through a router to a host on another subnet. What happens is the host gets the MAC address of the router using ARP and sends the packet. The router gets the packet, and moves it to its outgoing interface. Now the router uses ARP to find the MAC address of the destination (of the next hop) and sends off the packet. This way we keep ARP to only a single subnet at a time.

# Ethernet

Ethernet was invented in the mid-70s and now is the dominant LAN technology, mainly due to its early arrival and relative simplicity. Ethernet frames can carry a payload of 46-1,500 bytes. If the message being passed is smaller than 46 bytes, the message is padded and the receiver can look at the IP header to get the packet's length and remove this padding.

The header begins with an 8-byte preamble. The first 7 bytes are 10101010 and the eighth is 10101011. This basically just tells the receiver to wake up and when two 1s are found next to each other, important data is arriving next. This is followed by the destination MAC and then the sender MAC. After this comes a 2-byte type (Ethernet can carry more than just IP datagrams so we need to let the receiver know what's arriving). Then the data, followed by 4 bytes of cyclic redundancy check (CRC) so the receiver can detect errors.

Ethernet is connection-less and unreliable. The CRC allows the receiver to detect errors but no acknowledgements are sent (neither in the case of receipt of a frame nor if an error was detected). This allows the protocol to remain relatively simple and cheap. Technically, Ethernet is just a name we use to refer to a wide swath of protocols. Nowadays, we mostly used switched Ethernet which negates the whole problem of collisions. With this upgrade, as well as the greatly increased speed, today's Ethernet is really different than the original specification. However, the frame format has never changed.

# Switches

A **switch** is a router that looks only at link layer information. Routers can be configured to do this as well but in general, a router looks at IP addresses while a switch looks at MAC addresses. Hosts don't need to know about switches because they don't affect anything. When a frame arrives at a switch, it consults its **switch table**. If there is an entry for that MAC address, the switch follows that entry for forwarding the frame. If there is no entry then the switch broadcasts the frame on all links except the one it was received on. If the MAC address is supposed to be forwarded to the link on which it was received, this likely implies the switch has received a broadcast message, so the switch simply drops it.

These switch tables are built automatically. When a frame arrives at a switch, the switch adds an entry for that MAC address with the link it arrived on as the destination. These tables have a value called the **aging time** and entries are purged after this amount of time. Switches are just plug-and-play, no configuration needed. Switches often also gather statistics which network administrators can use.

# Next Post

We have now covered all of the layers of the network stack except for the physical layer. The physical layer isn't covered in the [textbook](https://www.amazon.com/Computer-Networking-Top-Down-Approach-7th/dp/0133594149#:~:text=Unique%20among%20computer%20networking%20texts,down%20toward%20the%20physical%20layer%2C) and while I keep meaning to learn the engineering-side of this, I've been too busy and haven't gotten around to it yet. Because of that, the next post will *not* cover the physical layer and instead we'll move into the next chapter of the book which is all about wireless technology.
