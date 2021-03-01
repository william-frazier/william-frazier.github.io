---
layout: post
title:  "Network Layer Pt. 2: The Control Plane"
date:   2021-3-1 00:00:00 -0500
tags: [networking]
---

# Introduction

Last time we talked a bit about how IP works through a discussion of DHCP, CIDR, and more. This time around, were gonna take a wider view and see more about how the internet as a whole works.

# Routing Algorithms

Routing algorithms are essentially a shortest path graph problem (with routers as nodes and links as edges). We can use min-hop routing (set the weight of each edge to 1), min-delay routing (set the weight to the delay between the two routers), or something like minimum cost (set the weight to the expected cost of using that link). The two main approaches are known as **Link State** (which uses Dijkstra's algorithm) and **Distance Vector** (which uses the Bellman-Ford algorithm).

For link state, the idea is that routers broadcast out their current delays and such. The other routers in the network gather this info and they can all run Dijkstra's locally. This delay information, known as a **link state packet (LSP)**, will contain a sequence number. If a router encounters an LSP with an already seen sequence number, they drop it. Otherwise, incorporate the new info and send the LSP on to other routers.

For distance vector, we occasionally send our neighbors all of our distance vectors which is the tuple _(destination, distance)_. We use the received distance vectors to compute our own. If our neighbor _X_ says it is 5 units away from _Y_ and we are 2 units away from _X_ then we are at most 7 away from _Y_ and we add (7,_Y_) to our distance vectors. Now other routers know that we have a path to _Y_ which is 7 units long. If the best path they know of is longer than that, they can start routing traffic to _Y_ through us.

 This leaves us open to getting stuck in loops. To prevent this, each router adds itself to this distance vector. Then if some router is looking at destination _X_, it advertises that it itself is infinitely far away from _X_ to all of the routers on the path (it sends its real distance to the other routers). This process of sending infinity to routers on the path is a technique known as **poisoned reverse**.

Distance vector is easier to implement and if the path changes without affecting the shortest path, no update messages need to be sent. On the other hand, the routing messages are proportional in size to the number of nodes in the network because the whole path needs to be encoded and it's slow to converge because the computation is distributed.

The thing is, this is all way too simplified. The internet has an unbelievable number of routers so clearly these algorithms can't be implemented as is. The way we solved this problem is with the creation of **autonomous systems (ASes)**. The idea is that an ISP's entire network is one AS (some ISPs break it into several). This means that each AS runs these algorithms. Now, ISPs have control over the algorithms that their routers will use and there are fewer routers to work with. Each AS is given a globally unique number (called an ASN) which is used to identify it. Like IP addresses, these are given out by ICANN. Now we just need to route between ASes as a whole, and route within each AS individually--a much simpler problem than routing over the entire internet.

A link state based algorithm commonly used in ASes is OSPF (open shortest path first). The "open" means the algorithm is public. The main changes to what was laid out in the section on link state is that here the router broadcasts its info to all routers in the AS, not just its neighbors; at least every 30 minutes, routers update the system even if there were no changes; and authentication can be added to ensure only authorized routers are communicating. OSPF can also be configured to work with hierarchies, breaking the AS into smaller areas and routing between them.

To coordinate between ASs, all ISPs need to use the same routing protocol. Here they all use the **border gateway protocol (BGP)**. This is a really important protocol and I'm expecting to make a whole blog post soon about it.

In an AS, a router is either a **gateway router** (meaning it has connections to one or more other AS) or it is an **internal router**. BGP runs over TCP and when it connects two ASes, it is called an **external BGP connection (eBGP)** and when it is within one AS, it is called an **internal BGP connection (iBGP)**. There is typically one eBGP connection for each link connecting ASs. A common configuration would be to have an iBGP connection for each pair of routers in an AS. Each BGP path also has some attributes. Two of the most important are AS-PATH (the ASNs for the ASes the path traverses) and NEXT-HOP (the IP address of the first router in a different AS).

One type of BGP routing is called "hot potato" routing. In this, the packet will be forwarded to whichever NEXT-HOP IP address can be reach with the lowest cost (i.e., for all possible options of forwarding the packet, which one gets the packet out of the AS fastest?). In practice, we use a more complicated algorithm called "route selection". Here, we add another attribute to each path, called its "local preference". This value can be determined in anyway a network administrator sees fit (e.g., by the ISP's preferred routing algorithm). The route with the highest local preference will be chosen. If there is a tie, the route with the smallest AS-PATH is chosen. If there remains a tie, hot potato routing is used. Often, BGP tables will include more than half a million routes. Of course, no one is enforcing any of this. If you control an AS, you can route packets however you want, this is just a common way of doing it.

This protocol is really useful for CDNs (think Netflix owning many servers to host content) and similar systems which allow us to serve data from multiple servers around the world. Simply assign all servers the same IP address and BGP will naturally determine, at each router, where to send incoming packets headed for that address. BGP has no concept of this IP being used multiple times but rather thinks it's choosing the best path. In reality, it's always giving the user the route to the easiest to reach server that can fulfil the request. This is known as **IP-anycast**. In practice, we don't actually use this for CDNs (because a TCP connection might end up getting routed to two different servers) but we do use this for DNS.

A final note on routing, ISPs will often hide paths in order to achieve certain goals. For example, ISPs don't want to just forward traffic from one ISP to another so they make sure to not advertise to ISP B any paths going from them to ISP A and vice-versa. There are a number of different thing like this that we can do as well as some rules of thumb that ISPs provide (such as forwarding each other's traffic as long as the destination or source is part of our own ISP). One important rule of thumb is that the largest ISPs tend to allow their competitors traffic to flow through their ASes at no cost; I believe the assumption is that all parties benefit an equal amount from such an arrangement.

# ICMP

The **Internet Control Message Protocol (ICMP)** is used by hosts and routers to communicate network-layer information to each other. Most commonly this is used for errors. For example, when you get a "destination unreachable" error while browsing the web, that means some router didn't know where to send your request.

ICMP is often considered part of the IP protocol but actually it's situated just above it. ICMP packets are sent as the payload of an IP datagram (with a protocol number of 1). In addition to specifying the type of ICMP packet, these packets also include something called a code which further specifies the type (e.g., type 3 code 1 means destination host unreachable while type 3 code 3 means destination port unreachable) and the first 8 bytes of the datagram which caused the error so that the sender can examine it.

ICMP isn't only for errors though; for example, ping is also an ICMP message with type 8 code 0. There is another message (type 4 code 0) which tells the sender that the network is congested but this is rarely used in practice due to TCP's built in congestion control. Traceroute also comes from ICMP. Basically, UDP packets are sent to an odd port number starting with a TTL of 1, then 2, and so on. When a packet expires, the router replies with a ICMP packet of type 11 code 0 and from this the program can conclude the path. When the host is finally reached a type 3 code 3 (destination port unreachable) is sent back and traceroute knows it can stop sending more packets.

ICMP was updated slightly for IPv6 I don't know of any major changes.

# SNMP

The **Simple Network Management Protocol (SNMP** or sometimes **SNMPv2)** is a protocol which allows a network administrator to gather statistics about the network. Each managed device in the network has a **management information base (MIB)** which is basically a collection of information about the device (e.g., the number of UDP segments dropped). SNMP can be used to query these device for those statistics, ask the status of devices, or for devices to inform the managing server of events such as a link going down. Note that the managing server is the center-piece of all this where the network administrator would work.

SNMPv2 has seven different types of messages, known as protocol data units (PDUs). The messages are typically carried in UDP segments. Because UDP is unreliable, SNMP headers include an ID number that allows the managing server to detect lost packets. There is no set rule for retransmission, it's up to the network administrator.

The new version, SNMPv3, is mainly the same as version 2 but with added security features. Due to the lack of these before, SNMP was mainly used for monitoring networks rather than control.
