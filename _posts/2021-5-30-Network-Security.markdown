---
layout: post
title:  "Network Security"
date:   2021-5-30 00:00:00 -0500
tags: [security]
---

[Standard disclaimer](https://william-frazier.github.io/2021-05-28-Standard-Disclaimer/)

# Introduction

This is the final post on the textbook we've been covering. I'm sure I'll touch on these topics more in the future but I wanted to go ahead and give a brief review of them in order to finish up posts on this textbook.

# SSL/TLS

SSL/**TLS** is the core protocol that keeps your information secure online. When you see the little lock icon in your browser, that means you're secured by this protocol. (TLS is the updated version of SSL. If you're curious about the name change, my understanding is that something happen with Microsoft. I'm not clear on exactly what but I believe they forced the name change.)

TLS begins with the TCP 3-way handshake. After this, the TLS handshake happens. First, the client sends a list of cryptographic algorithms it supports and a **nonce**, a number used only once. The server replies by choosing a symmetric encryption algorithm, a public key encryption algorithm, and a MAC algorithm (more on all of these in future posts). The server also sends a nonce. The client then checks the certificate and all to verify this communication. The client then uses the server's public key and generates something known as the pre-master secret (PMS) which is encrypted with the server's public key and sent to the server. The client and server are able to use the PMS and the nonces to construct the master secret. This master secret is not used as a key but is used to derive an encryption and an authentication key for both parties (4 keys total). It actually gets more complicated here with there actually being 3 ways of establishing a shared key.

At this point, all future messages will be encrypted. The protocol ends with the client sending a MAC of the whole handshake. The server will reply with a MAC as well. This allows the client and server to verify that they have all seen the same messages.

When the client wants to send data, TLS will break the data up into records and apply a MAC to each one. Then the record and the MAC are encrypted and given to TCP. MAC-then-encrypt is not something you would normally do (this is simply a mistake in the specification, as always, more on this in a future post). This leaves open a MitM attack where Mallory rearranges the packets (after all, TCP sequence numbers are unencrypted so she can change these). So we actually calculate the MAC over the message and a TLS sequence number which starts at 0 and increases every record.

Before the data and MAC, each record has a type (e.g., handshake, data, or closing the connection), a version number, and a length. These fields are all left unencrypted.

Now, a new version of TLS, TLS 1.3, is rolling out. It changes a lot of this up. I don't want to go into detail here as there are lots of great articles on the changes but it basically made the whole protocol simpler and more secure (including by fixing that nasty MAC-then-encrypt bit).


# IPSec

This is the secure protocol for the network layer. It is often used for VPNs. There are two main protocols: **authentication header (AH)** which provides authentication and integrity and **encapsulation security protocol (ESP)** which provides authentication, integrity, and confidentiality. ESP is much more common and it's what is described here.

First, let's discuss **security associations (SAs)**. These are logical one-way connections between two hosts. Suppose we have an SA from router R1 to router R2. This SA will maintain state information including a 32-bit number called the security parameter index, the origin IP, the destination IP, the type of encryption, the encryption key, MAC algorithm, and authentication key. Whenever an IPSec datagram needs to be sent, R1 will consult this state information in order to construct it. R2 will need to maintain the same state. The internet key exchange (IKE) protocol is used to transfer this information.

There are two modes of datagrams, transport and tunnel. Because VPNs use tunneling, that's what is discussed here. When a router is given an IPv4 datagram to be converted to an IPSec datagram, it first appends an "ESP trailer" (containing padding to make the datagram work with a block cipher, the length of that padding, and the type of data such as UDP contained within), then it encrypts the original datagram (including the header) along with the ESP trailer, and then to the front of this it adds the "ESP header" (containing the security parameter index and a sequence number). This whole thing is called the **enchilada** (no, I'm not kidding). Then the whole thing is MACed and a new IPv4 header is put on it. This new header has source and destination of the two sides of the SA rather than the original destinations. The protocol number in the header is set to 50.
