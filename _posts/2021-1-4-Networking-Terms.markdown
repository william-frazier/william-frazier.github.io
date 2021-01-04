---
layout: post
title:  "Networking Terms"
date:   2021-1-4 00:00:00 -0500
topic: networking
---
#Networks

Packets of data on the internet are routed through, unsurprisingly, **routers** (and packet switches but we'll get to that in a later post). A rrouter is a node that connects networks. The addresses sent with each packet need to be unique in order to know how to route the packet. Routers use store-and-forward transmission. This essentially means that a router needs to receive the full packet before it will send the packet on. Routers also have an output buffer where packets can be stored while they're waiting to be sent. If this buffer gets too full, the packet will be dropped. Traditionally, this was the source of most packet loss, but in modern networks, packet loss also often occurs due to WiFi which is far more likely than wired networks to corrupt a packet.

Modern networks are called packet switching networks. The gist of the idea is we build a bunch of internet infrastructure and send packets into it, hoping the arrive at the destination. That is an intentionally vague statement. We'll explore this idea soon enough. There is another type of network we won't really touch on in these posts called a circuit switching network. In these networks, all the resources needed for communication are reserved ahead of time (think telephone networks). Packet switching is more efficient but also more unpredictable.

The first form of a network was a WAN (wide area network). WANs have a max packet size that can be sent on them. There are a few reasons for this such as switches only having limited memory and long delays for packets stuck behind huge packets. This max size is called the **maximum transfer unit (MTU)**. This will come up again later in our study of networks.

WANs become too complicated to provide worldwide networks largely because it's too hard to detail the path a packet should take to traverse the whole globe. So we have an internet (lowercase i) which is a bunch of WANs and LANs connected together.

You'll often see the term **Local Area Network (LAN)** when reading about networks. LANs are basically a bunch of computers all hooked to one link. They are actually pretty complicated to control and we can have collisions where two messages are sent at the same time and their bits are scrambled (see the next section for how we handle this). The data frames are sent along with some header that says which computer the frame is destined for (because all computers on the LAN can view any message).

There are two main limitations to how large a LAN can be. The first is workload: essentially there end up being too many collisions. The second is **signal attenuation**: this is a physical property where bits become more similar after long distances (i.e., the voltage difference between 0 and 1 becomes smaller) and the signal must be boosted.



#Transmitting

When we want to send a number of packets down a link (a wire) at the same time, we have a few ways of doing this. We can use **frequency-division multiplexing (FDM)** which essentially means we divide up the frequency spectrum of the link and each packet gets its own frequency to transmit on. So if we have a 10Mbps link and 10 packets we want to send, each will be sent at a rate of 1Mbps. We could also use **time-division multiplexing (TDM)** where we divide time into fixed duration frames, then divide these frames further into time slots. Each packet now gets one of these time slots each frame. Packets get full use of the frequency during its time slot. So in the previous scenario, we might send the first packet at a rate of 10Mbps but only for a tenth of a second. Then the full link capacity is given to the second packet. And so on. If a packet wasn't fully transmitted during its time slot then it will wait until the next frame when its time slot comes up again and send more of its data.

TDM leads to some waste because when a host is given a time slot, they may have no packets to send. So we tend to use something called **statistical multiplexing (SM)** which is basically TDM but time slots are assigned based on demand, not equally. So if multiple people send packets to the router, it will use the full bandwidth to transmit them, sending packets in the order that they arrived (there is no requirement that this uses a FIFO protocol but that's pretty standard).


#ISPs

An **internet service provider (ISP)** is the entity that provides internet connectivity. An **access ISP** is something like a university or business providing internet. They connect to a **regional ISP** which links these together. They connect to a **tier-1 ISP** which are more global entities. There are about a dozen of these (think AT&T). Major content providers like Google also make their own private networks and connect directly to local ISPs when possible. ISPs at the same level will also **peer** with each other which means that they allow each other's traffic to flow through. This is generally done without payment by either side. Sometimes, companies provide **Internet Exchange Points (IXPs)** as dedicated sites for this.

#Delay

**Processing delay** is the time required to examine the packet’s header and determine what to do with it. This is typically microseconds or less. **Queuing delay** is the length of time a packet has to wait before it can be sent along the wire due to congestion. This is typically microseconds to milliseconds at most but could be 0 if the wire has space for the packet as soon as it arrives. **Transmission delay** is the length of the packet divided by the transmission rate of the link (the amount of time it takes a router to push out the packet). This is typically microseconds to milliseconds. **Propagation delay** is the amount of time it takes a packet to traverse the wire. In a large network, this is on the order of milliseconds. The total **nodal delay** is the sum of all of these delays. Queuing delay is the most interesting and typically we talk about it in terms of statistics like average queuing delay because it can vary wildly from packet to packet.

The average number of bits arriving per second at a router divided by its transmission rate is known as the **traffic intensity**. This value needs to be kept under 1 or the queuing delay will approach infinity. Obviously we want to keep this value as low as possible but it's important to note that queuing delays increase exponentially as we get close to a traffic intensity of 1 (or rather it increases exponentially until the buffer fills and then we start seeing more packets getting dropped). So if we are near that point, even small changes to our traffic density can dramatically alter the delay. Typically, we think of traffic intensity of less than 0.8 as causing only small delays. The performance at a node is typically measured both in terms of delay and probability of packet loss.
