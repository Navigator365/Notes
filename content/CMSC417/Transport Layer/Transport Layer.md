
The Transport Layer connects applications running on different hosts. This allows applications to message each other without dealing with network topologies, routing rules, or link connections required to transmit those messages. 

Unlike lower layers, Transport layer protocols are entirely implemented on end systems, not on network routers or links. Here's a useful metaphor to describe the difference between the network layer and the transport layer: 
- Say two houses are full of children who all want to send letters to the other house. 
- In both houses, one child is responsible for collecting all the children's messages, splitting them up into appropriately-sized letters, and sending them off to the post office. 
- Whenever there's mail, that same child will distribute letters from the other house to the matching child. 
Here, the post office is the network layer: it handles delivery. Meanwhile, the child is the transport layer: once a message is received, it redirects it to the appropriate application. 


We use 2 transport layer protocols: 
- UDP (User Datagram Protocol, connectionless)
- TCP (Transport Control Protocol, connection-based, reliable transfer, congestion-controlled)

However, both provide some fundamentals: 
- (De)multiplexing
- Error Detection
Also, we call transport-layer packets segments. When sending those segments on the network, we'll encapsulate **each** segment in its own IP datagram, and pull out the segment from datagram bodies when receiving messages. 
# Multiplexing

In the analogy above, multiple children could all write and receive letters, which would be handled by a single child. Each of these letters are application streams , which are implemented  as sockets. Delivering data from the transport layer to the correct socket is called **demultiplexing**, and the process of gathering data chunks from different sockets, encapsulating each chunk with a header to create a segment, and passing segments to the network layer is called **multiplexing**.

The (de)multiplexing process is different for TCP and UDP, but both rely on ports, which help us identify which socket a segment should be delivered to. Every networked application is tied to some port, and that port is associated with that application whenever it sends or receives messages. 

## UDP Multiplexing

As always, UDP is simple: we can uniquely identify a UDP socket through \[destination IP addr, destination port]. When multiplexing, we'll put our destination application's port in the segment containing our application data, and pass it to the network. When demultiplexing at our destination, we'll pull out that destination port, find the socket that port corresponds to, and deliver the segment to the socket. 

UDP does include the source port as well, just in case we want to send a message from the destination back to the source. Then, we'll reverse our port order, pull the source IP from the IP header, and send a message back to the socket uniquely identified by \[source IP, source Port]. 

## TCP Multiplexing

You might've realized that UDP multiplexing means that two packets from different devices sent on different ports, to the same device on the same port, will be directed to the same destination socket. This isn't vary convenient for a connection-oriented protocol like TCP: we'd like to have some logical separation of data from multiple devices to support multiple distinct connections. 

We can accomplish this by uniquely identifying TCP sockets through \[source IP, source Port, dest IP, dest Port]. This means that every device connecting to a TCP-based application will communicate with that application through its own socket. However, each socket doesn't have to be in its own process: for example, modern web servers maintain socket connections through threads. 



