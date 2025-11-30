
Though wireless networks can come in a variety of forms, we can classify networks by their basic topology. 

*Wireless hosts*, or end-user devices, use *wireless links* to transmit and receive data to *base stations*, which connect to the larger *network infastructure* that the hosts want to communicate with. Wireless technologies differ on the strength, distance, and encoding used for wireless links, and on the exact responsibilities of the base station. 

Each host is associated with a base station (running in *infrastructure mode*, meaning that it's within the communication of the base station and uses it to relay data between it and a larger network. The base station can itself be connected to infrastructure, which can provide traditional networking services like DNS and DHCP. However, wireless networks don't need to use base stations. In *ad-hoc* networks like Bluetooth, one host in the network can coordinate transmissions among the other hosts. 

# Wireless Physical Properties

Like some wired networks, wireless networks are broadcast networks. However, they have a couple of different, and important, properties. 

• **Decreasing signal strength**. Electromagnetic radiation attenuates as it passes through matter (e.g., a radio signal passing through a wall). Even in free space, the signal will disperse, resulting in decreased signal strength (sometimes referred to as path loss) as the distance between sender and receiver increases.
• **Interference from other sources**. Radio sources transmitting in the same frequency band will interfere with each other. For example, 2.4 GHz wireless phones and 802.11b wireless LANs transmit in the same frequency band. Thus, the 802.11b wireless LAN user talking on a 2.4 GHz wireless phone can expect that neither the network nor the phone will perform particularly well. In addition to interference from transmitting sources, electromagnetic noise within the environment (e.g., a nearby motor, a microwave) can result in interference. For
this reason, a number of more recent 802.11 standards operate in the 5GHz frequency band.
• **Multipath propagation**. Multipath propagation occurs when portions of the electromagnetic wave reflect off objects and the ground, taking paths of different lengths between a sender and receiver. This results in the blurring of the received signal at the receiver. Moving objects between the sender and receiver can cause multipath propagation to change over time.![[Pasted image 20251130113951.png]]

These properties indicate that we'll have to deal with more errors than we would in wired networks. There are two ways to solve this: either we increase the power of our transmissions and waste our batteries, or we develop stronger methods of error detection and correction. 

Wireless networks also suffer from the **hidden terminal problem**, where two nodes can hear one node they connect to but not each other due to a physical obstruction. This can be combined with **fading**, where two nodes can interfere with each other's signals but not hear each other, making each unaware of their interference with the other. This adds another layer of complexity onto wireless error detection. 
![[Pasted image 20251130114621.png]]

# WiFi - Wireless LANs

802.11, or WiFi, is the standard protocol for connecting devices together on a wireless LAN. Its architecture starts with the **BSS**, or Basic Service Set, referring to a group of devices connected to each other by at most one AP (the set spanned by multiple APs connected together is called an **ESS**, or extended service set). In an **infastructure wireless LAN**, devices are connected to a base station called an **access point (AP)**, which itself connects to a router which provides wider Internet access. ![[Pasted image 20251130115224.png]]
Note that this is not required; devices can communicate on WiFi without the need for an AP through **ad-hoc wireless LANs** like the one below. ![[Pasted image 20251130115257.png]]

## Channels and Association

When an administrator configures an AP, they have a couple of choices to make. First, they must assign an SSID, or "name" to the AP, along with a channel number. Since we want to communicate over non-overlapping channels, and the only choices we have in the 2.4Ghz spectrum are 1, 6, and 11, we usually assign APs one of those channels, provided no neighboring AP is on the same channel. Note that this allows multiple APs to communicate in the same area simultaneously, just on different channels. 

Devices need to **associate**, or connect, to an AP. But first, we need to find APs. We can do this in two ways: passive scanning, and active scanning. No matter how we get a list of available APs, we usually choose the one with the strongest signal strength. 
- **Passive scanning**: APs will periodically send out beacon frames, including their SSID and MAC address. Devices scan all 11 channels for those frames to learn about nearby APs available for connection. 
- **Active scanning**: A device can send out a probe request broadcast, inviting all nearby APs to send out a response. The device will then choose a device to associate from among the ones that send back a probe response.

Once we know what device we want to connect to, we'll send an association request to the chosen AP, which the AP will acknowledge with an association response. This handshake confirms to the AP that the device is connected to it, which is necessary as otherwise the AP has no way of knowing so. 

![[Pasted image 20251130120556.png]]

## WiFi Access Protocol

WiFi sends frames through a **CSMA/CA** scheme, similar to Ethernet's CSMA/CD but with collision avoidance rather than collision detection. Why not? 
- To detect all collisions, we need to be able to send and receive simultaneously to know if we need to jam and retransmit. The strength of the received signal is very small relative to the strength of our sent signal, which would require expensive hardware to detect. 
- We also need to be able to detect a collision that happens anywhere on the network, which we can't because of the hidden terminal problem and fading. 

So, we'll always transmit our frame in its entirety. But before we talk about collision avoidance, we need to talk about missing frames that could occur due to a variety of transmission problems that we discussed in the previous section. We can counter this through a **link-layer acknowledgement**, where we'll send an ACK frame a short while after receiving a valid data frame. This is how we identify collisions: if the transmitting host doesn't receive an ACK, it assumes an error has occurred and retransmits the frame according to the CSMA/CA protocol. If we do this too often and never get an ACK, we'll eventually discard the frame. Note that we also use sequence numbers in these frames, similarly to TCP, so that we can send out multiple frames back-to-back without waiting for an ACK, while still knowing which frames an ACK is confirming it received. 

The CSMA/CA protocol looks like this: 
1. If the channel is idle and we have something to send, send it after waiting a little bit. 
2. If the channel isn't idle, pick a random backoff value using binary exponential backoff (like the one used in CSMA/CD), and count down this value any time the channel is idle. When the channel is busy, the counter will be frozen. 
3. When the counter expires (and the channel is therefore idle), transmit the frame in its entirety. 
4. We wait for an acknowledgement. If we receive it and have something else to send, we go back to step 2, and if we don't receive it, we'll expand our random interval and go back to step 2. 

In CSMA/CD, we transmitted as soon as we detected a channel was idle. Why don't we do the same in CSMA/CA? Let's say two hosts have a frame to transmit, but detect another frame currently being transmitted. In CSMA/CD, both hosts would transmit their frames at the same time as soon as the frame currently being transmitted finished. This would cause a collision, which both hosts would detect, and stop transmitting their frames as a result. CSMA/CA wants to avoid these types of collisions, so uses a random backoff value for each in the hopes that the two hosts end up transmitting at different times; one earlier, and the other later, freezing its counter while the other is transmitting. 

But collisions can still happen even under this system: the two hosts could be hidden from each other via the hidden terminal problem. We can resolve this with **RTS/CTS frames**. Before a host sends its data frame, it will send a **Request to Send frame** to the AP, describing how long it will take for the host to transmit its data. The AP will then broadcast a **Clear-to-Send frame** to all hosts on the network, telling them not to send messages for the time it takes the other host to send its messages. 

This helps defeat the hidden terminal problem, since even if hosts can't see each other's frames, they'll see if the CTS frame the AP sent in response matches their RTS, and know if they're allowed to transmit or not.  This also allows multiple APs to exist in the same general channel space: If host C is sending to AP D, and host B is sending to AP A, if C hears an RTS from B but not a CTS from A, it knows its transmission won't interfere with B's, solving what is called the **exposed terminal problem**. 
![[Pasted image 20251130123810.png]]

## Wifi Frame Structure

There are 3 key addresses in WiFi frames, which is about 1 more than we'd expect. They function as follows: 

Address 1: MAC address of the receiver
Address 2: MAC address of the sender
Address 3: MAC address of the router interface to use. 

This allows us to easily internetwork wireless BSS with a wired LAN system like Ethernet. Say a router is connected via Ethernet to a variety of APs. 
- If it wants to send a message to Host H1, it will first perform an ARP lookup to get its MAC address, then encapsulate it in an Ethernet frame with source addr: R1 dest addr: H1. It will then send it along via Ethernet. This whole time, the router doesn't know an AP is connected to it. It doesn't need to; the AP will convert the Ethernet frame into a WiFi frame, with addr1: H1 addr2: AP, addr3: R1, so that the receiving host knows which router interface a message came from. 
- Conversely, when the host wants to send a message to the wider internet, it will send a frame with addr1: AP addr2: H1, addr3: R1. When the AP receives this frame, it will transform it into an Ethernet frame with source addr: AP, dest addr: R1. 
![[Pasted image 20251130143314.png]]

## Mobility within the same subnet

To support longer-range networks, like those of a big corporate office, we'll want to have multiple APs within a single IP subnet. But how can we maintain connections based on an IP (like TCP sessions) whenever a user moves between two different BSS's? 

First, we DON'T want to use a router to connect our two APs, since that would mean two BSS sets would have to be their own subnets (otherwise, the router wouldn't be able distinguish between the two and know to route traffic between them). A host moving between the two APs would have to wait until a new IP is assigned, terminating any existing IP-based connections. 

Instead, we'll use a switch, which can forward traffic independently of IP addresses, allowing BSS's living on the same subnet to occupy multiple interfaces in a switch.

As a host moves from one host to another, it will detect a weakening signal from the AP it's moving from, and beacon frames from the AP it's moving to. It will disassociate from the more distant AP and associate with the closer AP, all without changing the IP address associated with it. But how does the switch know to route incoming messages for that device to the other AP? We can take advantage of switches' self-learning by broadcasting an Ethernet frame with the host's source address just after the association change, which will prompt the switch to update its  table and start forwarding messages meant for the host to its new AP (belonging to a different interface on the switch). 
![[Pasted image 20251130144541.png]]

## WiFi Advanced Features

There are two notable advanced features of WiFi, rate adaption and power management. 
- **Rate adoption**: We'll choose the physical-layer modulation technique used in transmission based on recent channel characteristics. For example, if we send 2+ frames in a row and don't receive an ACK, we'll lower our transmission rate. If 10+ frames in a row are ACK'd or if a timer we set expires, we'll move up a transmission rate. This is similar to TCP's probing in congestion control, allowing us to get the highest reliable transmission rate. 
- **Power management**: Nodes will indicate to AP's that they're going to sleep by sending them a frame with an option set to 1. The node will wake up just before the AP sends a beacon frame. Knowing that the node is asleep, the AP will buffer any frames meant for that node, and include a list of all nodes with frames it has buffered in its beacon frames. On hearing this frame, the node will check if it's on the list. If it is, it will poll the AP for the frames. If it isn't, it can go back to sleep. This allows nodes to sleep up to 99% of the time, saving precious battery on small mobile devices. 

# Bluetooth

TODO, not covered in class

# Cellular Networks

Mostly TODO, what's covered in class was super outdated (2G voice??) All we really covered was that there are two ways to go from mobile to base-station radio: combining FDM with TDM (divide into frequency channels, and then divide those into time slots), or CDMA (old, code division, not explained at all)

