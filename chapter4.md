Forwarding = router-local action of transferring a packet from an input link to an output link. Like going through a single interchange on the highway.

Routing = network-wide process that determines the end-to-end paths that packets take from source to destination. Process of planning the entire driving trip.

**link-layer switches** = base forwarding decision on values in the fields of the link layer frame. Switches are referred to as link-layer devices.

**Routers** = base forwarding decision on the value in the network-layer field.

## Internet architecture

Service Model: Best Effort
Bandwidth Guarantee: None
No-Loss Guarantee: None
Ordering: Any order possible
Timing: Not maintained
Congestion indication: None

## Virtual Circuit and Datagram Networks

Computer architectures at network layer provide either a host-to-host connectionless service or a host-to-host connection service, not both.

Networks that provide only a **connection** service at the network layer are called **virtual-circuit networks (VCs)**.

Networks that provide only a **connectionless**  service at the network layer are called **datagram networks**.

## Parts of a VC

1. Path: series of links and routers between source and destination host
2. VC Numbers: one number for each link along the Path
3. Entries in the forwarding table in each router along the path

Packet belonging to a virtual circuit carries a VC number in its header. VC may have a different VC number on each link, each intervening router must replace the VC number of a traversing packet with a new VC number from the forwarding table.

The incoming VC #'s are used to determine where to send the incoming packet by keeping entries in the forwarding table. VC # 12 comes in, know to send it out on interface 2 and with VC # 22.

Even if there is no VC # changes, there is still connection state information required for associating VC #'s with output interface numbers.

## VC Phases:

* VC Setup: transport contacts network, specifies receiver's address, waits for network to setup VC. Network layer determines path between sender/receiver, the series of links and routers through which all packets of the VC will travel. Network layer also determines VC # for each link along path.

Network layer adds an entry in the forwarding table in each router along the path.

* Data Transfer: once the VC has been established, packets can begin to flow along the VC.

* VC Teardown: Sender informs the network layer of its desire to terminate the VC. Update forwarding tables in each of the packet routers on the path to indicate that the VC no longer exists.

## Datagram Networks:

In a datagram network, each time an end system wants to send a packet, it stamps the packet with the address of the destination end system and pops the packet into the network.

No VC setup and routers do not maintain any VC state information. Packet passes through routers.

Uses packet destination address to forward the packet. Maps destination address to link interfaces.

Router uses **prefix matching** in its forwarding table to map IP addresses to the link interface for the outbound packet.

When there are multiple prefix matches, pick the **Longest**, this is the **Longest prefix matching rule**.

Routers in datagram networks maintain no connection state information, but do maintain forwarding state information in the forwarding table.

Forwarding tables are modified by routing algorithms, which update a table every 1-5min.

## Router Components

1. **Input Ports**: performs the physical layer function of terminating incoming physical link at a router.

Also performs link-layer functions needed to interoperate with the link layer at the other side of the incoming link.

Lookup function is also performed at the input port. It is here that the forwarding table is consulted to determine the router output port to which an arriving packet will be forwarded via the switching fabric.

Control packets, those carrying routing protocol info, are forwarded from an input port to the routing processor.

1) Physical and link layer processing must occur
2) packet's version number, checksum and time-to-live fields must be checked and the latter two rewritten
3) Counters used for network managment such as the number of IP datagrams received must be updated

2. **Switching fabric**: connections the router's input ports to its output ports. This switching fabric is completely contained within the router.

3. **Output Ports**: stores packets received from the switching fabric and transmis these packets on the outgoing link by performing the necessary link-layer and physical-layer functions.

4. **Routing processor**: The routing processor executes the routing protocols, maintains routing tables and attached link state information and computes the forwarding table for the router.

Consider a roundabout:

The input port corresponds to the entry road and entry station that determines where to get off, the roundabout itself is the switching fabric, and the roundabout exit is the output port.

## Switching

Switching fabric switches (forwards) packets from an input port to an output port.

### Switching via memory

Simplest, earliest routers were traditional computers, with switching between input and output ports being done under direct control of the CPU.

Input port would signal the routing processor via an interrupt, packet was then copied from input port into processor memory, and processor extracted the destination address from the header, looked up the appropriate output port in the forwarding table and copied the packet to the output port's buffers.

### Switching via a bus

input port transfers a pakcet directory to the output port over a shared bus without intervention by the routing processor.

done by having the input port prepend a switch-internal header to the packet indicating the local output port to which this packet is being transferred and transmitting this packet onto the bus.

The packet is received by all output ports, but ony the port that matches the header will keep it.

Only one packet can cross the bus at a time, only one car on the roundabout at a time.

### Switching via an interconnection network

Overcome bandwidth limitation of a single shared bus by using a sophisticated interconnection network, like those to connect processors in a multiprocessor computer architecture.

Crossbar switch has 2N buses that connect N input ports to N output ports. Each vertical bar intersects each horizontal bar.

Allows multiple packets to be routed from input to output ports at once provided that they don't require the same buses.

## Output Processing

Takes packets that have been stored in the output port's memory and transmits them over the output link.

## Where is Queuing

Packet queues may form at both the input ports and the output ports. Location and extent of queuing depends on traffic load, speed of the switching fabric, and line speed.

It is here at the queues within a router that a packet can be dropped as a result of memory overflows.

For many years, the rule for buffer sizing at the input/output ports was one RTT x link capacity.

Thus a 10Gbps link with RTT of 0.25 seconds needs 2.5Gb of buffer.

**Packet scheduler** at the output port must choose one packet among those queued for transmission, could use FCFS, or weighted fair queuing.

Similarly, if there is not enough memory to bufer an incoming packet, a decision must be made to either drop the arriving packet, **drop tail**, or remove one or more already queued to make room.

## Internet Protocol (IP): Forwarding and Addressing in the Internet

Lots of fields compose an IP datagram, couple I will record:

**Time to live:** included to ensure that datagrams do not circulate forever. Field is decremented by one each time the datagram is processed by a router. If the TTL is 0, the datagram must be dropped.

Length, Protocol (TCP or UDP), Version, Header length, types (for low delay, high throughput, reliability, etc), Header checksum, Source/Dest Address

Note: checksum must be recomputed and stored again at each router, as the TTL field changes after passing through each router.

Why does TCP/IP perform error checking in both the transport layer segments and network layer datagrams?

Only the IP header is checksummed at the network layer, while the TCP/UDP checksum is computed over the entire TCP/UDP segment.

Also, TCP/IP do not have to be used together, so incase used with one that doesn't checksum, need each individually.

### IP Datagram Fragmentation

If router receives an IP datagram from one link and determines that the outgoing link has an MTU smaller than the length of the IP datagram, then the solution is to fragment the data in the IP datagram into two or more smaller IP datagrams.

Each smaller datagram is referred to as a **Fragment**.

IPv4 puts identification, flag and fragmentation offset fields in the IP datagram header to allow host applications to determine if a datagram received is a fragment. The network protocol does not handle the re-assembly of fragments itself, as this is too burdensome for routers.

Offset field used to determine where the fragment fits in the original datagram.

Flag is set to 1 if not the last, 0 if last to know when done.

## IPv4 Addressing

A host has a single link into the network, when IP in the host wants to send a datagram, it does over this link. The boundary between the host and the physical link is the **interface**.

Because a router's job is to receive a datagram on one link and forward the datagram on some other link, a router necessarily has two or more links to which it connected. The boundary between a router and one of its links is also called an interface.

each host and router has its own IP address. IP addresses are more associated with interfaces, the boundary between the host and the link.

A network that connects host interfaces and one router form a **subnet**. IP addressing assigns an address to this sbunet: 223.1.1.0/24 for example, where the /24 notation is the **subnet mask** indicating that the leftmost 24 bits of the 32-bits define the subnet address.

A router with 3 hosts then could have 4 addresses, 223.1.1.4/24 for the router, and 223.1.1.0-223.1.1.3 for the hosts.

The mask determines the specific host under the router interface.

## Classless Interdomain Routing (CIDR)

CIDR generalizes the notion of subnet addressing, 32-bit IP address is divided into two parts a.b.c.d/x where x indicates the number of bits in the network part of the address. This is the network prefix of the address.

An organization is typically assigned a block of contiguous addresses.

When a router outside the organization forwards a datagram whose destination address is inside the organization, only the leading x bits are considered. Reduces the size of the forwarding tables in these routers, since a single entry a.b.c.d./x will be sufficient for forwarding packets to any destination within the organization.

The remaining bits distinguish devices within the organization. These are the bits that are considered when forwarding packets at routers within the organization.

Before CIDR, network portions of an IP address were constrained to 8, 16, 24 bits, this was **classful addressing** but this was limiting.

## Broadcast Address

IP broadcast address 255.255.255.255, when a host sends a datagram with this destination address, the message is delivered to all hosts on the same subnet.

## Obtaining a Block of Addresses

Networking admin can contact ISP, which would provide addresses from a larger block allocated to the ISP.

ex. ISP has 200.23.16.0/20, gives out:

200.23.16.0/23 to 200.23.30.0/23, so 8 organizations get segments, with 9 bits for their internal subnet hosts.

ISPs get their blocks from ICANN, Internet Corporation for Assigned Names and Numbers.

Manages DNS root servers, allocates IP addresses.

## Dynamic Host Configuration Protocol

Organization can assign individual IP addresses to the host and router interfaces in its organization. Sysadmin typically manually configures the IP addresses in the router.

Host addresses can be manual also, but usually done using DHCP. DHCP can give the same host the same IP each time or different each time it connects.

DHCP is a plug-and-play protocol, automates network-related aspects of connecting a host into a network.

In the simplest case, each subnet has a DHCP server that has a pool of addressses used or available that it assigns to clients.

If no server is present on the subnet, a DHCP relay agent like a router knows the address of a DHCP server for that network is needed.

DHCP protocol is a four-step process:

1. **DHCP Server Discovery:** newly arriving host uses a **DHCP Discover Message** to find a server, UDP packet on port 67. Sent to 255.255.255.255, which gets broadcast to all nodes on the subnet and source address of 0.0.0.0, since it doesn't yet have a proper IP.

2. **DHCP Server Offer:** DHCP server receiving a DHCP discover message responds to the client with a **DHCP offer message** that is broadcast to all nodes on the subnet, using IP address 255.255.255.255 as expected. If multiple servers, client might be able to choose since all servers respond with offers.

Server offer contains the transaction ID of the received discover message so that the host knows if this offer was intended for it, the proposed IP for the client, the network mask, and an **IP address lease time**: the amount of time for which the IP address will be valid, usually hours or days.

3. **DHCP Request:** Newly arriving client will choose from among one or more offers and respond to its selected offer with a **DHCP request message**, echoing back the config params. Must be broadcast to everyone on 255.255.255.255, so that they know if their offer was accepted or not.

4. **DHCP ACK**: The server responds to the DHCP request message with a **DHCP ACK message**, confirming the request parameters. This is also broadcast, since the client cannot use the IP given until this protocol is complete.

DHCP also allows clients to renew their leases.

## Network Address Translation (NAT)

Simple approach to address allocation. The address space 10.0.0.0/8 is one of three portions of the IP address reserved for a private network, a network whose addresses only have meaning to devices within that network.

There are many home networks each using the same address space, 10.0.0.0/24.

Devices within a given home betwork can send packets to eachother using 10.0.0.0/24 addressing, but these addresses cannot be used outside the home network, there would be conflicts.

A NAT router looks like a single device with a single IP address to the outside world.

All traffic leaving a home router has a source IP of 138.76.29.7 for example, hiding the details of the home network.

The router gets its IP from the ISP's DHCP server and acts as a DHCP router for its home network itself.

Uses a **NAT Translation Table** at the NAT router.

ex. Person makes web request to 128.119.40.186 on port 3345, coming from its NAT private network IP of 10.0.0.1.

NAT router receives it, generates a new port number 5001 for the datagram, replaces the source IP of 10.0.0.1 with its own IP address of 138.76.29.7.

NAT router adds a translation for 10.0.0.1, 3345 -> 5001, 138.76.29.7. It uses a different port number for each, so it can map them back.

Some people not pleased, say that port numbers are intended for use by processes, not for hosts.

Problems with P2P, if Peer B is behind a NAT, it cannot act as a server and accept TCP connections directly.

## UPnP

How can external hosts talk to the internal ones? Need to know its NAT address.

Universal Plug-And-Play is a protocol that allows a host to discover and configure nearby NAT. Application in host can request a NAT mapping between its private IP and private port number to its public IP and public port number for some requested number.

If the NAT accepts the request and creates the mapping, then nodes from the outside can initiate TCP connections to that port and IP.

UPnP lets the application know the value of the public IP and port number so that the application can advertise it.

ex. host with UPnP enabled NAT has a private address of 10.0.0.1 and is running BitTorrent on port 3345.

asks NAT router to make a mapping:

10.0.0.1, 3345 -> 138.76.29.7, 5001

external host sends a SYN to 138.76.29.7, 5001. NAT receives this and changes it to 10.0.0.1, 3345 and forwards it to the host.

## Internet Control Message Protocol (ICMP)

Used by routers and hosts to communicate network-layer info to eachother. Most typical use case of ICMP is for error reporting.

When running a Telnet, FTP, or HTTP, may encounter an error like *destination network unreachable*. At some point, IP router was unable to find a path to the host specified in the Telnet, FTP or HTTP app. That router created and sent a type-3 ICMP message to host indicating error.

Often considered part of IP but architecturally lies just above IP< as ICMP messages are carried inside IP datagrams, not in the header.

Parts of an ICMP message:

1. Type
2. Code
3. Header and first 8 data bytes of the IP datagram that caused the ICMP message to be generated.

Ping programs send an ICMP type 8 code 0 message to a specified host. The destination host seeing the echo request, sends back a type 0 code 0 ICMP echo reply.

Traceroute allows us to trace a route from a host to any other host in the world. Traceroute is implemented with ICMP messages.

To determine the names and addresses of the routers between the source and the destination, Traceroute sends a series of ordinary IP datagrams to the destination. Each carries a UDP segment with an unlikely UDP port number.

The first of these datagrams has a TTL of 1, the second 2 and so on.

When the nth datagram arrives at thenth router, the nth router observes that the TTL of the datagram has expired and discards it, sending an ICMP warning message to the source with type 11 code 0, and includes its router name and IP.

Eventually one of the datagrams containing a UDP segment reaches the requested host, using its unlikely port number. The host sends a port unreachable ICMP message type 3 code 3 in response back to the source. The source then knows it can stop sending datagrams.

## IPv6

Running out of 32-bit addresses. Improved header.

IPv6 data format:

fixed length 40-byte header. No fragmentation allowed.

Increases the size of the IP address from 32 to 128 bits.

Components:

1. Flow Labeling and Priority: audio and video transmission could be considered a flow. But traditional applications like file transfer and email might not be considered flows.

Can indicate priority datagrams in a flow.

2. Next Header: identify upper layer protocol for the header, like TCP or UDP.

Other changes:

checksum: removed entirely to reduce processing at each hop
options: allowed but outside of header, indicated by *next header* field.
ICMPv6: new version of ICMP, additional message types like "packet too big", multicast group management functions.

## Transitioning from IPv4 to IPv6

Use tunneling, IPv6 datagram is carried as the payload of an IPv4 datagram among IPv4 routers and then used as IPv6 datagram if going to IPv6.

Otherwise, data like flow identifier field would be lost as the IPv6 is downcast to IPv4.

## IPsec

On the sending side, the transport layer passes a segment to IPsec. IPsec then encrypts the segment, appends additional security fields to the segment and encapsulates the resulting payload in an ordinary IP datagram.

## Routing Algorithms

Host is directly attached to one router, the **default router** or **first-hop  router**.

**global routing algorithm**: computes the least-cost path between a source and desination using complete, global knowledge about the network. The algorithm then needs to somehow obtain this info before performing the calculation. Also called **link-state (LS)** algorithm since it is aware of the cost of each link in the network.

**decentralized routing algorithm**: calculation of the least-cost path is carried out in an interative, distributed manner. No node has complete info. Each node begins only with the knowledge of the costs of its own directly attached links. Called **distance vector (DV)** algorithm.

**static routing algorithm** = routes change very slowly over time, as a result of human intervention, manual forwarding table changes.

**dynamic routing algorithm** = changes the routing paths as the network traffic loads or topology changes.

## LS Link-State Routing Algorithm

All link costs are known, global routing algorithm. Accomplished by having each node broadcast link-state packcets to all other nodes on the network, with each link-state packet containing the identities and costs of its attached links.

Use Dijkstra's.

Can run into problems if it oscillates. Everyone runs the algorithm independently, so they see that one path is low traffic, all go that way. Then the next time they see everyone went that way and everyone goes the other way, oscillations of the group as a result of lack of coordination.

Could randomize the time the routers send out a link advertisement, advertising who it is connected to.

## DS Distance-Vector Routing Algorithm

Each node receives some info from one or more directly attached neighbours, performs a calculation and distributes the results of its calculation back to its neighbours.

This process continues until no more info is exchanged between the neighbours.

The Bellman Ford equation starts at x, and calculates the least-cost path from x to y as the min of x to v, then v to y, so a neighbour of x, and then the least cost from that neighbour to y for all such neighbours of x.

This is used in routing. The solution to the Bellman-Ford equation provides the entries in node x's forwarding table. To see this, let v* be any neighbouring node that achieves the minimum among x's neighbours to y. Then if node x wants to send a packet to y, it should forward the packet to v* first.

x's forwarding table therefore specifies v& as the next-hop in the router for the destination y.

Algorithm:

Each node x begins with $D_{x}(y)$, an estimate of the cost of the least-cost path from itself to node y for all nodes in N.

Each node x maintains the info:

* For each neighbour v, the cost c(x,v) from x to directly attached neighbour v.

* Node x's distance vector from x to all other nodes y in N.

* The distance vectors of each of its neighbours for each neighbour v of x, so the least cost for each of its neighbours to all other nodes y in N.

This last point is achieved by having each node from time to time sending a copy of its distance vector to each of its neighbours. When a node x receives a new distance vector from any of its neighbours v, it saves v's distance vector and then uses the Bellman Ford to update its own distance vector.

taking the min of $D_{x}(y) = c(x,v) + D_{v}(y)$ for all v. If node x's distance vector changes as a result, x will send its updated vector to each of its neighbours, which in turn update their own.

As long as all the nodes continue to exchange their distance vectors asynchronously, each cost D_{x}(y) converges to the actual cost of the least-cost path from x to y determined in Dikstra's.

In the DV algorithm, a node x updates its distance vector estimates when it either sees a cost change in one of its directly attached links or receives a distance-vector update from some neighbour.

## Count to Infinity Problem

The DV algorithm described can run into problems when a link increases its cost. Suppose that in the xyz triangle, the cost from x to y increases from 4 to 60. at time 0, y acknowledges this change and recomputes its paths, using z's cost of 5 to x plus y's cost to z to now get to x. But that way that z had computed for getting to x was through y. So now y thinks it can get to x in 6, so it goes through z.

This is a **routing loop**, y thinks z is its best option to get to x, and z thinks y is its best option to get to x, so they bounce back and forth, increasing the cost by 1 each time since it is cost 1 to get between then, until they get to 50, the cost of z going to x directly, and then z tells y it can get to z in 50, rather than via y, which would continue the routing loop, and then y takes z to x along the 50 path, ending the loop.

This is called the count to infinity problem, because if the cost of z to x had been much higher like 10000, the routing loop would have continued for a long time.

## Poisoned Reverse

To fix this problem, use a process called poisoned reverse. It's simple, if z routes to x via y, then it tells y its distance to x is infinity, since it must go through y to get there. It still tells everyone else it can to get x in 5, since it can use y, but for anyone whose best way to somewhere includes them, it tells them it cannot get there.

In our particular example, now y's cost to x increases to 60, and it still picks this route to x, since z's cost to x appears as infinity. It then advertises this to z, who sees a cost of 61 to x now through y and chooses instead its direct link to x of 50. It advertises this to y, who now chooses to go through z to x with a cost of 51.

This solves a 2-node cost to infinity problem but the issues persists for routing loops of 3 or more nodes.

## Hierarchical Routing

Study of LS and DV has assumed a network as a system of interconnected routers. We assumed all routers execute the same routing algorithm to compute routing paths through the entire network.

This does not work for 2 reasons:

1. **Scale**: The number of routers becomes large, the overhead involved in computing, storing and communicating routing info with LS updates is huge. Recall that LS systems like Dijkstras have the shortest path calculated for each node at all times, that's hundreds of millions of references in each node's routing tables. too much.

2. **Administrative Autonomy** = An organization should be able to run and administer its network while still being able to connect its internal network to the other external networks.

## Autonomous System (AS)

Solve this with ASs. Each AS consists of a group of routers that are typically under the same admin control, operated by the same ISP or belonging to the company network.

Connect ASs to eachother, and have one or more routers in an AS acting as gateway routers to the outside.

How does a router within some AS know how to route a packet to a destination that is outside the AS, which gateway router does it send it to?

AS1 needs to learn which destinations are reachable via AS2 and which destinations are reachable via AS3 and propagate this reachability information to all of its routers so that each router can configure its forwarding table to handle external AS destinations.

This is handled by an **inter-AS routing protocol**.

In the Internet, all ASs run the same **inter-AS routing protocol** called **BGP4**.

If subnet x is reachable from both AS2 and AS3, which are reachable from AS1 via router 1b our 1c, then router y which is looking to send data to x can use **hot potato routing**, part of the inter-AS protocol.

In **hot potato routing**, the AS gets rid of the packet (the hot potato) as inexpensively as possible. This is done by having y route the packet to the cheapest gateway router that can reach x and adding an entry in its forwarding table for subnet x, to link z, the link leading to the cheapest gateway router going to x.

If AS1 learns from AS2 that subnet x is reachable from AS2, it can then tell AS3 that x is reachable from AS1. AS3 could then forward a packet to AS1 which would in turn forward the packet to AS2 and on to x.

Steps in inter-AS routing:

1. All routers in AS1 learn from inter-AS protocol that subnet x is reachable via multiple gateways

2. Use routing info from intra-AS protocol to determine the costs of least-cost paths to each of those gateways.

3. Hot potato routing: choose the gateway that has the smallest cost.

4. Determine the forwarding table entry for the interface leading to that least cost gateway and add it to the table.

Choosing who and what it wants to tell other ASs is a **policy decision**.

## Routing in the Internet

Two intra-AS routing protocols:

1. **Routing Information Protocol (RIP)**:

DV protocol that uses hop count as a costs metric, from a source router to a destination subnet, it has cost >=1. **hop** is the number of subnets traversed along the shortest path from the source router to the destination subnet, including the destination subnet.

The max cost of a path in RIP is limited to 15, therefore RIP systems are fewer than 15 hops in diameter.

AS a DV protocol, RIP routers exchange distance vectors with neighbours to evaluate the least cost distance from each router to a subnet. Routing updates are exchanged every 30 seconds using a **RIP response message/advertisement.** Contains a list of up to 25 destination subnets within the AS as well as the sender's distance to each of those subnets.

Each router maintains a RIP table called its routing table. The routing table contains both the router's distance vectors and the router's forwarding table.

The routing table has 3 columns for each entry:

1. The destination subnet
2. The next router along the shortest path to the destination subnet
3. The cost, the number of hops to the destination subnet.

If a RIP router does not hear form its neighbour at least once every 180s, that neighbour is considered to no longer be reachable. RIP modifies the local routing table and propagates this info by sending advertisements to its neighbour routers.

Routers can also send RIP request messages requesting info about its neighbour's cost to a given destination. RIP messages are sent using UDP segments. Weird how a network layer functionality like RIP relies on an transport layer protocol like UDP.

In fact, RIP is implemented as an application layer protocol running over UDP and is an application process called *routed*.

2. **Open Shortest Path First (OSPF)**:

Another intra-AS routing protocol, deployed in upper-tier ISPs where RIP is used in lower-tier ISPs and enterprise networks.

It is a link-state LS protocol that uses flooding of link-state info and a Dijkstra least-cost path algorithm. With OSPF, a router constructs a complete topological map, a graph of the entire autonomous system.

The router then locally runs Dijkstra's algorithm to determine a shortest-path tree to all subnets, with itself as the root.

Link costs are configurable, might choose to set them to 1, achieving minimum-hop routing (dont care about weighted distances, just the minimum number of hops required) or might choose to set the link weights to be proportional to capacity or to discourage traffic on certain links.

A router broadcasts routing info to all other routers in the autonomous system, not just to its neighbouring routers. Broadcasts LS info whenever there is a change in a link's state or every 30 min.

Contained in OSP messages that are carried directly by IP, no TCP or UDP so it must itself implement functionality such as RDT.

Advantages of OSPF:

1. Security: Exchanges between OSPF routers like LS updates can be authenticated, allowing only trusted routers to be included in the OSPF protocol within an AS, preventing malicious intrusions.

2. Multiple same-cost pathways: When multiple pathways to a destination have the same ccost, OSPF allows multiple pathways to be used, since it doesn't pick one lowest, but maintains a complete topological map that it can send divide traffic on.

3. Integrated support for unicast and multicast routing: Multicast OSPF, MOSPF, provides a simple extension to OSPF to provide for multicast routing.

4. Support for hierarchy within a single routing domain: Ability to structure an autonomous system hierarchically. To be discussed.

An OSPF autonomous system can be configured hierarchically into areas. Each area runs its own OSPF link-state routing algorithm, with each router in an area broadcasting its link-state to all other routers in that area.

Within each area, one or more **area border routers** are responsible for routing packets outside the area. Additionally, exactly one OSPF area in the AS is configured to be the **backbone area**.

The backbone area routes traffic between the other areas in the AS. The backbone always contains **all** area border routers in the AS and may contain non-border area routers as well for intra-area routing through the backbone to the area border router that is in the destination area.

## Inter-AS Routing: Border Gate Protocol (BGP)

BGP4 is the de facto standard of inter-AS routing in today's Internet.

Provides each AS a means to:

1. eBGP (external BGP): Obtain subnet reachability info from neighbouring AS

2. iBGP (internal BGP): Propagate the reachability info to all routers internal to the AS

3. Determine the good routes to subnets based on the reachability info and AS policy

Most importantly, BGP allows each subnet to advertise its existence to the rest of the Internet. A subnet declares that it exists and BGP makes sure that ll ASs in the Internet know about the subnet and how to get there.

## BGP Basics

Extremely complex. Glues the entire Internet together.

In BGP, pairs of routers (peers) exchange routing information over semi-permanent TCP connections using port 179.

There is usually one eBGP TCP connection for each link that directly connects two routers in two different ASs and many iBPG TCP connections between routers in the AS, creating a mesh of TCP connections within each AS.

The TCP connection and all the BGP messages sent over it is collectively called the **BGP session.**

Destinations are not hosts but are instead CIDRized prefixes, with each prefix representing a subnet or a collection of subnets.

for example if AS2 had 4 subnets, 138.16.64/24, 138.16.65/24, 138.16.66/24, 138.16.67/24 then AS2 could aggregate the prefixes for these four subnets and use BGP to advertise the single prefix 138.16.64/22 to AS1.

## Distribute Prefix Reachability

from AS1 to AS2, eBPG would be used over the session between the gateways routers from each AS to send the list of prefixes that are reachable from AS2, and vice-versa.

They then use their iBGP sessions to distribute the prefixes to the other routers in the AS. All routers in AS1 learn about AS2 prefixes as well as its own gateway router. Any other gateway routers with other eBGP sessions can then advertise to them as well with the new prefix information. When a router in AS1 learns about a new prefix, it creates an entry for the prefix in its forwarding table.

## Path Attributes and BGP Routes

In BGP, an AS is identified by its globally unique **autonomous system number (ASN)**. When a router advertises a prefix across a BGP session, it includes with the prefix a number of BGP attributes. A prefix combined with its attributes is called a **route**.

BGP peers advertise routes to each other with the following attributes:

1. **AS-PATH**: Contains the ASs through which the advertisement for the prefix has passed. When a prefix is passed into an AS, the AS adds its ASN to the AS-PATH attribute.

ex. If 138.16.64/24 is first advertised from AS2 to AS1, if AS1 then advertises the prefix to AS3, the AS-PATH would be AS2 AS1. Routers use the AS-PATH attribute to detect and prevent looping advertisements and if a router sees that its own AS is contained in the path list, it rejects the advertisement.

2. **NEXT-HOP**: The IP address of the interface that begins the AS-PATH. So the IP address is in an adjacent AS that will be the first in the path from the AS where you are to your destination AS. You use this IP to calculate the least cost path to the gateway router leading to that IP in the adjacent AS.

ex. When gateway router 3a in AS3 advertises a route to gateway router 1c in AS1 using eBGP, the route includes the advertised prefix x and an AS-PATH to the prefix. This ad also includes the NEXT-HOP, which is the IP address of 3a router that is the gateway router with the eBGP link to the next AS.

Now when router 1d learns about this route to x over iBGP from 1c, router 1d may want to forward packets to x along this route, and adds an entry (x, l) in its forwarding table where l is its interface that begins the least-cost path from 1d towards the gateway router 1c.

To determine l, 1d provides the IP address in the NEXT-HOP attribute, the IP of 3a, to its intra-AS routing module, which will use an intra-as protocol like RIP or OSPF to figure out the next subnet to go to on the path out of the subnet.

The intra-AS routing algorithm has determined the least-cost path to all subnets attached to the routers in AS1, so 1d receives the least cost path 1d to the 1c-3a subnet, leading out of the AS  on the route to the desired AS and it adds x and this interface as (x, l) in its forwarding table.

3. **BGP preference attributes**: Includes attributes that allow routers to assign preference metrics to routes, and an attribute that indicates how the prefix was inserted into BGP at the origin AS.

When a gateway router receives a route advertisement, it uses its **import policy** to decide whether to accept or filter the route and whether to set certain attribute such as the router preference metrics.

The import policy may filter a route because the AS may not want to send traffic over one of the ASs in the route's AS-PATH or because it already knows a preferable route to the same prefix.

## BGP Route Selection

BGP uses eBGP and iBGP to distribute routes to all the routers within ASs. A router may learn about more than one route to any one prefix, in which case the router must select one using the following elimination rules until one route remains:

1. The local preference value of a route could have been set by the router or could have been learned by another router in the same AS. This is a policy decision that is left up to the ASs network admin. Route with the highest local preference value is selected.

2. From the remaining routes tied for highest local preference, the route with the shortest AS-PATH is selected. If this was the only rule for route selection, then BGP would be using a DV algorithm for path determination, where the distance metric uses the number of AS hops rather than the number of router hops.

3. From the remaining routes, all with equal local preference and AS-PATH length, the route with the closest NEXT-HOP router is selected. Here, closest means the router for which the cost of the least-cost path determined by the intra-AS routing algorithm is the smallest. This is **hot potato routing** in action.

4. If more than one route still remains, the router uses BGP identifies to select the route.

## BGP Messages

Message types:

1. OPEN: Opens TCP connection to peer and authenticates sender

2. UPDATE: Advertises new path or updates old

3. KEEPALIVE: Keeps connection alive in absence of UPDATEs, also ACKs OPEN requests.

4. NOTIFICATION: reports errors in previous message, also used to close connection.

## Routing Policy

Example of BGP tied together:

Six interconnected ASs, A, B, C, W, X, Y.

W, and Y are **stub networks**, networks where all traffic entering the network is destined for it, and all traffic leaving it originated at it. Essentially a leaf AS.

X is a **multi-homed stub network**, since it is connected to the rest of the network via two different providers.

To ensure that X, a multi-home connected to both providers B and C, doesn't forward info from B to C, since it is supposed to be a stub, X will only advertise that it has pathways to itself.

Even if X knows of a path to XCY, it will not advertise to B that it knows how to get to Y, because then B might try to take BXCY and X would not be a stub, since traffic leaving X did not originate from it.

Now consider AS B. B has learned from A that A has a path AW to W. B can therefore install the route BAW into its routing info. B wants to advertise the path BAW to its customer, X so that X knows it can route to W via B. But should B advertise the path BAW to C? Then C could route traffic to W via CBAW. If A, B and C are all backbone providers, then B might not want to take on the burden of routing C to its customer w via CBAW when C could just do it itself via CAW. Why is that B's job?

There are no official standards on how backbone ISPs should route among themselves, but a rule of thumb is that any traffic flowing across an ISPs backbone network must have either a source or destination or both in a network that is a customer of that ISP, otherwise the traffic would be getting a free ride on the ISP's network.

These agreements are negotiated between pairs of ISP and are confidential.

## Getting an Entry in the Forwarding Table

1. Router learns about prefix from BGP message containing routes. Router may receive multiple routes for same prefix, uses BGP route selection to pick, such as based on shortest AS-PATH

ex. Path:

AS2 AS17 to 138.16.64/22 or AS3 AS131 AS201 to 138.16.64/22 we want to get to that IP.

2. Router determines output link for that prefix via intra-AS algorithm like OSPF.

ex. AS-PATH: AS2 AS17, NEXT-HOP: 111.99.86.55

We are currently in AS1 at router 1c and the next-hop is the IP of the router interface in AS2, the router interface for the AS that begins the AS-PATH. Router then uses OSPF to find the shortest path from 1c to 111.99.86.55 and we put in the first interface along that intra-AS path to the forwarding table for 1c.

If there are two best inter-AS routes, so we have an AS-PATH of AS2 AS17 or AS3 AS17, same distance, then we use the policy rule of least cost intra-AS route, so the hot potato routing, from 1c to the gateway router leading to AS2 and AS3 respectively. Whichever one is the smaller cost we choose to use as our intra-AS path to the next AS.

## Summary of Steps for Adding Entries in the Forwarding Table

1. Router becomes aware of prefix via BGP route advertisements from other routers.

2. Determines router output interface for prefix, use BGP route selection to find best inter-AS route. Use OSPF to find best intra-AS route leading to best inter-AS route.

3. Enter (prefix, port) entry into forwarding table.

## Why Different intra/inter-AS Routing

1. Policy:

* Inter-AS: Admins wants control over who routes through its net.

* Intra-AS: Single admin, so no policy decisions needed.

2. Scale:

* Hierarchical routing saves table size, reduced update traffic

3. Performance:

* Intra-AS can focus on performance

* Inter-AS may care more about policy sometimes than performance

## Broadcast and Multicast Routing

In **broadcast routing** the network payer provides a service of delivering a packet sent from a source node to all other nodes in the network.

**multicast routing** enables a single source node to send a copy of a packet to a subset of the other network nodes.

Simplest way to broadcast is **N-way-unicast**, make N copies and send to all N destinations.

This is inefficient, if the source node is connected to the rest of the network via one link, then N separate copies of the same packet will traverse the same link, which is not smart. Could just send one and tell the router on the other side to then duplicate it, reducing traffic on the first link.

It would be more efficient for the network nodes themselves, rather than usts the source node, to create duplicate copies of a packet.

Another drawback is that you'd need the IP of each of the N destinations, that's a lot of DNS and ARP.

## Uncontrolled Flooding

**Uncontrolled Flooding** = Node receives broadcast packet, sends a copy to all neighbours.

If the graph has cycles, then copies of the broadcast packet will cycle indefinitely.

Even without cycles, the broadcast duplication can get out of hand, if node x has many neighbours, and each of those have many neighbours, then duplication creates a **broadcast storm**.

## Controlled Flooding

**Controlled Flooding** = Node chooses when to send flood a packet. Node only broadcasts a packet if it hasn't broadcast that packet before.

1. Can be achieved with **sequence-number controlled flooding** where a source node puts its address or identified as well as a **broadcast sequence number** into a broadcast packet and then sends the packet to all of its neighbours.

Each node maintains a list of the source address and sequence nubmer of each broadcast packcet it has already received, duplicated and forwarded. When a broadcast packet arrives, it first checks if the packet is in it list and drops it if so.

2. Second approach to controlled flooding is **reverse path forwarding (RPF)**, when a router receives a broadcast packet with a given source address, it only floods if the packet arrived on the shortest unicast path back to its source.

It can drop it otherwise since it knows that it either has or will receive the packet on the shortest path.

## Spanning Tree Broadcast

While sequence-number-controlled flooding and RPF avoid broadcast storms, they still have many redundantly transmitted packets. Every node should receive only one copy of the broadcast packet.

A spanning tree would satisfy this requirement, a tree containing every node with no cycles.

A minimum spanning tree would send to all nodes once, with minimal cost.

Main complexity comes from the creation and maintenance of the spanning tree.

In the **center-based approach**, a center node is defined. Nodes then unicast tree-join messages addressed towards the center until it either arrives at a node that already belongs to the spanning tree or arrives at the center.

The path that the tree-join message has followed defines the branch of the spanning tree between the edge node that initiated the tree-join and the center.

## Multicast Routing

A multicast packet is delivered to only a subset of network nodes, like in a software upgrade where you want to send it to everyone who needs the upgrade.

Need to identify the receives of a multicast packet and address the packet to those receivers.

Uses **address indirection** = a single identifier is used for the group of receivers and a copy of the packet that is addressed to the group using this single identifier is delivered to all of the multicast receivers associated with that group.

This single identifier is called a D multicast IP address, and the receivers associated with a class D address is a **multicast group**.

Multicast consists of two components:

1. **IGMP (Internet Group Management Protocol)**:

Operates between a host and its directly attached/first hop router. IGMP provides the means for a host to inform its attached router that an app running on the host wants to join a specific multicast group.

IGMP messages are encapsulated in an IP datagram.

IGMP has 3 message types:

1. **Membership query**: message is sent by a router to all hosts on the LAN to determine the set of all multicast groups that have been joined by the hosts on that interface.

2. **Membership report**: hosts sends a report when responding to a query or when first joining a multicast group without waiting for the query.

3. **Leave group**: Optional, the router infers that a host is no longer in the multicast group if it no longer responds to a query message with a report.

2. **Multicast Routing Protocols**:

The goal of multicast routinng is to find a tree of links that connect all of the routers that have attached hosts belonging to the multicast group.

The tree may contain routers that do not have attached hosts belonging to the multicast group, since there may be no way to connect the hosts are do belong without including some that don't.

Two approaches:

1. **Multicast routing using a group-shared tree**: Build a tree that includes all edge routers with attached hosts belonging to the multicast group. Center-based approach is used to construct the multicast routing tree like talked about above.

2. **Multicast routing using a source-based tree**: Construct a multicast routing shortest-path tree for each source in the multicast group = dijkstra's for each source. An RPF algorithm with source node x is used to construct a multicast forwarding tree for multicast datagrams originating at source x.

The RPF broadcast algorithm requires tweaking for this multicast approach. Under broadcast RPF< it would forward packets to router G from D, even though G has no attached hosts in the multicast.

G receives an unwanted packet, the solution to the problem of receiving unwanted multicast packets is called **pruning**.

**pruning**: A multicast router that receives multicast packets and thas has no attached hosts joined to that group will send a prune message to its upstream router, informing it that it does not need to send multicast packets down to it.

If a router receives prune messages from each of its downstream routers, then it can forward a prune message upstream.
