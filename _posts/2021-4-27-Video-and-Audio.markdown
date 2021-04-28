---
layout: post
title:  "Video and Audio"
date:   2021-4-27 00:00:00 -0500
tags: [networking]
---

# Introduction

Video can be efficiently compressed to essentially any bit rate. The higher the bit rate, the better the video quality/fidelity. There are two main types of compression: **spatial** and **temporal**. The idea behind spatial compression is that if a video has a solid black background, there's no reason to encode values for every pixel and instead we just use spatial compression to say that there's a black background. Now suppose someone is talking in front of that background, we use temporal compression to say that the background and the speaker's torso haven't moved and then we only need to send new info about the speaker's face.

Audio is typically constructed by taking a number of samples and then **quantizing** (rounding) those samples. So for example, CDs take 44,100 samples a second and then quantize each sample to one of 65,536 values, meaning this encoding uses more than 700Kbps. This strategy is known as **pulse code modulation (PCM)**. The more samples we take and the more values the samples can take, the higher the fidelity of the audio but the more bits required to represent it. PCM uses a lot of bits so we tend not to use this for streaming music. Instead we compress it. Perhaps the most popular compression algorithm is MP3 (MPEG 1 layer 3). This can compress to a number of different rates but we commonly use 128Kbps which is able to produce music that is almost as good as CD encoding.

When using the internet to talk to someone, called voice-over-IP (**VoIP**), or video conferencing, we face a very different challenge from other uses for the web. These use cases are very loss tolerant (a few dropped packets will only rarely hurt the flow of the conversation) but the delay needs to remain quite small or else the flow of conversation can break. Human speech can reliably be compressed to less than 10Kbps.

# Streaming Video

Video can be streamed using UDP, HTTP, or adaptive HTTP (see below); although UDP streaming is becoming less common. UDP streaming basically works by having a server just send data at the rate the client wishes to watch it, with the client keeping only a small buffer (typically less than one second of video). This has the major drawback that any drop in throughput will hamper streaming; it also requires the server to keep track of which clients are currently playing/pausing/fast-forwarding/etc. which greatly increases the size of the server state.

Most streaming services (including YouTube and Netflix) use HTTP and TCP to stream video. In this case, each video is stored as a file and the user's web browser requests that file. Once enough of the video has been buffered, it will begin playing. TCP's sawtooth behavior (where its throughput varies due to congestion control and such) was thought to make TCP unusable for video streaming but recently we've seen that with buffering and prefetching it ends up not normally being a huge problem (given that TCP can provide a throughput at least twice as high as the video bit rate). If a user wants to jump to some other position in the video, they are able to use the HTTP byte-range header to specify where they want the server to send data from. Note that jumping ahead means all the buffered video goes unwatched which wastes resources so most streaming services impose a somewhat small buffer on clients. This is why even if you leave a YouTube video paused for a long time, only a little of it will end up being fetched.

Adaptive HTTP (formally called dynamic adaptive streaming over HTTP or DASH) is essentially the same as HTTP streaming but it allows for different video qualities.

# VoIP

Voice over IP introduces its own set of challenges. Some packets can be lost (but not too many) and the delay as well as the variance in delay needs to remain quite low.

One way to ensure fewer packets are lost is to, after *n* packets are sent, send another packet that's the XOR of them all. If only one packet was lost than it can be recovered. If more than one packet was lost, this is useless. Also, if *n* is set to be low, then this dramatically increases the transmission delay.

A better solution is to build a much lower bit-rate encoding of each chunk. The low level encoding of chunk *n* can then be attached to the normal encoding of chunk *n+1*. This way, if a packet is lost but the one after it arrives, a low bit-rate encoding can still be played for that chunk. We can also tweak this by including the worse encodings at more locations. This will take more bandwidth but allows for a higher chance of recovery.

Another technique is called **interleaving**. Suppose each packet contains 4 snippets of audio. Then we wait for 4 packets to be ready to send and then spread their snippets out. This way, if a packet is lost, the lost data is spread out and so there is no big gap. This increases latency so it's not a great idea for conversations but it is useful for streaming audio.

Many programs will also simply try to guess what the missing audio would have been. The easiest way to do this is to just repeat the last packet. A better but more computationally expensive approach is to try to actually simulate what the next packet would have contained. It turns out that we can normally do this with human speech.

# Next Post

The next post is going to be a quick overview of some network security. I'll have a lot more to say about that in future posts but for now let's keep it simple. This post will also be the last in this network series that I've been working on and soon I'll start on something new. Stay tuned!
