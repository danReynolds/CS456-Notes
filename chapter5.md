# Link Layer

Link layer services:

1. Flow Control: Pacing between adjacent sending and receiving nodes

2. Error detection: Errors caused by signal attenuation, noise. Receiver detects presence of errors: signals sender for retransmission or drops frame.

3. Error correction: Receiver identifies and corrects bit errors without resorting to retransmission.

4. Half-duplex and full-duplex: with half-duplex, nodes at both ends of link can transmit, but not at the same time.

**adapter** = network interface card, one in each host. Ethernet card 802.11 card. Attaches into host system's bus.

**Sending Adapter:**

Encapsulates datagram in frame, adds error-checking, flow control, rdt, etc.

**Receiving Adapter:**

Looks for errors, rdt, flow control. Extracts datagram, passes to upper layers on receiver side.

## Error Detection and Correction

At the sending node, data D is protected against bit errors with error-detection and correction bits (EDC). Typically the data to be protected includes not only the datagram passed down from the network layer for transmission but also the link-level addressing info in the frame header.

Receiver receives D' and EDC' and must determine if they've changed.

## Parity Checks

## Single parity bit

detects errors. Suppose that D has d bits. In an even parity scheme, the sender includes one additional bit and chooses its value such that the total number of 1s in the d+1 bits is even.

If receiver looks and sees that the total number of 1s is odd in the d+1 bits, know that there was an error.

If the probability of errors is small enough, only need a system like this that is best at detecting a single error.

## Two-Dimensional Parity

the d bits in D are divided into i rows and j columns. A parity value is computed for each row and for each column. The resulting i+j+1 parity bits comprise the error-detection bits.

With the two-dimensional parity scheme, a single-bit error is correctable. The parity of a both a column and row would be incorrect, and taking these two together can find specific bit that is wrong and fix it.

## Checksumming Methods

**Internet Checksum**:

Sender:

Break the data into segments of 16-bits. Sum them together and take the 1's complement (flip the bits) and include that result as the checksum. Place checksum in the header.

Receiver:

Sum all the 16-bits like the sender did, but the receiver's will also include the checksum, so when checksum is summed with the rest, you should get all 1's since it was the 1's complement.

If it is not all 1's, know that there is an error.

## Cyclic Redundancy Check

CRC codes are used widely today.

d-bit piece of data D. The sender and receiver agree on a r+1 bit pattern called the generator, denoted G.

For a given piece of data, D, the sender will choose r additional bits, R and append them to D such that the resulting d+r bit pattern is exactly divisible by G mod 2.

Error checking is simple:

the receiver divides the d+r received bits by G, if there is a non-zero remainder, the receiver knows an error has occurred.

Given D and R, the quantity D * 2^r XOR R yields the d+r bit pattern we want. It concatenates R onto the end of D.

So we want D * 2^r XOR R = nG, meaning that it is divisible by G for some integer n.

Therefore D * 2^r = nG XOR R.

This equation tells us that if we divide D * 2^r by G, the value of the reaminder is precisely R.

Therefore R = remainder (D * 2^r) / G

ex. D = 101110, d = 6, G = 1001, r = 3

R = remainder (D * 2^r)  / G = 101110000 / 1001 = 011

So then we send 101110000 XOR 000000011 = 101110011

## Multiple Access Link Protocols

Ideally, we want a access protocol that performs such that given bandwidth R, with one node wants to transmit, it can transmit at rate R and if n nodes want to transmit, they can transmit at rate R/n.

## Channel Partitioning Protocols

**TDMA: Time Division Multiple Access** = Each node gets a time slot. 6 nodes, 6 slots, if 3 want to transmit, their 3 slots get used and the other 3 remain unused. No collisions. Perfectly fair.

But a node is limited to a rate of R/n, not very good if lots of nodes and only 1 ever wants to transmit.

Also a node must wait for its turn to transmit, even when noone else is there.

**FDMA: Frequency Divison Multiple Access** = Channel spectrum divided into frequency bands. Each station assigned fixed frequency band.

Break a frequency into 6 bands, you are always allowed to transmit on band 1, and noone else, so no collisions, fair, and can transmit whenever you want but still limited to R/n rate.

**CDMA: Code Division Multiple Access** = CDMA assigns a different code to each node. Each node then uses its unique code to encode the data bits it sends. If the codes are chosen well, CDMA can have different nodes transmit simultaneously and receivers correctly receive data.

All the spectrum gets used at all times, used in cell networks.

## Random Access Protocols

When a node has a packet to send, transmits at full channel data rate R. Does not know what other nodes are doing.

Get collisions. Need to detect and recover from collisions: Slotted ALOHA, ALOHA, CSMA, CSMA/CD.

**Slotted ALOHA** = All frames consist of L bits, time is divided into slots of size L/R, the time to transmit one frame.

Nodes start to transmit frames only at the beginning of time slots. The nodes are synchronized so that each node knows when the slot begins.

If two or more frames collide in a slot, then all the nodes detect the collision event.

If there is a collision, the node flips a weighted coin with probability p and if p, it retransmits, else it waits and tries again in the next time slot.

Decentralized, simple, can have rate R if only one node.

Efficiency:

With n nodes, if they all want to transmit, the probability after a collision that you will be able to successfully transmit is p(1-p)^n-1 since you need all the others to flip opposite you.

so for all N, the efficiency is the p that maximizes Np(1-p)^n-1

Max efficiency is 1/e = 37%.

**Unslotted ALOHA** = Simpler, no synchronization, no slots. Without synchronicity, a frame can have a collision not only at the start with another frame, but at 2*the size of transmitting the frame, since it could collide with a frame that is already transmitting, or get collided with later while it finishes transmitting by a new frame.

If it collides, does the same p probability coin toss, and if it fails it waits for a frame transmission time.

Efficiency:

Because it needs two frames of space to ensure it transmits, its efficiency is much lower at (1-p)^N-1 for each of the two frame time durations = (1-p)^2(N-1).

This has efficiency 1/(2e) = 18.5% or half the efficiency of Slotted ALOHA.

**Carrier Sense Multiple Access with Collision Detection (CSMA/CD)**: Listen before transmitting, if channel is sensed to be idle, transmit entire frame, otherwise defer transmission for the length of a frame.

Listens before speaking = Carrier Sensing
If someone else talking, stop talking = Collision Detection

Even though listening, can still get collisions due to **channel propagation delay**, takes time for bits to propagate on the wire, and there is a race condition where another node could check if anyone's transmitting as you start.

It uses collision detection so that when a collision is detected, both parties stop transmitting their now corrupted frame.

After a collision, both nodes wait a fixed amount of time. The amount of time should be variable, short when the number of colliding nodes is short and long when the number is large.

**binary exponential backoff** = used in Ethernet, a transmitting frame waits K * 512 bit times, the time to send 512 * K bits, where K is randomly selected from {0, 1, 2, .. (2^n)-1} and n is the number of times it has collided. N is capped at 10.

The more collisions experienced by a frame, the greater the variety of intervals it waits, so if 2 keep colliding, there is a greater probability with each successive attempt that they won't collide.

Steps:

1. Network interface card (NIC) receives datagram from network layer, creates frame

2. If NIC senses channel idle, starts frame transmission. If NIC senses channel busy, waits until channel idle by waiting until it senses no signal energy and then then starts to transmit the frame.

3. If NIC transmits entire frame without detecting another transmission, NIC is done with frame, waits for another.

4. If NIC detects another transmission while transmitting, aborts and sends jam signal.

5. After aborting, NIC enters **binary exponential backoff** and then goes back to step 2.

Efficiency:

If only one node sending, can send at the full rate.

Long term efficiency is $\frac{1}{1+\frac{5d_{prop}}{d_{trans}}}$

where $t_{prop}$ is the max prop delay between 2 nodes in the LAN and $t_{trans}$ is the time to transmit a max-size frame.

## Taking Turns Protocols

Channel partitioning protocols like TDM, FDM share channels efficiently and fairly at high loads while random access protocols like the ALOHAs and CSMA are efficient at low loads, allowing single nodes to fully utilize the channel.

Taking turns protocols try to achieve the best of both:

**Polling** = Master node invites slave nodes to transmit in turn. It polls them to see if they have anything to say.

It first tells node 1 that it can transmit up to the max number of frames, and then moves on either if node 1 says it's all good, or when node 1 has finished which the master can determine by watching what node 1 transmits until node 1 stops.

Eliminates the collisions and wasted slots from random access protocols, and is faster than TDM for example, since it doesn't wait for everyone to take their turn, even if they don't do anything, it just asks everyone if they want to take their turn so while there is still overhead in the time it takes to ask everyone, faster than not asking and waiting no matter what.

It adds a **polling delay**, the time it takes to notify a node that it can transmit.

And if the master node fails, everything is fucked. Bluetooth is polling.

**Token-passing protocols** = No master node, a small frame called a token is exchanged among the nodes in a fixed order. Only the node with the token can transmit, if it has something to transmit it holds it until done and then passes it on, otherwise it passes it on right away.

Decentralized, efficient, but if the node with the token dies, bad things happen.

## Cable Access Network

Data-Over-Cable Service Interface Specifications (DOCSIS) specifies the cable data network architecture and its protocols.

A cable access network connects thousands of residential cable modems to a cable modem termination system (CMTS) at the cable network headend.

DOCSIS uses FDM to divide the downstream (CMTS to modem) and upstream (modem to CMTS) network segments into multiple frequency channels.

Frames transmitted on the downstream channel by the CMTS are multiplexed to the modems, so no collision.

But frames transmitted on the upstream are going from many to few back to the CMTS and collisions must be handled:

Each upstream channel broken up by FDM is also divided by TDM into time intervals, each containing a sequence of mini-slots during which cable modems can transmit to the CMTS.

CMTS tell which modems which TDM slots they can have based on requests. The cable modems send slot request frames to the CMTS during a special set of interval slots that are dedicated for this purpose.

The requests are transmitted using random access protocols so collisions occur and modems cannot use carrier sensing or collision detection.

Instead, the cable modem infers that a slot request frame experienced a collision if it does not receive a response to the requested allocation in the next downstream control message.

When a collision is inferred, the modem uses binary exponential backoff to defer its retransmission of its request frame to a future time slot.

A cable access network has FDM, TDM, random access and centrally allocated time slots all within one system.

## Ethernet

Dominant wired LAN tech. Cheap $20 for NIC. Kept up with speed requirements over years.

**Bus** = popular through 90s, all nodes in same collision domain branched off of one line.

**Star** = prevails today, active switch in the center.

Ethernet Frame:

Preamble | Dest Addr | Source Addr | Data (payload) | CRC

preamble: 7 bytes with pattern 10101010 followed by one byte with pattern 10101011 used to synchronize receiver and sender clock rates.

addresses: 6 byte source, destination MAC addresses.

type: indicates higher layer protocol, usually IP.

CRC: redundancy check at receiver, frame is dropped if error detected.

Ethernet is:

1. Connectionless: No handshaking between sending and receiving NICs

2. Unreliable: Receiving NIC doesn't send acks or nacks to sending NIC. Date is RDT only if sender uses higher layer RDT like in TCP

3. Ethernet's MAC protocol: unslotted, CSMA with binary backoff.

## Ethernet Switch

A switch is transparent to the hosts and routers in the subnet and is plug-and-play, requiring no configuration.

**Filtering**: Switch function that determines whether a frame should be forwarded to some interface or just dropped.

**Forwarding**: Switch function that determines the interface to which a frame should be directed and then moves the frame to those interface.

Switch filtering and forwarding are done using a **switch table** present in the switch.

An entry in the switch table maps a MAC address to an interface leading towards the host with that MAC address.

Hosts have a direct connection to the switch, switch buffers packets. Ethernet protocol is used n each switch's link, which are separate from eachother and do not have collisions between each other.

How it works:

1. There is no entry in the table for MAC x. The switch broadcasts the frame over all of its interfaces.

2. There is an entry in the table for MAC destination x with interface y. The frame needs to be forwarded to interface y.

Self-Learning:

If the source MAC address in the frame is not in the switch table, a new entry is added <source MAC, arriving interface, current time>

The switch table starts empty and adds mappings as it goes and deletes a mapping in the table if no frames are received with that address as its source address after some period of time.

## Switches vs Routers vs Hubs

* Router: **Network Layer**, handles IP packets and more complex than switch, able to do NAT, DHCP, WAP, and other functions.

* Switch: **Link Layer**, handles Link layer frames, has a switch table of MAC address to interface mappings. Table starts empty, self-learning.

* Hub: **Physical Layer**, not smart, just replicates every bit of a packet and broadcasts on all interfaces.

Generally you don't waste money on a hub, switches are cheap and smarter. Hubs can be used to tap Internet traffic for analysis, the network admin connects a hub and everything sent and received on the network comes to you. Switches are cheaper than routers and can still talk to the outside world. Switches are simpler than routers and can be faster.

Routers are good for connecting networks together, with NAT and DHCP services.

## Virtual Local Area Networks (VLANs)

Modern institutional LANs are often configured hierarchically with each department having its own switched LAN connected to the switched LANs of other groups via a switched hierarchy.

This has three drawbacks:

1. Lack of traffic isolation: The hierarchy localizes group traffic to within a single switch, but broadcacst traffic like frames carrying ARP and DHCP messages or frames whose destination have not yet been learned by a self-learning switch must still traverse the entire institutional network.

Limiting the scope of such broadcast traffic would improve LAN performance and it may be necessary to limit LAN broadcast traffic for security and privacy reasons.

2. Inefficient Uses of Switches: If instead of three groups, the institution had 10 groups, then 10 first-level switches would be required. If each group were small, say less than 10 people, then a single 96-port switch could be large enough to accommodate everyone. But this single switch would not provide traffic isolation.

3. Managing Users: If an employee moves between groups, the physical cabling must be changed to connect the employee to a different switch. Employees belonging to multiple groups make this even harder.

VLANs creates virtual local area networks to be defined over a **single physical local area network infrastructure**.

Hosts with a VLAN communicate with each other as if they and no other hosts were connected to the switch.

**Port-based VLAN**: The switch's interfaces/ports are divided into groups by the network manager. Each group constitutes a VLAN with the ports in each VLAN forming a broadcast domain.

ex. Have a single switch with 16 ports, ports 2-8 in the EE VLAN and ports 9-15 in the CS VLAN. Ports 1 and 16 are unassigned.

Solves all of the problems above.

If an EE user on interface 8 switches to the CS department, just reconfigure the VLAN software so that interface 8 is now associated with the CS VLAN.

The switch hardware only delivers frames between ports belonging to the same VLAN, so now a broadcast gets limited to the separate VLANs.

But now that they are completely isolated, how does traffic from EE get sent to CS?

**Forwarding between VLANs**:

Solution 1: Connect a VLAN switch port like port 1 to an external router and configure that port to belong to both the EE and CS VLANs.

Now an IP datagram going from EE to the CS department would first cross the EE VLAN to reach the router and then be forwarded by the router back over the CS VLAN to the CS host.

Switch vendors make this easy by making a single device that contains both a VLAN switch and a router so a separate external router is not needed.

## VLANs Spanning Multiple Switches

Now suppose that rather than having a separate EE department, some EE and CS faculty members are housed in a different building and want to be part of their department's VLAN.

Need to connect them, define a port belonging to the CS VLAN and EE VLAN respectively on each switch and connect these ports to each other. But this doesn't scale, N VLANS would require N ports on each switch simply to connection them together.

Use a **Trunk Port**, carries frames between VLANs defined over multiple physical switches.

This special port on each switch, port 16 on the left and 1 on the right, belongs to all VLANs and frames sent to any VLAN are forwarded over the trunk link to the other switch.

But how does a switch know that a frame arriving on a trunk port belongs to a particular VLAN? The IEEE defined an extended Ethernet frame format 802.1 Q for frames crossing a VLAN trunk.

Consists of a four-byte VLAN tag added in the header that carries identify of the VLAN to which the frame belongs located after the source address in the Ethernet frame header. Includes a 2-byte TAG Protocol Identifier, 2-byte Tag Control Information with a 12-bit VLAN ID field and a 3-bit priority field.

The VLAN tag is added into a frame by the switch at the sending side of a VLAN trunk, parsed and removed by the switch at the receiving side.
