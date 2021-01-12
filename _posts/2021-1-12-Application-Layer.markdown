---
layout: post
title:  "The Application Layer"
date:   2021-1-12 00:00:00 -0500
topic: networking
---

# Introduction

This post is going to be a little bit odd because it can be a little hard to talk about the application layer. There's so much that *could* be said but not a whole lot that is actually worth saying. This is for a few reasons but to me the most relevant one is that applications aren't a great insight into computer science. Sure DNS and HTTP are important to understand but they don't really bring up any new or exciting concepts. Therefore, I'm going to keep this post somewhat short and stick to the basics. If you're interested in any of these applications then it's very easy to find detailed explanations online.

# HTTP

HTTP (the hypertext transfer protocol) is the basic mechanism for delivering webpages (as well as sometimes email and videos but that's a discussion for later). You've almost certainly noticed it when you go to a website like [http://aratools.com](http://aratools.com), a good online Arabic dictionary. Here the "http://" is specifying which protocol to use. You may have seen alternative protocols like "file://" if you use Chrome or another browser to open a file on your computer.

HTTP is pretty simple and was originally human-readable (HTTP/2 and /3 have moved away from this, I won't talk to much about them because the underlying concepts remain the same). Basically, when you want to fetch a website, say [google](google.com), you send out a GET request. This request includes a few lines of information such as the file the user is requesting, the version of HTTP being used, the host website, the browser being used, accepted languages, and more or less depending on how everything is configured. If all goes well, the server will reply with "200 OK" along with the webpage, the type of the file being returned (e.g., text/html, note that this has some security implications I hope to explore in a future blog post), the date the file was last modified, the length of the file, and again more or less information depending.

Here the "200 OK" is the status code. You've also almost certainly seen some of these, such as the infamous 404 error. There are quite a few of these codes but for reasons I'm not totally clear on, a lot of web developers decided to only use a handful of these. Assuming the webpage was successfully delivered, it is likely that more requests have to be made. For example, if there were images embedded in the page, you will almost certainly have to make a request for those pages as well.

I should also note that there is more than just GET requests. There are also PUT, HEAD, POST, DELETE, and a couple others. HEAD is interesting because it works like a GET request but only asks for the header to be returned and not the actual webpage itself (i.e., it wants the status code, the date the file was last modified, and so on). This is interesting because developers tend to forget that it exists which has [led to some problems in the past](https://blog.teddykatz.com/2019/11/05/github-oauth-bypass.html).

There is also a thing called HTTPS (HTTP secure) which is what every website should be using these days. It works the same as HTTP except it is, obviously, more secure. I think a discussion of how it works would be better placed in a security blog post.

# DNS

Now this is all well and good but how do we know where to send our HTTP request? This is where DNS (the domain name service) comes into play. All servers are addressed by a 32-bit IP address (or 128-bit in the case of IPv6, yet again, a topic for a future post), something like 192.168.12.2. Of course, that's not very memorable and we prefer to use human-readable names like [google.com](google.com). DNS allows us to translate these human-readable names into IP addresses.

Basically, what happens when you try to reach [google.com](google.com), is your computer will first contact a root DNS server whose IP addresses are hardcoded into your computer. That root server will know where lots of other DNS servers are and will send you the IP address for one that has jurisdiction over .com addresses. This DNS server will then send the IP address of Google's DNS server. You can then contact this server and it will tell you which IP address you want (there may be different IP addresses for Google image search vs regular [google.com](google.com) for example).

This is in theory how it works but it's likely to be a little different in practice. For example, you almost never need to contact a root server. For the vast majority of requests, you can just connect a level immediately. Also, it's unlikely that the DNS server will return the IP address of another DNS server; more likely, the DNS server will just go ahead and ask the next server for you. This way, you send out one request and get one reply which is the final IP address that you wanted. Also, there are some security features I'm not going to talk about that have been added to prevent attacks such as [DNS cache poisoning](https://arstechnica.com/information-technology/2020/11/researchers-find-way-to-revive-kaminskys-2008-dns-cache-poisoning-attack/). The last thing that I'll add is that DNS servers don't actually return IP addresses but these things called resource records. This could be an IP address but depending on the request it could also contain other information such as the name of the website's email server which I don't find particularly interesting to explore.

# Conclusion

There are a huge number of other protocols that I could talk about here. There's SMTP (the simple mail transfer protocol) which moves email from the sender's inbox to the recipient's inbox. There's FTP (the file transfer protocol) which does exactly what is implied by the name. I may bring up some additional protocols in future blog posts but for now, I think this is good enough.

Additionally, I think a part of being a good computer scientist is necessarily being a good citizen; I certainly don't intend to make this a political blog so I will make this brief but I do think it's important to raise our voices when the moment calls for it. This is the first blog post I have made since the sack of the Capitol on January 6th and as such, I just want to say: the attack on our democracy was as despicable as it was frightening to watch play out. I have plenty of thoughts on the matter but I'm not sure this is the best venue to express them. 
