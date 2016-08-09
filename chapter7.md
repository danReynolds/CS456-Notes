# Multimedia Networking

## Audio

Converting analog to digital signals:

The analog audio signal is sampled at some fixed rate, for example, at 8000 samples per second. Each of the samples is then rounded to a finite number of values. This is called **quantization** and it is typically a power of 2.

Each of the quantization values is represented by a fixed number of bits. For example, if there are 256 quantization values, then each value and each audio sample is represented by one byte.

The bit representations of all the samples are concatenated together to form the digital representation of the signal.

ex. if an analog audio signal is sampled at 8000 samples per second and each sample is quantized and represented at 8 bits, then the resulting digital signal will have a rate of 64000 bits per second.

For playback the digital signal is then converted back and decoded to an analog signal.

By increasing the sampling rate and the number of quantization values, the decoded signal can better approximate the original analog signal.

## Video

Sequence of images displayed at a constant rate, 24 images/sec etc.

Digital image is an array of pixels each pixel represented by bits.

In HD, 1920x1080 * 60fps = large.

Because of its size, it is important to compress video. There are two main types of redundancy in video, both of which can be exploited to enhance compression:

1. **Spatial Redundancy**: An image of the sky consisting most of white space has a high degree of redundancy and can be efficiently compressed without significantly sacrificing image quality.

Can send "red for next 500" to tell the video player that this frame is just red for the next 500 pixels in the frame, making the size of the data sent much smaller.

2. **Temporal Redundancy**: Reflects repetition from image to subsequent image. If one frame and the next is the same image, then there is no need to re-encode and send the next frame, it is more efficient to indicate during encoding that the next image will be the same.

We can only send what changed in the next image, o for the next frame, send the pixels that changed, like pixel number 12 is now blue, etc, and have the receiver update the picture.

**Variable Bit Rate**: The video encoding rate changes as the amount of spatial, temporal coding changes.

**Constant Bit Rate (CBR)**: Video encoding rate is fixed.

## 3 Application Types

## Streaming, stored audio/video

Streaming can begin playout before downloading the entire file. It can transmit faster than audio/video will be rendered, allowing for storing/buffering at the client.

* Streaming: In a streaming stored video application, the client typically begins video playout within a few seconds after it begins receiving video from the server.

Streaming is the process of playout out from one location in the video while still receiving later parts of the video from the server.

* Interactivity: Because the media is prerecorded, the user may pause, reposition, forward, and perform other operations on the video content.

* Continuous Playout: Once a playout of the video begins, it should proceed according to the timing in the original video. This can be difficult because network delays are variable (jitter) so we need a client-side buffer to match the playour requirements. Video packets may be lost and retransmitted and we need to handle those scenarios.

If we have a video server sending data into our client buffer at a rate x(t) and playout rate r, if x > r, then we are removing from the buffer slower than we are filling it and it will overflow.

If x < r, then we are playing too fast and we will run out of frames.

We could also lose packets, waiting on frame 5 but it gets lost, so we repeat frame 4 in its place and then move on to frame 6. The packet for frame 5 comes later but it missed its chance and we drop it. The user generally won't notice a repeated frame.

The client can use a client playout delay and delay the start of playout until a later time, then it will have all its blocks and not play t1 without having t2 fast enough to play it at the correct rate.

## Streaming Multimedia: UDP

With UDP streaming, the server transmits video at a rate that matches the client's video consumption rate by sending the chunks over UDP at a steady rate.

If the video consumption rate is 2Mbps, and each UDP packet carries 8kb, then it must send oout a UDP packet every 8kb/2mb = 4ms.

Because UDP does not use congestion control, the server can push packets into the network at the consumption rate of the video without the rate control restrictions of TCP.

In addition to the video stream, the client must also maintain in parallel a separate control connection over which the client sends commands regarding session state changes such as pause, resume, etc.

UDP streaming has three major drawbacks:

1. Due to the unpredictable and varying amounts of bandwidth between server and client, constant rate UDP streaming can fail to provide continuous playout. If the video consumption rate is 1Mbps and the server-to-client available bandwidth is usually more than 1 Mbps, but every few minutes it drops below 1 Mbps for a few seconds, then a UDP streaming system that transmits video at a constant rate of 1Mbps would freeze or skip when the bandwidth falls too low.

2. It requires a media control server such as an RTSP server to process client-to-server interactivity requests and to track client state like its playout point, whether it is being paused or played for each client session.

3. Many firewalls are configured to block UDP traffic, preventing the users behind these firewalls from receiving videos.

## Streaming Multimedia: HTTP

In HTTP streaming, the video is stored on an HTTP server as an ordinary file with a specific URL. When a user wants to see the video, the client establishes a TCP connection with the server and issues an HTTP GET request for that URL.

The server then sends the video file with an HTTP response message as quickly as TCP congestion control and flow control allow and retransmissions allow.

On the client side, the bytes are collected in a client TCP receive buffer. Once the number of bytes in this buffer exceeds a threshold, the client begins playback, taking them out of the buffer, decompressing the frames and displaying them to the user.

TCP transmission rates can vary significantly due to TCP's congestion control mechanism in a "saw-tooth" manner with the CWND changing and dropping down due to lost packet and timeouts.

The use of HTTP over TCP also allows the video to traverse firewalls and NATs more easily and streaming over HTTP also obviates the need for a media control server. Due to all these advantages, most video streaming applications including YouTube and Netflix use HTTP underlying the streaming protocol.

**Prefetching Video**: Client-side buffering can be used to mitigate the effects of varying end-to-end delays and varying available bandwidth. For streaming stored video, the client can attempt to download the video at a higher rate than the consumption rate, prefetching video frames to be stored in the buffer.

ex. Video consumption rate is 1Mbps, network capable of delivering video from the server to the client at a rate of 1.5Mbps. Then the client will be able to play after a very small playout delay, since it get many frames quickly, but will also be able to increase the amount of buffered video data by 500kb every second.

If in the future the client receives data at a rate less than 1 Mbps, the client will be able to continue to provide continuous playback due to the reserve in its buffer.

Video file -> TCP send buffer -> TCP receive buffer -> TCP application buffer -> read from app buffer, displayed on screen.

Client app reads data from TCP receive buffer and places the bytes into the client application buffer.

When the user pauses the video during the streaming process, bits are not removed from the client application buffer, even though bits continue to enter the client app buffer from the TCP receive buffer. Once the client app buffer becomes full, bytes can no longer be removed from the TCP receive buffer, so it also becomes full.

The server then cannot send any more bytes into the socket, blocking the server until the user resumes the video.

Therefore the server send rate can be no higher than the video consumption rate at the client, else the buffers fill and the rate gets lowered. Therefore the client application buffer indirectly imposes a limit on the rate that video can be sent from the server to the client over HTTP.

## Streaming Multimedia: Dynamic, Adaptive Streaming over HTTP (DASH)

HTTP streaming is widespread, but it has a shortcoming: All clients receive the same encoding of the video, despite large variations in the amount of bandwidth available to a particular client.

DASH overcomes this problem, encoding the video into different versions, with each version having a different bit rate and a different quality level.

The client dynamically requests chunks of video segments of a few seconds in length from different versions.

When the amount of bandwidth is high, the client selects chunks from a high-rate version, and when it drops, it selects from a low-rate version. It selects these different chunks one at a time with HTTP GET queries.

This feature is important for mobile users who typically see their bandwidth fluctuate as they move with respect to base stations.

DASH stores each video version in the HTTP server, each with a different URL. The server also has a **manifest file**, which provides a URL for each version along with its bit rate:

Bit Rate |   Quality   | URL
100mbps     Excellent    http://...
10mbps   |     Good    | https://...

1. The client first requests the manifest file and learns about the various versions.

2. The client then selects one chunk at a time specifying a URL and a byte range in an HTTP GET request message for each chunk.

3. While downloading chunks, the client measures the received bandwidth and runs a **rate determination algorithm** to select the appropriate chunk to request next based on available bandwidth.

Lots of video buffered on the client and high bandwidth = high quality version requested from manifest file.

A sudden drop may be noticeable to the user, so you can do a smooth quality transition dropping chunks from high to low quality progressively.

Since it is the client that is intelligent and monitoring bandwidth to determine what to request, the server remains scalable.

The server also stored many versions of the audio for the same chunk selection by the client.

Client gets to choose:

1. When to request chunk, so that buffer starvation, or overflow does not occur

2. What encoding rate to request, so get the best quality that allows continuous playback

3. Where to request the chunk, can request from a URL with a server that is close to the client

## Content Distribution Networks

How to stream millions of videos to hundreds of thousands of users simultaneously?

Option 1: Can use a mega-server, but it's a single point of failure that has lots of congestion and long paths to distant clients.

Option 2: Store multiple copies of videos at multiple geographically distributed sites (CDN).

2 Types of server distribution techniques:

1. **Enter Deep**: Many access networks, lots of smaller clusters close to users. Akamai has 1700 such locations. Harder to manage all of these distributed clusters.

2. **Bring Home**: Fewer number (10s) of larger clusters at key locations further from users but using very fast links. Lower maintenance and management overhead, at the expense of higher delay and lower throughput to end users.

## CDN Content Access Scenario:

1. User visits the Web page for NetCinema.

2. When the user clicks on the like for video.netcinema/ID, the user's host sends a DNS query for video.netcinema.com

3. The user's local DNS server relays the DNS query to an authoritative DNS server for NetCinema, which observes a CNAME for the subdomain video on the domain NetCinema to a hostname in KingCDN's domain, such as 1105.kingcdn.com and returns it to the user's local DNS.

4. The local DNS contacts the authoritative DNS server for kingCDN, which returns an IP address to the local DNS server where the video is hosted by KingCDN.

5. The local DNS server forwards this IP address of the content-serving CDN node to the user's host.

6. Once the client receives the IP address for the KingCDN server, it establishes a direct TCP connection with the server at that IP and issues an HTTP GET request for the video, or if DASH is used, the server will first send to the client a manifest file with a list of URLS and the client picks.

## Cluster Strategies

In the above example, the CDN learns the IP address of the client's local DNS server via the client's DNS lookup at the CDN's authoritative DNS. After learning the local DNS server address for the client, the CDN needs to select an appropriate cluster based on the IP address.

It has four options for picking the cluster to use:

1. Pick CDN node geographically closest to the client

2. Use characteristics of recent and ongoing traffic between the clients and CDN servers. Pick CDN node with shortest delay or min hops to the client based on having a small number of probe requests pinging between clients and CDN nodes to measure delay and timing. Requires redirecting clients to potentially suboptimal clusters from time to time in order to measure the properties of paths to these clusters.

3. **IP Anycast**: Have the routers in the Internet route the client's packets to the closest cluster as determined by BGP. During the IP-Anycast configuration stage, the CDN company assigns the same IP address to each of its clusters and uses standard BGP to advertise this IP address from each of the different cluster locations.

When a BGP router receives multiple route advertisements for this same IP, it treats these advertisements as providing different paths to the same physical location and will pick the best one based on AS-hop count, then intra-AS hot potato, and so on.

The client's requests for videos are then routed to the best cluster using standard BGP. Cool!

When any client wants to see any video, the CDN's DNS returns the anycast address no matter where the client is located.

4. **Let the client decide**: Give the client a list of several CDN servers and the client pings them, picking the best ping response. This is Netflix's approach.

## Case Study: Netflix

Owns registration, payment servers, but uses Amazon 3rd party cloud services to create multiple versions of movie with different encodings, uploads versions from cloud to CDNs.

Uses three different CDN hosots: Akamai, Limelight, Level-3.

1. Content Ingestion: Before Netflix can distribute a movie to its customers, it must first ingest and process the movie. Netflix receives studio master versions of movies and uploads them to hosts in the Amazon Cloud.

2. Content Processing: The machines in the Amazon Cloud create many different formats for each movie, suitable for a diverse array of client video players running on desktop computers, and at multiple bit rates for DASH.

3. Uploading versions to the CDNS: Once all of the versions of a movie have been created, the hosts in the Amazon cloud upload the versions to the CDNs.

Steps to playing on Netflix:

1. User selects a movie to play and the user's client obtains a manifest file from servers in the Amazon cloud.

2. The manifest file includes a variety of information including a ranked list of CDN and the URLs for the different versions of the movie which are used for DASH playback. The ranked list of CDNs is determined by Netflix and may change from one streaming session to the next. Typically the client selects the CDN ranked highest.

3. After the client selects the CDN, it leverages DNS to redirect the client to the chosen CDN server.

4. The client and CDN server then interact using DASH. Client uses byte-range header in HTTP gET request messages to request chunks. While the chunks are being downloaded, the client measures the received throughput and runs a rate-determination algorithm to determine the quality of the next chunk to request.

## Conversational voice/voice-over-IP (VoIP):

Real-time conversation voice over the Internet.

The Internet is a best-effort protocol and makes no guarantees about getting a packet to a destination within some delay bound or with some percentage of guaranteed packet delivery.

End-to-end Delay Requirement: Needed to maintain convesational aspect.

Higher delays noticeable, impair interactivity.
< 150ms = Good
> 400ms = bad

Session Initialization: How does callee advertise IP address, port number, encoding algorithms?

Value Added Services: call forwarding, screening, recording

Emergency Services: 911

## VoIP Characteristics

Speaker's audio: alternating talk spurts, slient periods.

64kbps during talk spurt. Packets generated only during talk spurts. 20ms chunks at 8KBps = 160 bytes of data.

Application layer header added to each chunk, chunk + header is encapsulated in UDP or TCP, generally UDP.

Application sends segment into socket every 20ms during talk spurt.

### VoIP: Packet Loss

Loss could be eliminated by sending the packets over TCP, which provides RDT. But TCPs retransmission mechanisms are often considered unacceptable for conversational real-time audio applications like VoIP because they increase end-to-end delay.

For this reason most VoIP applications run over UDP by default. UDP is used by Skype unless a user is behind a NAT or firewall that blocks UDP segments in which case TCP is used.

Network Loss: IP datagram lost due to network congestion, router buffer overflow

Delay Loss: IP datagram arrives too late for playout at receiver.

Loss Tolerance: Depending on voice encoding, loss concealment, packet loss rates between 1% and 10% can be tolerated.

### VoIP: End-to-End-Delay:

Accumulation of transmission, processing and queuing delays in routers, propagation delays in links and end-system processing delays. For real time conversational applications like VoIP, need end-to-end delays smaller than 150ms to not be perceived by listener.

Delays exceeding 400ms can seriously hinder the interactivity in voice conversations.

### VoIP: Packet Jitter

Constant bit rate transmission from sender, with variable network delay jitter, and a client playout delay after client reception followed by constant bit rate playout at the client.

Crucial component of end-to-end delay is the varying queuing delays that a packet experiences in the network's routers. Because of the varying delays, the time from when a packet is generated at the source until it is received at the receiver can fluctuate from packet to packet, this fluctuation is called **jitter**.

If the first packet arrives at a router with no other packets, gets to first in the queue and sent quickly. Then between the first and second, many packets arrive and the 2nd one get put behind in the queue and takes longer.

If the receiver ignores jitter and plays out chunks as soon as they arrive, then the resulting audio quality can become unintelligible to the listener.

Jitter can be removed by using **sequence numbers, timestamsp and playout delay**.

## VoIP: Removing Jitter

The receiver should attempt to provide periodic playout of voice chunks in the presence of random network jitter. This is done by combining two mechanisms:

1. Prepending each chunk with a **timestamp** The sender stamps each chunk with the time at which the chunk was generated.

2. **Delaying playout** of chunks at the receiver. The playout delay of the received audio chunks must be long enough so that most of the packets are received before their scheduled playout times. This playout delay can either be fixed throughout the duration of the audio or vary adaptively.

## VoIP: Fixed Playout Delay

Receiver attempts to play out each chunk exactly q milliseconds after the chunk is generated. Therefore if a chunk is timestamped at the sender at time t, when the receiver gets the chunk with timestamp t, it will play it out at time t+q assuming that time hasn't passed by the time it is received. If t+q > now, discard packet.

Tradeoff: Large q = less loss, Small q = More conversational

If large variations in end-to-end delay are typical, preferable to use a large q >150ms, < 400 ms, otherwise if delay is small and variations in delay are also small, use a small q < 150ms.

ex. Sender generates packets at regular intervals of 20ms. The first packet in this talk spurt is received at time r.

For the first playout schedule, the fixed initial playout delay is set to p-r. For the second playout schedule, the fixed initial playout delay is set to p'-r and for this schedule, all packets arrive before their scheduled playout times and there is no loss.

## VoIP: Adaptive Playout Delay

By making the initial playout delay large, most packets can make their deadlines like with initial fixed delay p' but long delays can be bothersome for voice communications.

We would like the playout delay minimized subject to the constraint that the loss be below a few percent.

Can do this by estimating the the network delay and variance at the beginning of each talk spurt. This adaptive adjustment of playout delays at the beginning of the talk spurts will cause the sender's silent periods to be compressed and elongated, but it will ideally not be noticed in speech.

ti = timestamp of the ith packet, the time the packet was generated by the sender
ri = the time packet i is received by the receiver
pi = the time packet i is played by the receiver

The end-to-end delay of the ith packet is ri - ti, and due to network jitter this delay varies.

di = the average network delay upon reception of the ith packet. This estimate is constructed from the timestamps as follows:

di = (1-u)di-1 + u(ri-ti)

Where u is a fixed constant. Then di is the combination of the average network delay plus its own network delay, placing more weight on the recently observed delay than on the ones from the distant past.

Let vi denote an estimate of the average deviation of the delay from the estimated average delay. This estimate is also constructed from the timestamps:

vi = (1-u)vi-1 + u|ri - ti - di|

The estimates di and vi are calculated for every packet received, although they are used only to determine the playout point for the first packet in any talk spurt.

Once calculated, the receiver employs the following algorithm for the playout of packets:

If packet i is the first packet of a talk spurt, its playout time pi is computed as:

pi = ti + di + Kvi

Where K is a positive constant. The purpose of the Kvi term is to set the playout time far enough into the future so tat only a small fraction of the arriving packets in the talk spurt will be lost due to late arrivals.

The playout point for any subsequent packet in a talk spurt is computed as an offset from the point in time when the first packet in the talk spurt was played out:

qi = pi - ti is the length of time from when the first packet in the talk spurt is generated until it is played out. If the jth packet also belongs to this talk spurt, it is played out at pj = tj + qi, so the time it was generated at plus the time we waited between the first one's generation and the first one's playout.

This makes sense, we update the average every time, but after the first of a talk spurt, we add a fixed amount that was calculated for the first packet to each of the successive ones until the next talk spurt.

How does receiver determine whether a packet is first in a talk spurt?

If the system has no loss, receiver can look at timestamps. Different of successive timestamps > 20ms means a talk spurt begins, since the packets within a spurt are sent 20ms apart.

If there is loss in the system, the receiver must look at both timestamps and sequence numbers. Difference of successive time stamps  20ms and sequence numbers without gaps means that a new talk spurt begins.

## VoIP: Recovering from Packet Loss

A packet is lost if it never arrives at the receiver or if it arrives after its scheduled playout time.

Retransmitting is useless, since retransmitting a packet that missed its deadline is useless, and retransmitting one that was lost before it was time to play will likely not get there in time.

VoIP instead uses a loss anticipation scheme called **forward error correction (FEC)** and **Interleaving**.

## VoIP: Forward Error Correction (FEC)

Add redundant information to the original packet stream. For the cost of marginally increasing the transmission rate, the redundant information can be used to reconstruct approximations or exact version of some of the lost packets.

We want to send enough bits to allow recovery without retransmission, like in two-dimensional parity.

Two mechanisms:

1. Send a redundant encoded chunk after every n chunks. The redundant chunk is constructed by XORing the n original chunks. Therefore if any one packet of the group of n+1 chunks is lost, the receiver can fully reconstruct the packet by XORing all other packets with the redundant chunk.

But if two or more packets in a group are lost, the receiver cannot reconstruct the packet. By keeping the group size small, a large fraction of the lost packets can be recovered when loss is not excessive.

Downsides: This scheme increases the playout delay, since the receiver must wait to receive the full group of packets before it can begin playout, otherwise if one of them is lost, it cant correct it because it doesn't have them all.

2. Send a lower-resolution audio stream as the redundant information. The sender might create a nominal audio stream and a corresponding low-resolution, low-bit rate audio stream. The sender constructs the nth packet by taking the nth chunk from the nominal stream and appending to it the n-1 chunk from the redundant stream.

Whenever there is a packet loss, the receiver can then conceal the loss by playing out the low-bit rate chunks encoded with the next packet to arrive.

This works whenever there is non-contiguous packet loss of more than one packet.

The stream of mostly high-quality chunks, occasional low quality chunks and no missing chunks gives overall good audio quality.

In this scheme, the receiver only has to receive two packets before playback, since it can recover a loss from the first with the second before the rest arrive.

This scheme can be generalized and we can send as many redundant previous packets as we want attached to a successive packet.

## VoIP: Interleaving

An alternative redundant transmission, sender resequences units of audio data before transmission so that originally adjacent units are separated by a certain distance in the transmission stream.

If the units are 5ms in length and chunks are 20ms in length, we have 4 units per chunk. Then the first chunk could contain units 1,5,9,13 the first from each original chunk. The second could contain 2,6,10,14 the 2nd unit from each chunk, and so on.

The loss of a single packet would then result in a gap in the reconstructed stream of 5ms per chunk, as opposed to a single large gap that would occur normally.

Interleaving has low overhead since there is no redundant information, but increases playout delay since you need to wait for all packets to arrive and then reconstruct the original stream before you can play.

## Network Support for Multimedia

Three broad approaches towards providing network-level support for multimedia:

1. Making the Best of Best-Effort Service:

The application-level mechanism and infrastructure can be successfully used in a well-dimensioned network where packet loss and excessive end-to-end delay rarely occur.

Granularity: All traffic treatetd equally

Guarantee: None or soft

Mechanisms: Application-layer support, CDNs, overlays, network-level resource provisioning.

Complexity: Minimal

Deployment to Date: Everywhere

2. Differentiated Service:

Different types of traffic, like those indicated in the Type-of-Service field in the IPv4 packet header could be provided with different classes of service rather than a single one-size-fits-all-best-effort approach. Normal and VIP services.

With Differentiated Service, one type of traffic might be given strict priority over another class of traffic when both types are queued at a router. Packets belonging to real-time conversational applications might be given priority over other packets due to their stringent delay constraints.

Introducing differentiated service into the network would require new mechanisms for packet marking, packet scheduling and more.

Granularity: Different classes of traffic treated differently

Guarantee: None, or soft

Mechanisms: Packet marking, policing, scheduling.

Complexity: Medium

Deployment to date: Some

3. Per-Connection Quality-of-Service (QoS) Guarantee:

Moves from all traffic equal, to types of traffic at different priorities, to individual connections at different priorities. So an individual specific connection gets guaranteed bandwidth, rather than a class of communication getting prioritized bandwidth.

Each instance of an application explicitly reserves end-to-end bandwidth and has a guaranteed end-to-end performance. A **hard guarantee** means that applications will receive its requested quality of service (QoS) with certainty.

A **soft guarantee** means that the application will receive its requested quality of service with high probability.

For example, if a user wants to make a VoIP call from host A to host B, the user's VoIP application reserves bandwidth explicitly in each link along a route between two hosts. But permitting applications to make reservations and requiring the network to honor the reservations require some big changes.

Need a protocol that on behalf of the applications reserves link bandwidth on the paths from the senders to their receivers.

Then we need a new scheduling policy in the router queues so that per-connection bandwidth reservations can be honored.

Finally, in order to make a reservation, the applications must give the network a description of the traffic they intend to send into the network and the network will need to police each application's traffic to make sure it abides by that description.

Granularity: Each source destination flow treated differently

Guarantee: Soft or hard, once app makes agreement with network its hard.

Mechanisms: Packet marking, policing, scheduling, call admission and signaling

Complexity: High

Deployment to Date: little

ex. H1 and H2 go through router but there are two queues. Only send from one queue if the higher priority one is empty. They share bandwidth though so if there is constant VIP traffic, then the lower priority queue starves.

## Principles for QoS Guarantees:

Simple network scenario in which two application packet flows originate on hosts H1 and H2 on one LAN and are destined for hosts H3 and H4 on another LAN.

The routers R1 and R2 on the two LANs are connected by a 1.5Mbps link. Assume the LAN speeds are significantly higher than 1.5Mbps and focus on the output queue of R1 that gets a full queue.

Let's further suppose that a 1Mbps audio application shares the 1.5Mbps link between R1 and R2 with an HTTP app downloading a web page from H2 to H4.

In best-effort, the audio and HTTP packets are mixed in the output queue at R1 and are transmitted in FIFO order.

In this scenario, a burst of packets from the web server could potentially fill up the queue, causing IP audio packets to be excessively delayed or lost due to buffer overflow at R1.

Given that the HTTP web browsing app does not have time constraints, we could give priority to audio packets in R1, always transmitting the audio packet before an HTTP packet.

In order for R1 to distinguish between the audio and HTTP packets in its queue, each packet must be marked as belonging to one of these two classes of traffic.

This was the first original goal of the type-of-service ToS field in IPv4.

**Principle 1:** Packet marking needed for router to distinguish between different classes and new router policy to treat packets accordingly.

Suppose that the router is configured to give priority to packets marked as belonging to the 1Mbps audio app. Since the outgoing link speed is 1.5Mbps, even though the HTTP packets are lower priority, they can still on average receive 0.5Mbps transmission.

But if the audio app starts sending packets at a rate of 1.5Mbps or higher then the HTTP packets will starve. Similar problems occur if multiple aggregate higher priority applications cumulatively use more bandwidth than the link can handle, again resulting in starvation.

We need a degree of isolation among classes of traffic so that one class cannot starve the other. This protection could be implemented at different places in the network. At each and every router, first entry to the network, inter-domain network boundaries, etc.

**Principle 2:** It is desirable to provide a degree of **traffic isolation** among classes so that one class is not adversely affected by another class of traffic that misbehaves. Such as in a per-connection QoS system where the application uses more bandwidth than it reserved, or a class system where an audio class is designed for its class not to exceed xMbps.

This can be addressed with **traffic policing**, if a traffic class or QoS flow must meet a certain criteria, like not exceeding a peak rate of 1Mbps then a policing mechanism can be put into place to ensure that the criteria are met.

If the policed application misbehaves, then the policing mechanism will take some action, drop or delay packets in violation so that the traffic actually entering the network conforms to the criteria.

This problem could be addressed by making hard limits on how much each flow can use, for example the audio flow requested 1Mbps and can only get 1Mbps, and if it drops to 0 because no-one is talking right now, the HTTP server is still limited to 0.5Mbps because the audio flow has reserved 0.5Mbps. This is, however, inefficient and allocating fixed, non-shareable bandwidth goes against:

**Principle 3**: While providing isolation, it is desirable to use resources as effectively as possible. If we isolate the audio and HTTP flows using hard limits, that's not using the resources of the link as efficiently as possible, since when one has no traffic, the other cannot use it's reservation.

## Scheduling and Policing Mechanisms

The manner in which queued packets are selected for transmission on the link is called the **link-scheduling discipline**.

**FIFO** = Wait in a queue in order of arrival, if queue fills, then the queue's **Packet-discarding policy** determines whether the packet is dropped or another is removed from the queue.

**Priority Queue** = packets arriving at the output link are classified into priority classes at the output queue. A packet's priority may depend on an explicit marking that it carries in its packet header like the value of the ToS bit in an IPv4 packet, or its IP, port, or other criteria.

Send highest priority queued packet first.

**Round Robin** = Multiple classes, cyclically scan through them, sending one complete packet from each class in descending priority. A work-conserving queue will move on to the next class if there is nothing in one it checks and not wait.

**Weighted Fair Queuing** = Like Round-Robin except each class may specify a differential amount of service in any interval time. Under WFQ, during any interval of time during which there ree class i will be guaranteed to receive a fraction of service equal to its weight as a percentage of all weights.

## Policing: Leaky Bucket

It is important to police classes and applications flows, but what aspects of a flow's packet rate should be policed?

Three important policing criteria:

1. **Average Rate**: Network may wish to limit the long-term average rate of packets per time interval at which a flow's packets can be sent into the network.

2. **Peak Rate**: While the average-rate constraint limits the amount of traffic that can be sent into the network over a relatively long period of time. A peak-rate constraint limits the max number of packets that can be sent over a shorter period of time. The average rate may be 6000 packets per minute, while the peak rate is 1500 packets per second, so no big spikes.

3. **Burst Size**: The network may also wish to limit the max number of packets, the burst of packets, that can be sent into the network over an extremely short interval of time. The burst size limits the number of packets that can be instantaneously sent into the network. Even though it is physically impossible to send them instantaneously, the abstraction is useful.

The leaky bucket mechanism is an abstraction that can be used to characterize these policing limits. A leaky bucket consists of a bucket that can hold up to b tokens. Tokens are added to this bucket as follows:

New tokens which may potentially be added to the bucket are always being generated at a rate of r tokens/second

If the bucket is filled with less than b tokens when a token is generated, the newly generated token is added to the bucket, otherwise the newly generated token is ignored and the bucket remains full with b tokens.

Suppose that before a packet is transmitted into the network, it first must remove a token from the bucket.

If the bucket is empty, the packet must wait for a token. Let us now consider how this behaviour polices a traffic flow:

Because there can be at most b tokens in the bucket, the max burst size for the policed flow is b packets. Because the token generation rate is r, the max number of packets that cen enter the network at any interval of time of length t is rt+b, the number of tokens rt that can be made in the interval, plus b assuming it is full.

Therefore token generation rate, r, limits the long-term average rate at which packets can enter the network.

It takes two buckets to handle peak rate.

## Leaky Bucket + Weighted Fair Queuing = Provable Max Delay in Queue

Consider a router's output link that multiplexes n flows, each policed by a leaky bucket with parameters b_i, r_i from i=1..n

Flow = set of packets that are not distinguished from eachother by the scheduler.

Each flow i is guaranteed to receive a share of the link bandwidth at least its weight as a percentage of all weights * R the transmission rate of the link.

Max delay that a packet will experience while waiting for service in the WFQ? Suppose that flow 1's token bucket is initially full. A burst of b_1 packets then arrive to the leaky bucket policer for flow 1.

These packets remove all of the tokens without wait from the leaky bucket and then join the WFQ waiting area for flow 1.

Since these b_1 packets are served at a rate of at least R * w_i/sum of weights packets per second, the last of these packets will then have a max delay, d_max, until its transmission is completed where

d_max = b_1 / (R * w_1 / sum weights).

ex. If b_1 is 10 packets and R is 10 packets per second, then if bucket 1 has 100% weight, it would send all 10 in 1s, but if it has 50% weight, then it would be able to send 5 of its packets over the interval. So the max d_max is b/R if weight is 1 for that bucket.
