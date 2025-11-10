
It's basic! It's simple! It's UDP! Apart from multiplexing, it basically passes application data directly into IP datagrams. 

# UDP Structure

![[Pasted image 20251027181420.png]]

Pretty basic, just the destination port, the source port to provide return-messaging capability (mentioned in [[Transport Layer#UDP Multiplexing|UDP multiplexing]]), a checksum, the total segment length (header + data), and the data. We need length, since the size of a UDP segment varies based on its data. 
# Error Checking
It does provide end-to-end error checking through a checksum on the whole segment, which IP doesn't provide (v4 only checksums its headers). But link-layer protocols like Ethernet do! What gives? Well, those protocols help us detect errors in link transmission. It's still possible for routers to corrupt segments they're storing in their memories prior to transmission, and not all link-layer protocols provide error detection. 

Error-checking is necessary here, since it's the only time we'll have a complete view of what we're sending across networks. However, UDP only error-checks: if we find an error, there's no scheme by which we can correct it. Application-layer protocols using UDP will have to write their own retransmission processes to correct any errors UDP catches. 

We actually compute the checksum by taking our data, splitting the data into 16-bit chunks, and adding all those chunks with overflow. 

# Why use it?

With no error handling or support for connections, why does anyone use UDP? Well, there are a couple reasons: 
- Fine-grain control/no connection setup delay: Since UDP doesn't have congestion control, you can send what you want whenever you want. Once an application has a message to send, UDP will immediately encapsulate it and pass it to the network layer. 
- No connection state: Sometimes you don't want a connection. DNS resolution is a one-time process: once you've got a response from a DNS server, you're done. Setting up a connection every time you'd want to make a DNS request is too slow. 
- Small header overhead: UDP packets are REALLY small: only 8 bytes in the header

Multimedia streaming applications that prioritize live, real-time messaging will use UDP, along with simple query-response protocols like DNS. 