
There are two types of network links: point-to-point links and broadcast links. A point-to-point link consists of a single sender at one end of the link and a single receiver at the other end of the link. Many link-layer protocols have been designed for point-to-point links; the point-to-point
protocol (PPP) and high-level data link control (HDLC) are two such protocols. The second type of link, a broadcast link, can have multiple sending and receiving nodes all connected to the same, single, shared broadcast channel. The term broadcast is used here because when any one node transmits a frame, the channel broadcasts the frame and each of the other nodes receives a copy. Ethernet and wireless LANs are
examples of broadcast or multiple-access link-layer technologies. But how do we coordinate the access of multiple sending and receiving nodes on a shared broadcast channel? We have a couple of strategies at our disposal. 

# Channel Partitioning Protocols

We can avoid collisions by splitting our channel into designated regions for each node, and only allowing one node to talk in each region. That way, no nodes will collide. There are a couple of ways we can do this splitting: Time-division multiplexing does it by splitting the channel into time slots for each node, which is when that node is allowed to transmit. Frequency-division multiplexing similarly divides the channel into areas specific to each node by frequency.

![[Pasted image 20251129220749.png]]

While these do avoid collisions, each node is limited to an average sending rate of the channel rate / number of nodes. This is inefficient when one node's doing most of the sending, so we'll look to other technologies instead. 

Of note: there's another partitioning protocol called code-division multiple access, but it's used in WiFi so I'll add that when we cover it. 

# Taking Turns Protocols

These protocols try to strike a balance between providing throughput for every node and maximum throughput when only one node is talking. There are two important subcategories here, polling and token passing. 

In polling protocols, one master node polls all the other nodes one-by-one, telling each how much they can send. This prevents collisions and wasted frames, but it does introduce a dependency on a singular master node and increase delay from the time it takes for polling announcements to be processed by the receiving node. 

In token passing protocols, nodes exchange a token in a fixed order. Each node only holds onto the token while they have something to transmit, and pass it on once they're done or if they had nothing to say in the first place. While this also prevents collisions without a central dependency, losing the node with the token could prevent the entire network from communicating unless there's a way to detect crashes and reintroduce the token. 
# CSMA/CD

There are two rules for polite conversation. 
1. Listen before speaking, that is, if someone else is talking, wait until they are finished. In networking, this is called *carrier sensing*. 
2. If someone begins talking at the same time as you, stop talking. In networking, this is called *collision detection*. 
Together, we get **carrier sensing multiple access with collision detection**. 

Collisions can happen in this protocol: what makes it special is how it can recover from them. Here's how it works: 

Whenever a node has a frame to send, it will listen to the channel. If it doesn't detect an active transmission (a change in voltage at its location), it will send the frame with probability p (we call this **p-persistence**). If the channel is busy, it waits until it's idle. 

Note that under baseline CSMA, collisions can still happen. Say we see the channel is idle, and we start transmitting. Another node further away could see the channel is idle from its point of view, and start transmitting as well. Thus, these two frames will eventually cause a collision. 

That's where the CD part of the protocol comes in. While we're transmitting a frame, we're also listening to the channel. If we detect a collision, we'll immediately stop sending our frame, send out a jamming signal to alert all the other nodes that a collision's occurred, wait a random amount of time, and then try to listen and transmit again. We wait a random time to ensure that we won't continually have collisions: if two nodes collided, detected their collision, and waited the same amount of time, their frames would collide again once their timers expired. 

To determine this random time to wait, we use the **binary exponential backoff** algorithm, where after the nth collision, we randomly choose a number from the interval {0, 2, ..., 2^{n-1}} and multiply it by 512, and wait for that long. 

Since CD relies on nodes detecting a collision while they're still transmitting, we need to make sure our network can support this, even in the worst-case scenario. Let's say we're trying to detect a collision from the farthest node in our network, a distance L away. Our signal speed is c, so the total time it will take to reach this node from our node is L / c. At time L / c, our frame will start to arrive at this node, which will just have started transmitting and detect a collision. It'll send a jamming signal in response, which will take L / c time to arrive. So, we still need to be transmitting at time 2L/c. Assuming a fixed bandwith, our packets have to be at least a certain size to support this. Thus, we have the formula 

$$ \frac{P}{B} \geq 2 \left(\frac{L}{c} + delay\right)$$
Our packets must be at LEAST size P and our length must be at MOST size L. Additional one-way delay can come in the form of repeaters, which operate in microseconds or $10^{-6}$ . Also important to know are Mbps which is $10^6$. Also, km is $10^3$ . 

Note that CSMA/CD is hard to do in wireless, since that technology makes it difficult to transmit and receive data simultaneously. 
## Ethernet

Ethernet is a particularly common wired implementation of CSMA/CD. Its p-persistency is 1, meaning a node will transmit frames as soon as they're ready if it detects an idle channel. Its frames are somewhat interesting: 
- A type field to indicate the higher-layer protocol it's carrying (IP for example). This allows the receiving adapter to demultiplex and redirect to the appropriate handler. This functions a lot like ports for the transport layer. 
- A preamble with repeating values except for the last couple, which tell a node to wake up, lock onto the sender's clock, and start expecting data. 
- The CRC check, which allows the receiver to detect bit errors. 
- Addresses: if the destination mac matches ours, take it up, otherwise, discard it
Ethernet is connectionless and unreliable: while we can detect errors through CRC, we'll just discard the frame if there are errors; there's no acknowledgement or retransmission system. We'll leave that to the higher-level protocols like TCP to detect gaps in our data and request retransmissions. 
