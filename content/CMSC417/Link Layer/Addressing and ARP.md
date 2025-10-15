At the link layer, we deal with moving information between individual nodes within a network. Like IP, we use an addressing scheme to uniquely identify hosts. However, that's where most of the similarities end. 
 > [!Why not just use IP addresses?]
 > If we did, then we'd have to pass link-layer packets "up" to the network layer, checking to see if the destination IP matches our IP. Not only does this violate encapsulation, but this means we'd have to repeat this process for every packet our interface picks up, even if they're not destined for us. Wasteful! Bad!
# MAC Addresses

MAC addresses are 48 bits, split into 6 octets. The first 3 octets are the OUI, referring to blocks of addresses bought out by manufacturers. Speaking of manufacturers, MACs are hardware-defined, unique to the hardware of a network interface present in a device. 

There's no IP-style network and host bits here, since we're not dealing with networks anymore: we want the same MAC address can be used on any network, so encoding a network prefix doesn't make sense here. 

# Address Resolution Protocol

If MACs are used to address link-layer packets, then we need a way of acquiring the MACs of devices in our network in order to send packets to them. Each host has its own ARP cache, mapping MACs to IPs, with a TTL for each (measured in seconds). If we don't find our destination IP in the cache, we'll broadcast an ARP request, saying "I, MAC addr A, want the MAC of IP Z." Then, each host will hear our request, check their own ARP caches, and send to MAC addr A "IP Z has MAC addr B". 

The choice of broadcast for requests and standard for responses lets us take advantage of all host's caches for requests, allowing for more efficient response times, while not clogging the network with duplicate broadcasts of responses. 

The best part for network administrators: this is all plug-and-play!

## ARP Spoofing

Since ARP is stateless and without authentication, an attacker can send ARP responses associating MAC addresses known to be in use within the network with malicious IPs, meaning that any packet meant to be sent to a target MAC will instead be sent to a malicious host. 

Detection mainly revolves around watching the ARP cache, and seeing if there are suspicious activities like multiple IPs associated with a single MAC. 
