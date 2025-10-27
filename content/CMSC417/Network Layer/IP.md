The ultimate network layer protocol, and probably even one of the ultimate Internet protocols, besides BGP. 

The basic idea: the IP protocol will deliver our data between different networks on a best-effort basis (reliability not guaranteed). 
## IPv4

Let's start by taking a look at an IPv4 datagram (we call network-layer packets *datagrams*):
![[Pasted image 20250923114755.png]]

Everything before data is all a part of the header, arranged in 32-bit/4 byte rows. 

Some info on the non-obvious/interesting fields: 
- **Type of service**: Distinguishes types of datagrams (for example, Is it real-time or not?) 
- **16-bit identifier/flags/fragmentation offset**: These all handle fragmentation, where a large datagram is decomposed into smaller datagrams, forwarded separately, then reassembled. Even though IPv6 doesn't care about fragmentation, 417 does so I'll cover it later. 
- **Header checksum**: A checksum formed from the datagram header; if a reciever computes a checksum out of the headers and gets a different result than the value in the header checksum field, some error or corruption has occurred. If you're thinking this is redundant since Transport and Link layers do some type of checksumming, you're right! IPv6 doesn't have this. 
- **Time to live:** How many hops an IP datagram should last for. Every receiver decrements the TTL before sending it. When a receiver notices a datagram's TTL is 0, it will send back an [[IP#ICMP|ICMP]] warning message to the source. 
- **Options**: a terrible idea. Originally designed to allow datagrams to optionally specify more fields. However, options can be of variable length, so it's not easy to tell where the data begins. Therefore, every IPv4 datagram must be scanned through to check for an option, and then process the option if it exists. This is not ideal for high-performance networking, so IPv6 doesn't have it. 

###  Fragmentation

Unique to IPv4 is the problem of fragmentation. Each network has its own Maximum Transmission Unit (MTU), which doesn't necessarily correspond to the size of the largest datagram. So, how can routers disassemble, forward, and reassemble those datagrams into smaller datagrams?

First, the sender will choose a 16-bit identifier for all its fragments that will be unique over a given timespan. Then, it'll send each individual fragment. Each fragment's offset field provides the byte offset of the fragment's data in the whole data. Each of these offsets are aligned to 8 bytes. Also, every fragment but the last will have the M  (more coming) flag set, except for the last fragment. Then, the receiver knows what larger datagram each fragment belongs to, how to reassemble that datagram, and how to figure out when all fragments have been received to start the reassembly process. 

But what if we miss a fragment in the middle? IP doesn't guaruntee reliability, remember? Our router will think we've sent all our fragments as soon as it recieves one without the M flag set, and will start reassembly...only to realize it can't reassemble and give up. This process wastes time and storage (we need to store all the fragments we receive until we reassemble). This is the primary reason why IPv6 doesn't support fragmentation on routers: it's just too inefficient. 

You might be reading this and think "Hey, wouldn't IPv6 require fragmentation too?" Yes, but those problems are dealt with by the host through MTU path discovery, where hosts determine the minimal MTU along the path to the receiver. Then, the host can prefragment its data before sending.  


FLAGS: DF=1, don't fragment, MF=1, more fragments, 0 if last fragment. If DF=1 and MTU too small, drop the packet. 

## IPv6


![[Pasted image 20251020214829.png]]

Some discussion on fields:
- **Traffic class**: Similar to Type of Service in IPv4, only this time based on priority rather than strictly type. Allows certain datagram types to be marked as "higher priority"
- **Flow label**: Some traffic flows may have special properties, like real-time service or audio-visual transmission, that distinguish them from normal traffic. This field identifies a datagram as part of a larger flow. 
- **Payload length**: NO MORE PACKET LENGTH! Since the packet is fixed-size and we only care about data, payload length is just the size of our payload.
- **Next hdr**: The protocol we're delivering datagram contents to (TCP, UDP)
- **Hop limit**: TTL but no longer confusingly-named "time"

IPv6 gives us some more communication options: 
![[Pasted image 20251020222758.png]]

To handle connections through networks where not all devices support IPv6, we can use tunneling by wrapping IPv6 packets inside IPv4 packets. 
## Addressing 

IP addresses are 32-bit addresses separated into byte-blocks with a`.`, which we call dotted-quad notation. IP is designed to facilitate internetworking, so we need to have some concept of what larger network we're a part of, and a unique host identifier. When combined, each IP address should be globally unique and hierarchical (network/host). 

### Classful 
The naive way to do this is through **classfull addressing**. To account for different networks with different amounts of hosts, an address will be allocated to one of several classes. The first few bits tell us what class the address belongs to, the next are its network id, and the rest are its host id. 
![[Pasted image 20250927174754.png]]
Incidentally, probability tells us that using this scheme, half of all addresses will be in class A, 1/4th will be in B, and 1/8th will be in C. 

If you want to get IP addresses from ICANN/RIRs, you can only do so in fixed-size blocks under classfull addressing. For example, if you have 10 million hosts or 66,000, you'll both need to buy out a class A network ID to populate with your host IDs. As you might imagine, this uses up our available address space very quickly, which is not good if we want the Internet to keep expanding!

This strategy also takes up space in router forwarding tables: every time a new block is registered, that's another unique network ID for routers to keep track of. We don't want degraded performance either! 

### Forwarding 

Forwarding, or the process of sending data to the next interface, uses the algorithm below. Each router stores a forwarding table, containing mappings from network ID -> next hop for given network IDs, those that are part of its interfaces, and a default router (since obviously no router can store every network ID, so if we don't find a match just send it there). The algorithm is as follows: 

```
if (NetworkNum of destination = NetworkNum of one of my interfaces) then
    deliver packet to destination over that interface
else
    if (NetworkNum of destination is in my forwarding table) then
        deliver packet to NextHop router
    else
        deliver packet to default router
```

By only using network numbers instead of host numbers, we drastically reduce the amount of information routers have to store. Now that we know where to send, we'll use ARP to begin the process of sending. 
### Subnetting

We have some range of IPs assigned to us. Most likely, they're all inside a single class block. That means we have a single network ID to work with. But what if we want to host and segment multiple internal networks? We don't want to buy another identifier block, since all these internal networks should resolve to the same network on the outside. We'll do something called subnetting. 

We'll derive our internal network ids, called subnet ids, by ANDing our host addresses and a subnet mask that every host knows about. We can derive the subnet mask based on the max amount of  hosts we expect any of our subnets to have, and use the remaining bits as part of the mask. When calculated, subnet ids will act as network ids within our network. 

We can form multiple subnets out of a single mask: for example, we can mask based on whether one bit is set or not, and produce 2 subnets from the IPs we have assigned to us. 

When deriving masks, think about which place-values shouldn't be part of the host values: those are the ones that should be 1s in your bitmask! If all else fails, use / notation!


#### Forwarding
Subnetting does complicate forwarding a little bit: instead of just storing the network id, we'll store (subnet id, subnet mask, next hop) of each router, and only route to an entry's next hop if our destination ip addr & that entry's netmask = that entry's subnet id. 

Our new psuedocode: 
```
D = destination IP address
for each forwarding table entry (SubnetNumber, SubnetMask, NextHop)
    D1 = SubnetMask & D
    if D1 = SubnetNumber
        if NextHop is an interface
            deliver datagram directly to destination
        else
            deliver datagram to NextHop (a router)
```

### Special IPs

Before we go any further, let's explain 2 special IPs: 
- an address with host bits all set to 0 is the "network address", used to identify the network, and is not assigned to a host
- an address with host bits all set to 1 is a "broadcast" address, which routers will interpret as a command to send the message to all devices on the subnet (if we're sending this from another subnet, it's called a *directed broadcast*)
	- *Limited broadcasts* are a special type of broadcast, 255.255.255.255, which will never be forwarded by routers. 

### CIDR

Classless Interdomain Routing is the solution to all our IP utilization problems! Instead of having to use a dedicated IP block, we'll be able to specify how many bit we'll need to identify all the hosts on all the networks we own, and in turn reserve only 1 network id for our entire organization. 

One important clarification: this is not the same as subnetting! We're not trying to divy up internal networks, but allocate IPs in a way that reduces their consumption and reduces route table storage. 

Here's how it works:  say we own 255.255.254.0/23. The slash notation means that the first 23 bits (called a **prefix**) of the IP are common to all of our addresses, and we own all possible values of the last 32-23= 9 bits. This way, every public router just has to store 255.255.254.0/23 in their routing tables, rather than clogging themselves up with our many hosts.

We can chain applications to CIDR to be precise in our allocations. For example, an ISP can aggregate results from nearby addresses 128.122.128/24 128.122.129/24.... into one external 128.112.128/21. We call this, unsurprisingly, **address aggregation**, and it further reduces routing table storage. 

![[Pasted image 20250929175929.png]]

#### Forwarding

CIDR also complicates forwarding. Now we store our prefixes in our routing tables, which can overlap. If we're trying to send something to address 201.10.6.17, should we listen to the entry for 201.10.0.0/21 or 201.10.6.0/23? 

The solution: we'll route to the longest-matching prefix in our routing table. There are some clever algorithms to do this quickly, but they're outside the scope of this class. 

## DHCP

Regardless of how we allocate IP addresses, we still need a way of assigning them to every host on our network. While we can manually do so, that doesn't scale well, and doesn't work for mobile technologies (say, someone who uses their laptop in a lecture hall and at their dorm room -> likely different subnets).

DHCP servers can dynamically configure hosts with IP addresses without manual user or network administrator intervention. Here's how they work: 
![[Pasted image 20251004184408.png]]

1. DHCP discover: A client broadcasts a discover message, checking if a DHCP server exists. Since the client doesn't have an IP address yet (that's the whole point), we use the source IP 0.0.0.0 (which basically translates to the default route when interpreted by the router). We use a limited broadcast as a destination, so we send this to everyone on our local network segment. The client includes a randomly-generated transaction ID, which will be included in all future messages in this exchange, so that it knows which server responses are talking to this particular client. Since two hosts could produce the same transaction ID, layer 2 communications include their MAC addresses, so the DHCP server can differentiate between two identical transaction IDs. 
2. . A DHCP server will broadcast (ie, send to everyone on the network, since there's no host IP to send to yet) a DHCP offer, with a proposed IP address,and a *lease time*, or how long that particular IP address would remain valid for if a host chose it. It can send other info too: 
	1. Address of the first-hop router
	2. name and IP of a DNS server
	3. subnet mask of the IP
3. The client listens to incoming messages and grabs a DHCP message with the right transaction ID. It sends a DHCP request echoing back the server's parameters. 
4. The server sends an Ack confirming IP assignment. 

Later on, the client and server can communicate to renew an address's lease. 

DHCP does have one big downside: it doesn't scale past subnets. To reach a DHCP server, we send a broadcast message, which is only sent to devices on our subnet. If we don't have a DHCP server on our subnet, are we out of luck?

NO! We can use a DHCP relay. Some device on our network will know the location of a DHCP server, and relay DHCP discover broadcasts and DHCP server responses. The DHCP server will also figure out the client's subnet, and assign an IP that's on that subnet. 
- The relay will add *Option 82* to the DHCP server, characterizing the specific remote network the server will be talking to. 

## NAT

When we connect a small office to a WAN, we'd need to allocate public-facing IPs for each device on the network. That's a lot of work. I guess DHCP could do it, but then that's a lot of addresses to issue. *Network Address Translation* (NAT) is a hack that allows us to get our home networks online while minimizing our IP usage. Here's how it works: 
- Our NAT-enabled router has a publicly-facing IP address assigned to it, often from an ISP's DHCP server. 
- Devices on our network are assigned private IPs (10/8, 172, 192). These don't route anywhere on the Internet, and the same IP can be used by thousands of devices around the world simultaneously, without any knowledge of each other. They're often assigned via a NAT-DHCP server run by the router. 
- When a device on the network sends a request to a public IP, our router will store its IP and port in the NAT table, and map it to one of its publicly facing IPs and a currently unused port. This IP and port will replace the IP and port in any packets from the device that the router sends out. These are then sent on the public Internet. 
- When a response comes back from the Internet, the router looks in the WAN side of its NAT table for the IP and port, and finds the private IP and port that corresponds. Then, it will replace the IP and port with what it found, and send that along the network. 
![[Pasted image 20251010200944.png]]
In this way, nearly $2^{16}$ devices on a private network can be serviced simultaneously through a single router with a single public IP. By separating public IP allocation from host setup, we can allow both to change without affecting the other. We can add or remove hosts without needing to deal with allocation, our ISP can change the IP allocated to us without having to reallocate all our hosts, all while ensuring nobody from the outside world can connect to our devices explicitly (unless they're able to infer the contents of our NAT table).  

However, NAT introduces some new problems. How can an external IP reach out to a device behind a NAT server if that device hasn't talked to the IP before? Say, a client connecting to a server hosted behind a NAT.  

## ICMP

While not a part of the IP protocol, ICMP is commonly associated with it. ICMP messages are carried in the bodies of IP datagrams. Let's talk about their structure: 
- A header and type code to refer to the specific error/control message the ICMP message represents
- The first 8 bytes of the IP datagram that prompted the receiver to send an ICMP message back to the sender. This lets the sender determine which of the datagrams they sent prompted the ICMP message. 
ICMP is generally used for error messaging, though it's also used for traceroute, since receivers will send an ICMP warning back to the sender if an IP datagram's TTL reaches 0. 

## Tunneling

Tunneling refers to the general process of connecting two networks through an abstracted "third network", with traffic from either end passing through a gateway connected to a gateway on the other side of this intermediate tunnel, encapsulating IP datagrams from one network with an IP datagram sent from one gateway to another. This process also encrypts network traffic in these gateway datagram bodies. 

![[Pasted image 20251018143625.png]]

Some advantages: 
- Increased security (avoiding potentially untrusted middlemen)
- Improved support for heterogenous networks/routers: Instead of having to figure out how to connect two differently-structured devices/networks together, we can wrap them in another network to share information between them
- Better support for multicasting (our gateway can connect us to an arbitrarily-sized network)
Some disadvantages: 
- This process increases packet length, which 
	- Wastes bandwitdth
	- Increases router processing delay
	- Can lead to fragmentation, and therefore reduced performance, on IPv4
- This increases the work network administrators have to do (this is definitely not plug-and-play)