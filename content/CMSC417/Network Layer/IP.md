The ultimate network layer protocol, and probably even one of the ultimate Internet protocols, besides BGP. 

The basic idea: the IP protocol will deliver our data between different networks on a best-effort basis (reliability not guaranteed). 
## The Datagram View

Let's start by taking a look at an IPv4 datagram (we call network-layer packets *datagrams*):
![[Pasted image 20250923114755.png]]

Everything before data is all a part of the header, arranged in 32-bit/4 byte rows. 

Some info on the non-obvious/interesting fields: 
- **Type of service**: Is it real-time or not? 
- **16-bit identifier/flags/fragmentation offset**: These all handle fragmentation, where a large datagram is decomposed into smaller datagrams, forwarded separately, then reassembled. Even though IPv6 doesn't care about fragmentation, 417 does so I'll cover it later. 
- **Header checksum**: A checksum formed from the datagram header; if a reciever computes a checksum out of the headers and gets a different result than the value in the header checksum field, some error or corruption has occurred.
- **Options**: a terrible idea. Originally designed to allow datagrams to optionally specify more fields. However, options can be of variable length, so it's not easy to tell where the data begins. Therefore, every IPv4 datagram must be scanned through to check for an option, and then process the option if it exists. This is not ideal for high-performance networking, so IPv6 doesn't have it. 

## Fragmentation

Unique to IPv4 is the problem of fragmentation. Each network has its own Maximum Transmission Unit (MTU), which doesn't necessarily correspond to the size of the largest datagram. So, how can routers disassemble, forward, and reassemble those datagrams into smaller datagrams?

First, the sender will choose a 16-bit identifier for all its fragments that will be unique over a given timespan. Then, it'll send each individual fragment. Each fragment's offset field provides the byte offset of the fragment's data in the whole data. Each of these offsets are aligned to 8 bytes. Also, every fragment but the last will have the M  (more coming) flag set, except for the last fragment. Then, the receiver knows what larger datagram each fragment belongs to, how to reassemble that datagram, and how to figure out when all fragments have been received to start the reassembly process. 

But what if we miss a fragment in the middle? IP doesn't guaruntee reliability, remember? Our router will think we've sent all our fragments as soon as it recieves one without the M flag set, and will start reassembly...only to realize it can't reassemble and give up. This process wastes time and storage (we need to store all the fragments we receive until we reassemble). This is the primary reason why IPv6 doesn't support fragmentation on routers: it's just too inefficient. 

You might be reading this and think "Hey, wouldn't IPv6 require fragmentation too?" Yes, but those problems are dealt with by the host through MTU path discovery, where hosts determine the minimal MTU along the path to the receiver. Then, the host can prefragment its data before sending.  