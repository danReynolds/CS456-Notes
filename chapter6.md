## Wireless and Mobile Networks

Challenges:

1. Wireless: Communication over wireless links

2. Mobility: Handling the mobile user who changes point of attachment to the network.

Elements of a wireless network:

1. Wireless hosts: hosts are the end system devices like laptops and smartphones.

2. Base stations: connected to a wired network. Relay responsible for sending packets between wired networks and wireless hosts. Cell towers, 802.11 access points. Host must be within the wireless communication range of the base station.

3. Wireless links: typically used to connect mobiles to base station, also used at backbone link. Connect wireless hosts located at the edge of the network into the larger network infrastructure.

Hosts associated with a base station operate in **infrastructure mode**, since all traditional network services are provided by the network to which a host is connected via a base station.

In **ad hoc networks**, wireless hosts have no such infrastructure like base stations with which to connect. The hosts themselves must therefore provide services for routing, address assignment and DNS-like name translation. Can only transmit to nodes in a very limited range.

When a mobile host moves beyond the range of one base station and into the range of another, it will change its point of attachment into the larger network, a process called a **handoff**.

Needs to handle addresses changing and maintaining its connections.

We can classify wireless networks by two criteria:

1. **Single Hop/Multi Hop**: Whether a packet in the network crosses exactly one wireless hop or multiple wireless hops

2. **Infrastructure/ No Infrastructure**: Whether there is infrastructure such as abase station in the network.

* **Single Hop, No Infrastructure**: No base station, no connection to larger Internet, Bluetooth and ah-hoc networks.

* **Single Hop, Infrastructure**: Host connects to base station which connects to larger Internet. All communication is between the wireless host and the base station over a single wireless hop. Cellular, 802.11 WiFi.

* **Multiple Hops, No Infrastructure**: No base station, no connection to larger Internet, nodes may have to relay messages among several other nodes in order to reach a destination. May have to relay to reach other wireless node. MANET (Mobile ad hoc networks), VANET (Vehicular ad hoc networks). Ongoing research.

* **Multiple hops, Infrastructure**: Host may have to relay through several wireless nodes to connect to larger Internet, so a wireless node has to relay their communication through other wireless nodes in order to communicate with the base station. Mesh net.

## Wireless Networks vs Wired Networks

When we replace a wired home network, we'd need to replace the wired Ethernet with a wireless 802.11 network, and a wireless network interface would replace the host's wired Ethernet NIC, and an access point would replace the Ethernet switch.

But nothing above the network layer changes. Changes come to the network and link layer:

* **Decreased signal strength**: Electromagnetic radiation attenuates as it passes through matter, (a radio signal passing through a wall). The signal will disperse decreasing signal strength as the distance between the sender and receiver increases.

* **Interference from other sources**: Radio sources transmitting in the same frequency band will interfere with each other. 2.4 GHz wireless phones and 802.11b wireless LANs transmit in the same frequency band, so they interfere with each other and neither performs well. Nearby motors, microwaves, all interfere.

* **Multipath Propagation**: Occurs when portions of the electromagnetic wave reflect off objects and the ground, taking different paths of different lengths between a sender and a receiver. This results in the blurring of the received signal at the receiver.

All of these factors add difficulty to wireless networks in areas like attenuation, interference and multipath propagation that wired networks take for granted.

## Signal to Noise Ratio (SNR)

Relative measure of the strength of the signal to noise. Larger SNR, easier it is to determine the signal over the noise.

**Bit Error Rate (BER)** = The probability that a transmitted bit is received in error at the receiver.

The higher the SNR, the lower the BER.

A sender can increase the SNR and lower the BER by increasing its transmission power. But more power means more likelihood of interfering with other nearby signals, since yours is now stronger.

Increase Power->Increase SNR->Lower BER

Choose the physical layer that meets BER requirement, giving highest throughput.

## Undetectable collisions

1. **Hidden Terminal Problem**

Station A is transmitting to Station B, suppose also that Station C is transmitting to Station B. Physical obstructions in the environment like a mountain or a building can prevent A and C from hearing eachother's transmission, even though they are both still sending to B.

Since C and A cannot hear eachother, they don't know the other exists and don't know they're causing interference at B.

2. **Fading**

Signal fades as it propagates through the wireless network, so if A and C are placed such that their signals are not strong enough to detect eachother, but they are strong enough to intefere with B. Like if the order in a big geographical areas was A, B, C, then A's signal may be faded by the time it reaches C on the other side of B, but not at B, and vice-versa.

## Code Division Multiple Access (CDMA)

Type of channel partitioning protocol, popular in wireless technology.

Each bit sent is encoded by multiplying the bit by a signal/code that changes at a much faster rate, the **chipping rate** than the original sequence of data bits and this chipping sequence is used to encode the data.

Encoded signal = original data x chipping sequence

ex. Suppose that the rate at which original data bits reach the CDMA encoder defines the unit of time. Each original data bit to be transferred requires a 1-bit time slot.

$d_{i}$ is the value of the data bit for the ith bit slot. So we have a slot of time allocated for sending one data bit. Each bit slot is further divided into M mini slots.

The CDMA code used by the sender consists of a sequence of M values each taking a +1/-1.
In this example, M=8. For the ith data bit $d_{i}$, the output of the CDMA encoder, $Z_{im}$ for the mth M value it is the value of $d_{i}$ multiplied by the mth bit in the assigned CDMA code.

Therefore $Z_{im} = d_{i} * c_{m}$ which makes sense, the resulting encoding Z_{i} is the combination of taking each of the M's and multiplying it by the d_{i} bit.

Then for the receiver, it would be easy to compute the original bit:

$d_{i} = \frac{1}{M} * \sum{m=1}{M} Z_{im} * c_{m}$

we take the result $Z_{im}$, for each $m_{i}$, which could be -1 or 1 and multiply it by the code $c_{i}$ to get the value of $d_{i}$, the data-bit, since each $m_{i}$ was created from taking its code an multiplying it by $d_{i}$. Then now we have M values of $d_{i}$, and we can get $d_{i}$ by dividing it by M as shown.

But CDMA must work in the presence of interfering senders that are encoding and transmitting their data using a different code.

How can a CDMA receiver recover a sender's original data bits when those bits are being tangled with bits being transmitted by other senders?

CDMA assumes that interfering transmitted bit signals are additive. For example, if three senders send a 1 value, and a fourth sender sends a -1 value during the same slot, then the received signal at all receivers during the mini-slot is a (1 + 1 + 1 - 1) = 2. The value received at a receiver during the mth mini-slot of the ith data bit is now the sum of the transmitted bits ith Z bits for each of the N senders during that mini slot.

**If the sender's codes are chosen carefully, each receiver can recover a specific sender's data out of the aggregate signal simply by using the sender's code exactly in the same way as before:**

$d_{i} = \frac{1}{M} * \sum{m=1}{M} Z_{im} * c_{m}$

Cool!

ex.

d0 = -1
d1 = 1
M = (1,1,1,-1,1,-1,-1,-1)

Now multiply each M bit by its d bit to get the encoding:

Z0 = (-1,-1,-1,1,-1,1,1,1)
Z1 = (1,1,1,-1,1,-1,-1,-1)

Now assume that we have a second sender with:

d0' = 1
d1' = 1
M' = (1,-1,1,1,1,-1,1,1)

Z0' = (1,-1,1,1,1,-1,1,1)
Z1' = (1,-1,1,1,1,-1,1,1)

and the result is: Z0'' = Z0 + Z0' = (0,-2,0,2,0,0,2,2)
                   Z1'' = Z1 + Z1' = (2,0,2,0,2,-2,0,0)

Then at the receiver

it decodes each using the code for it and the result it received:

Use M for d0, so (0*1,-2*1,0*1,2*-1,0*1,0*-1,2*-1,2*-1) / 8 = -8 / 8 = -1 = d0

and this works for d1 = 1, d0' = 1, d1' = 1 as well.

Cool, we just got back the original values!

## WiFi: 802.11 Wireless LANs

Types:

1. 802.11b:

* 2.4-5Ghz unlicensed spectrum
* Up to 11Mbps
* Direct sequence spread spectrum (DSSS) in physical layer
* All hosts use same chipping code

2. 802.11a:

* 5-6GHz range
* Up to 54Mbps

3. 802.11g:

* 2.4-5GHz range
* up to 54 Mbps

4. 802.11n:

* multiple antennae
* 2.4-5GHz range
* Up to 200Mbps

5. 802.11ac:

* multiple antennae
* 5GHz
* Up to 2 Gbps

## 802.11 Architecture

Building block of the 802.11 architecture is the **basic service set (BSS)**. A BSS contains one or more wireless stations and a central **base station** known as an **access point (AP)**.

In a typical home, there is one AP and one router that connects the BSS to the Internet.

Each 802.11 wireless host has a MAC address for its wireless interface. LANs that deploy APs/Base Stations are infrastructure wireless LANs, with the infrastructure being the APs along with the wired Ethernet infrastructure connected to the APs and the router.

## Channels and Association

802.11b: 2.4GHz-2.486GHz spectrum divided into 11 channels at different frequencies.

Admin picks a **Service Set Identified (SSID)** to the access point, the name you see for a WiFi, etc.

The admin also must pick a channel number. Any two channels are non-overlapping if and only if they are separated by four or more channels. 1,6,11 is the only set of 3 non-overlapping channels. You could therefore get max transmission rate by having three APs operating on 1,6,11 respectively and connected via a switch.

A wireless host must associate with exactly one of the APs from a specific subnet, like Eduroam vs Blackberry Mobile HotSpot 8553.

Only the associated AP will send data from the Internet to your wireless host, and vice-versa.

To find an AP, the 802.11 standard requires that an AP periodically send **beacon frames**, each of which includes the AP's SSID and MAC address.

Your wireless host, knowing that APs are sending out beacon frames, scans the 11 channels, seeking beacon frames from any APs that may be out there. You then select one to connect to.

Choosing the best host to connect may depend on factors like signal strength, and traffic on that AP. A wireless host must judge these criterion from an AP's beacon frames and there are multiple ways to obtain these frames:

* **Passive Scanning**:

1. Scanning channels and listening for beacon frames.

2. Beacon frames sent from APs.

3. Association request frame sent from wireless host to AP.

4. Association response frame sent from AP to wireless host

* **Active Scanning**:

1. Wireless host broadcasts a **probe frame** that will be received by all APs within the wireless host's range.

2. APs respond to the probe request frame with a probe response frame.

3. Association request frame sent from wireless host to desired AP from list.

4. Association response frame sent from AP to wireless host

Once associated with the AP, the host will want to join the subnet to which the AP belongs and initiates DHCP via the AP.

## Multiple Access Protocol

Multiple wireless hosts can transmit at the same time to the AP. Inspired by the huge success of Ethernet and its random access protocol, the designers of 802.11 chose a random access protocol for 802.11 wireless LANs.

This is **Carrier Sense Multiple Access with Collision Avoidance (CSMA/CA)**.

Similar to Ethernet's CSMA/CD, but whereas Ethernet has collision detection, wireless LANs are different and use Collision Avoidance.

Two reasons why 802.11 chose not to do CD:

1. The ability to detect collisions requires the ability to send (the station's own signal) and receive (to determine whether another station is transmitting) at the same time. Because the strength of the received signal is typically very small compared to the strength of the transmitted signal at the 802.11 adapter, it is costly to build hardware that can detect a collision.

2. Even if the adapter could transmit and listen at the same time, and abort transmission when it senses it busy, it could still miss collisions because of the hidden terminal problem and fading. It's too hard to detect collisions.

Therefore since 802.11 LANs do no use collision detection, once a station begins to transmit a frame, it transmits the **entire frame**, there is no turning back.

## Frame Acknowledgement

Transmitting entire frames when collisions are happening can significantly reduce a multiple access protocol's performance.

In order to reduce the likelihood of collisions, 802.11 employs several collision avoidance techniques.

Since a frame sent may not reach the destination intact, 802.11 uses link-layer acknowledgements. When the destination station receives a frame that passes the CRC, it waits a short period of time known as the **Short Inter-frame Spacing (SIFS)** and then sends back an acknowledgement frame.

If the transmitting station does not receive an acknowledgement within a given amount of time, it assumes that an error has occurred and retransmits the frame using the CSMA/CA protocol to access the channel.

If an acknowledgement is not received after some fixed number of retransmissions, the transmitting station gives up and discards the frame.

## CSMA/CA Protocol

Now that we get link-layer acknowledgements, here is the CSMA/CA protocol:

Suppose a station has a frame to transmit.

1. If initially the station senses the channel idle, it transmits its frame after a short period of time known as the **Distributed Inter-Frame Space (DIFS)**.

2. Otherwise, the the station chooses a random backoff value using our friend **binary exponential backoff** and counts down this value when the channel is sensed idle. While the channel is sensed busy, the counter value pauses.

3. When the counter reaches zero, which can only occur while the channel is idle, the station transmits the entire frame and then waits for an acknowledgement.

4. If an acknowledgement is received, the transmitting station knows that its frame has been correctly received at the destination. If the station has another frame to send, it begins the CSMA/CA protocol at step 1.

5. If the acknowledgement is not received, the transmitting station reenters the backoff phase in step 2 with the random value chosen from a larger interval. If this happens enough, it discards that frame.

Why does CSMA/CA have a countdown during the idle and not just transmit when it senses it idle like CSMA/CD?

Assume that two stations each have a data frame to transmit, but neither transmits immediately because each senses a third station is already transmitting. With CSMA/CD, the stations would each transmit as soon as they detect the third one has finished.

This would cause a collision, which is fine for CSMA/CD, since both would abort their transmissions. In 802.11, a frame suffering a collision will be transmitted in its entirety.

In 802.11, if they both sense it busy, they both immediately backoff, hopefully choosing different values and avoiding the collision. The one with the shorter backoff will begin, and the other will hopefully sense that it is transmitting using its carrier sense and pause its countdown, so that it won't interrupt it.

This is double security, since for a collision to occur between the 2 stations:

1. They would have to have overlapping backoff times

2. They would have to be unable to sense eachother, such as if there was a hidden terminal problem, or fading.

## Dealing with Hidden Terminals: RTS and CTS

802.11 MAC protocol includes an optional reservation scheme that helps avoid collisions even in the presence of hidden terminals.

Both of the wireless hosts/stations are within range of the AP (whose coverage is shown as a shaded circle) and both have associated with the AP.

Due to fading, each of the stations are hidden from each other.

Suppose station H1 is transmitting a frame and halfway through, station H2 wants to send a frame to the AP. H2, not hearing the transmission from H1, will first wait a DIFS interval and then transmit the frame, resulting in a collision.

In order to avoid this problem, 802.11 protocol allows a station to use a short **Request to Send (RTS)** control frame and a short **Clear to Send (CTS)** control frame to reserve accesss to the channel.

When a sender wants to send a data frame, it first sends an RTS frame to the AP indicating the total time required to transmit data the frame and the acknowledgement frame. When the AP receives the RTS frame, it responds by broadcasting a CTS frame. This CTS frame serves two purposes:

It gives the sender explicit permission to send and also instructs the other stations to not send for the reserved duration.

In our example, before transmitting its data frame, H1 first broadcasts an RTS frame which is heard by all stations in its circle including the AP. It still waits a DIFS before sending the RTS, and then waits a SIFS after the CTS before sending the data.

The AP then responds with a CTS frame, which is heard by all stations within range including H1 and H2.

Station H2, having heard the CTS, refrains from transmitting for the time specified in the CTS frame.

RTS and CTS improves performance in two ways:

1. The hidden station problem is mitigated, since a long data frame is transmitted only after the channel is reserved.

2. Because the RTS and CTS frames are short, a collision involving the RTS or CTS will only last for a short duration. Once they are correctly transmitted, the data and acknowledge will be transmitted without collision.

## 802.11 Frame Addressing

Shares many similarities with an Ethernet frame, but also contains a number of fields that are specific to its use for wireless links.

Adds additional address fields, has 4 address fields for MAC addresses. It turns out that three address fields are needed to move the network-layer datagram from a wireless host through an AP to a router interface.

The fourth address is only used when APs forward frames to eachother in AD-HOC no infrastructure mode.

Looking at the first 3 in infrastructure mode:

1. **Destination Address**: Address 1 is the MAC address of the wireless host that is to receive the frame. Thus if a mobile wireless host transmits the frame, address 1 contains the MAC address of the destination AP since the wireless host must send it there first.

2. **Source Address**: Address 2 is the MAC address of the host that transmits the frame. Thus f a wireless host transmits the frame, its MAC address is inserted in field 2. Similarly if an AP transmits the frame, then the AP's MAC address is inserted in address 2.

3. Both address 1 and 2 function like normal MAC addresses in Ethernet frames. For address 3, recall that the BSS consisting of the AP and the wireless hosts and router is part of a subnet and that this subnet is connected to other subnets via a router. Address 3 is the MAC address of the router to which the AP is attached.

We need three addresses because the **router does not know about the AP**, it thinks its directly connected to the wireless host.

ex. Two APs, connected via router R1, AP1 has H1 and AP2 has H2

Moving a datagram from the router interface R1 to the wireless host H1. **The router is not aware that there is an AP between it and H1**.

The router knows the IP address of H1 and uses ARP to determine the MAC address of H1. After obtaining H1's MAC address, R1 encapsulates the datagram in an Ethernet frame. The source address contains R1's MAC address and the destination address is H1's MAC address. It has no third address as this is still 802.3 Ethernet, the router doesn't know about 802.11. It then sends the frame on the interface that its forwarding table has for H1's IP, which leads to the AP.

When the Ethernet frame arrives at the AP, the AP converts the 802.3 Ethernet frame to an 802.11 frame before transmitting it on to the wireless channel to H1. The AP fills in address 1 with the MAC address of H1, and address 2 with its own MAC address. It then uses the MAC address of R1, the router, in the third address and sends it on the interface to H1.

Now when the wireless host H1 responds, it creates an 802.11 frame filling the fields for address 1 with the AP MAC, address 2 with its own MAC and address 3 with the MAC of R1.

Then when the AP receives the 802.11 frame, it converts the wireless LAN frame to an 802.3 with the H1's MAC as the source and R1 as the destination.

Address 3 has allowed the router to transparently talk to H1 without knowing about the AP or 802.11.

## Other Fields

The modified 802.11 frame has some other notable fields:

* duration field: reserved for use with RTS/CTS
* frame sequence number: for RDT at the network layer
* frame type: RTS, CTS, ACK, data types of frames

## Mobility within the Same Subnet

How do wireless devices move seamlessly from one AP in Eduroam to another?

As wireless host H1 moves away from AP1, H1 detects a weakening signal from AP1 and starts to scan for a stronger signal. H1 scan he Eduroam network, looks for Eduroam's APs and sends association request to the strongest one based on the beacon frames it received.

Since both APs are on the same subnet, H1 dissociates with AP1 and associates with AP2 while keeping its IP and maintaining ongoing TCP connections.

Switches then need to self-learn that you have switched interfaces.

One solution to update this is to have AP2 send a braodcast frame with H1's source address to the switch just after the new association. When the switch receives the frame, it updates its forwarding table and now anything destined for H1 will be forwarded onto the interface of AP2, as we want.

The 802.11f standards group is developing an inter-AP protocol to handle these issues.

## 802.11 Advanced Capabilities

### 802.11 Rate Adaptation

Different modulation techniques are appropriate for different SNR scenarios. Some 802.11 implementations have a rate adaptation capability that adaptively selects the underlying physical-layer modulation technique to use based on the current or recent channel characteristics.

So if you walk away from the AP and the SNR goes down, it needs to adjust its modulation technique to compensate or the BER will get too high.

When BER becomes too high, switch to a lower transmission rate with a lower BER. Since you transmit more slowly, there is is able to detect the information more clearly and get lower BER.

### Power Management

Power is important on mobile devices, as they have limited batteries. In 802.11, a node is able to alternate between sleep and wake states. A node indicates to the AP that it will be going to sleep by setting the power-management bit in the header of an 802.11 frame to 1.

A timer in the node is then set to wake up the node just before the AP is scheduled to send its beacon frame.

Since the AP knows that the node is going to sleep, it will buffer any frames destined for the sleeping host for later transmission.

The beacon frame received by the node includes a list of nodes that have frames buffered for them. If the node is in that list, it stays awake, otherwise it can go back to sleep.

### 802.15: Personal Area Networks

Less than 10m diameter. Replacement for cables like mice, keyboard, headphones.

Ad-hoc with no infrastructure.

Uses a master/slave system, slave requests permission to send to master. Master grants requests.

802.15 evolved from Bluetooth spec, operates on 2.4-2.5GHz at up to 721kbps.

## Cellular Network Architecture

**Global System for Mobile Commmuncations (GSM)** standards are used here.

* 1G: Analog FDMA sysems designed exclusively for voice-only communication.

* 2G: Original 2G systems were designed for voice only, but later extended 2.5G to support data as well as voice.

* 3G: Systems currently deployed and which support voice and data.

The term **cellular** refers to the fact that the region covered by a cellular network is partitioned into a number of geographic coverage areas called cells.

Each cell contains a **base transceiver station (BTS)** that transmits signals to and receives signals from mobile stations in its cell and it is analogous to a 802.11 AP. This is the cell tower.

## 2G Network

The GSM standard for 2G cellular systems use combined FDM/TDM (radio) for the air interface so we have channels partitioned into a number of frequency sub-bands, within each sub-band time is partitioned into slots.

This allows the channel to accept F*T calls.

A GSM network's **base station controller (BSC)** will typically service several tens of base transceiver stations.

The role of the BSC is to allocate BTS radio channels to mobile subscribers and perform **Paging** = finding the cell in which a mobile user is resident and perform handoff of mobile users.

The base station controller and its controlled base transceiver stations collectively constitute a **Base station system (BSS)**.

The **Mobile switching center (MSC)** provides the central role in user authorization and accounting such as determining if a mobile device is allowed to connect to the cellular network, call establishment and teardown and handoff.

A single MSC wil contain up to 5 BSCs, resulting in about 200K subscribers per MSC. A cellular provider's network will have a number of MSCs with special MSCs called gateway MSCs that connect the provider's cell network to the larger public telephone network.

## 3G Core Network

3G core cellular network connects radio access networks to the public Internet. The core network inter-operates with components of the existing 2G cellular voice network, in particular the MSC.

It leaves the existing core GSM cellular voice network untouched, adding additional cellular data functionality in parallel to the existing cellular voice network.

Adds two types of nodes in the 3G core network:

**Serving GPRS Support Nodes (SGSNs)**: SGSN is responsible for delivering datagrams to/from mobile nodes in the radio access network to which the SGSN is attached.

It interacts with the cellular voice network's MSC for that area, providing user authorization and hand-off, maintaining location (which cell) information about active mobile nodes and performing datagram forwarding between mobile nodes in the radio access network and a GGSN.

**Gateway GPRS Support Nodes (GGSNs)**: Acts as a gateway, connecting multiple SGSNs into the larger Internet. A GGSN is the last piece of 3G infrastructure that a datagram originating at a mobile node encounters before entering the larger Internet. To the outside world, the GGSN looks like any other gateway router.

The mobility of the 3G nodes within the GGSN network is hidden from the outside by the GGSN.

## 3G Radio Access Network: The Wireless Edge

The 3G **radio access network** is the wireless first-hop network that we see as a 3G user. The **Radio Network Controller (RNC)** controls several cell base transceiver stations.

Each cell's wireless link operates between the mobile nodes and a base transceiver station, like in 2G. RNC connects to both circuit-switched cellular voice network via an MSC and to the packet-switched Internet via an SGSN.

Therefore while 3G cellular voice and cellular data services use different core networks, they share a common first/last-hop radio access network.

So it goes:

RNC controls several base tranceiver stations and relays their information to 2 sources:

1. It connects to MSC, which manages calls and talks to the Gateway MSC that communicates with the public telephone network.

2. It connects to SGSN, which receives and delivers datagrams from the mobile nodes and is connected to a GGSN, which sends datagrams originating from mobile nodes into the Internet.

While 2G networks use FDMA/TDMA multiple access protocols, 3G uses a CDMA technique within TDMA slots.

## 4G: LTE

Two important innovations over 3G systems:

1. **Evolved Packet Core (EPC)**: Simplified all-IP core network that unifies separate circuit-switched cellular vice network and the packet-switched cellular data network.

It is all-IP because both voice and data will be carried in IP datagrams. Key task of EPC is to manage network resources to provide high quality service for VoIP.

The EPC allows multiple types of radio access networks including the legacy 2G and 3G systems to attach to the core network.

2. **LTE Radio Access Network**: Uses combo of FDM and TDM on the downstream channel.

Uses sophisticated multiple input/output antennas with higher speeds.

## Mobility

Mobility Spectrum:

* No Mobility: User moves only within same wireless access network. If you just move a laptop over a few meters, it is not mobile from a network layer perspective, same IP, and not mobile from a link-layer perspective, same interfaces.

* Some Mobility: User moves between access networks, shutting down while moving between networks.

* High Mobility: User moves between access networks, while maintaining ongoing connections.

### Mobility Vocabulary

**Home Network**: The first network used to make a call or establish TCP.

**Home Agent**: Entity that will perform mobility functions on behalf of the mobile when it is not there.

**Permanent Address**: The address in the home network, it must stay the same or all existing connections would need to be re-established.

**Visited Network**: network in which mobile device currently resides.

**Foreign Agent**: Entity in visited network that will perform mobility functions on behalf of the mobile.

**Care-of-Address**: Address in the visited network.

## Mobility: Approaches

### Router's Responsibility

If a mobile entity is able to maintain its IP address as it moves, then it is invisible from the application standpoint. Mobile IP provides this transparency that is critical to maintaining persistent connections like TCP.

To preserve this IP, all traffic addressed to a mobile node's home network must now be forwarded to its foreign network.

One solution is to have the foreign network advertise to all other networks that the mobile node is resident in its network using intradomain and interdomain routing.

its neighbour networks would propagate this information, and the home network would know how to reach the node in its new foreign network.

Drawback is scalability. If mobility management is left to the network routers, they would need to maintain forwarding table entries for potentially millions of mobile devices and update these entries as they move.

### End System's Responsibility

The viable solution, home network can track the foreign network in which the mobile node resides.

Do this with **Mobility registration**:

1. Mobile contacts foreign agent on entering the visited network. The mobile node registers with the foreign agent when attaching to the foreign network.

2. Foreign agent contacts home agent and says that this mobile resident is in its network, providing a COA address to register with the home agent.

Two types of foreign routing:

* **Indirect Routing**:

1. Correspondent addresses packets using the permanent address of the mobile

2. Home agent intercepts packets, forwards them to the foreign agent. The home agent will need to address the datagram using the mobile node's COA so that it will route to the foreign network, but the node should not know that the datagram was forwarded via the home agent, that should be transparent. So the home agent encapsulates the correspondent's original datagram within a new datagram destined for the COA.

3. Foreign agent receives packets destined for COA and will decapsulate the datagram, forwarding the original datagram to the mobile node with its permanent address as the destination. The mobile node is still accepting its permanent address in this foreign network.

4. Mobile replies directly to the correspondent using its permanent address as the source and the correspondent's address as the destination, there is no need to route the datagram back through the correspondent.

ex. Correspondent sends a packet to the home agent, which sees it destined for address A. It intercepts this, knows A is away, and uses A_{COA}, the care of address for A in the foreign network, sending the datagram to that address with the source address of the correspondent.

The foreign agent receives this packet addressed to A_{COA} and sends it to A, who replies directly to the correspondent.

Suffers from **Triangular routing** problem. Datagrams addressed to the mobile node must be forwarded through the home agent and then to the foreign network, even when there is a more efficient route.

If the foreign network the node is visiting is the same one as the correspondent, now the correspondent sends data to the home network, only to have it come back to its own network. This is inefficient.

* **Direct Routing**:

Overcomes inefficiency of triangular routing at the cost of added complexity.

In the direct routing approach, a correspondent agent in the correspondent's network first learns the COA of the mobile node. This can be done by having the correspondent agent query the home agent.

The correspondent agent, after learning the COA from the home agent, can then tunnel datagrams directly to the mobile node's COA similar to how the home agent tunneled in indirect routing.

The foreign agent receives the tunneled datagram and decapsulates it, providing the mobile node with a datagram with its permanent address as the destination and the correspondent's address as the source. It can then reply directly to the correspondent.

This fixes the triangular routing problem since the correspondent agent only has to speak to the home network once to get the COA address. From then on the correspondent can send messages directly to the foreign agent, so if it is in the same network as the mobile node, it will just send the packet locally to its foreign agent, which is much faster than in indirect routing.

Direct routing introduces two important challenges:

1. Need a way for the correspondent agent to query the home agent to obtain the mobile node's COA.

2. When the mobile node moves from one foreign network to another, what happens? In indirect routing, the new foreign agent tells the home network to update the COA and we're done, but in direct routing, the correspondent already has the old COA and doesn't talk to the home agent, so updating the COA with the home agent is not enough. The home agent or the new foreign agent needs to tell the correspondent and any other correspondents that its COA has changed.

The challenge of changing foreign agents can be solved using an **anchor foreign agent**. If data is currently being forwarded to the mobile node in the foreign network where the mobile node was located when the TCP connection first started, the foreign agent in that foreign network was the first foreign agent for the connection, and is the anchor.

When the mobile node moves to a new foreign network, the mobile node registers with the new foreign agent and the new foreign agent provides the anchor foreign agent with the mobile node's new COA. Now the correspondents can continue forwarding to where they think the mobile node is, the foreign network of the anchor, and the anchor can encapsulate the datagram using the new COA it has received and send it to the new foreign agent.

So the correspondent encapsulates its datagram destined for the permanent address in a datagram destined for the anchor foreign network's COA, which receives this datagram, decapsulates it and re-encapsulates it destined for the new COA.

## Mobile IP

The Internet architecture for supporting mobility, collectively known as mobile IP are defined in RFC 5944 for IPv4.

It includes the concept of home agents, foreign agents, foreign-agent registration, care-of-addresses, encapsulation.

It is broken into three components:

1. **Indirect Routing of Datagrams**: The standard defines the manner in which datagrams are forwarded to mobile nodes by a home agent, including rules for forwarding datagrams, rules for handling errors and several forms of encapsulation.

2. **Agent Discovery**: Mobile IP defines the protocols used by a home or foreign agent to advertise its services to mobile nodes, and protocols for mobile nodes to solicit the services of a foreign or home agent.

A mobile IP node arriving at a new network, whether attaching to a foreign network or returning to its home network, must learn the identity of the corresponding foreign or home agent.

With **agent advertisement**, a foreign or home agent advertises its services using an extension to the existing router discovery protocol. The agent periodically broadcasts an ICMP message with type field 9 on all links to which it is connected, informing them of the IP address of the router, which is the agent, and allowing a mobile node to learn the agent's IP.

The router discovery message also includes mobility agent info needed by the mobile node:

* Home agent bit (H): Indicates that the agent is a home agent for the network in which it resides

* Foreign agent bit (F): Indicates that the agent is a foreign agent in the network in which it resides.

* Registration required bit (R): Indicates that a mobile user in this network must register with a foreign agent. A mobile user cannot obtain a care of address in the foreign network using DHCP until it regsiters with the foreign agent.

* M,G Encapsulation bits: Indicates whether a form of encapsulation other than IP-over-IP will be used.

* Care of Address Fields: A list of one or more care of addresses provided by the foreign agent. The COA will be associated with the foreign agent, who will receive datagrams sent to the COA and forward them to the appropriate mobile node. The mobile node will select on of these addresses as its COA when registering with the home agent.

3. **Registration with the Home Agent**: Mobile IP defines the protocols used by the mobile node and/or the foreign agent to register and deregister COAs with a mobile node's home agent.

Once a mobile IP has received a COA, that address must be registered with the home agent.
This can be done either via the foreign agent, who then registers the COA with the home agent, or directly by the node mobile node itself.

Registration by the foreign agent:

1. Following receipt of a foreign agent advertisement, the mobile node sends a mobile IP registration message to the foreign agent.

The registration message is carried over UDP to port 434. The message carries a COA advertised by the foreign agent, the address of the home agent (HA), the permanent address of the mobile node (MA), registration identification and lifetime.

2. The foreign agent receives the registration message and records the mobile node's permanent IP address. The foreign agent now knows that it should be looking for datagrams containing an encapsulated datagram whose destination address matches the permanent address of the mobile node.

The foreign agent then sends a mobile IP registration method over UDP to port 434 of the home agent. The message contains the COA, HA, MA, encapsulation format requested, lifetime and registration identification info.

3. The home agent receives the registration request and checks for authenticity and correctness. It would be bad if someone spoofed these. The home agent binds the mobile node's permanent IP address with the COA, the home agent sends a mobile IP registration reply containing the HA, MA, actual lifetime, and the registration identification of the request that is being satisfied.

4. The foreign agent receives the registration reply and forwards it to the mobile node. The registration identification was required to map registration requests and replies.

The registration is then complete and the mobile node can receive datagrams sent to its permanent address while in a foreign network.
