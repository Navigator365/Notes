TCP is a connection-oriented protocol providing reliable transport along with a LOT of other things. 

# TCP Message Structure
![[Pasted image 20251101163554.png]]
Payload data is divided into segments based on the node's local MSS restrictions. These segments are identified by the offset into the bytestream each segment represents, starting from 0 and continuing onwards until the entire message has been divided into segments. 

# TCP Segment Structure

![[Pasted image 20251101163354.png]]
There are quite a few TCP fields, but many of them aren't used anymore. 
- **Sequence number**: This is the bytestream offset of the first byte in that segment's data. So, if the first byte of a segment was at an offset of 1,000 into the message's bytestream, the sequence number would be 1,000. In practice, a random sequence number is chosen to represent the start of the first message, rather than 0. So, the offset of 1,000 would produce a sequence number of (random value) +  1,000. 
	- Sequence numbers are 32-bit values, so there's a potential for an eventual wraparound on overflow. To prevent later messages from looking like earlier messages, we can optionally attach a timestamp to differentiate later messages. 
- **Acknowledgement number**: This represents the next byte we expect to receive, or equivalently, the next sequence number we expect to receive from the other host.
- **Flags**: All we care about is the SYN and ACK flag, indicating if a packet is trying to synchronize or acknowledge, respectively. The PSH flag indicates the reciever should pass the data to the application layer immediately, and the URG flag indicates that the sender has marked this segment's data as important. 
	- The urgent pointer refers to the last byte of this urgent data. 
# 3-Way Handshake

In order to communicate, both hosts need to generate their own sequence numbers, share them with the other, and acknowledge that they received the other's sequence number. 

Here's how it works: 
- **SYN**: A host generates a random sequence number (called an Inital Sequence Number, ISN), and sends that to the other host with the SYN flag set. 
- **SYN-ACK**: The receiver responds with by generating their own random sequence number to use, along with an acknowledgement number of the sender's sequence number + 1, and both the SYN and ACK flags set. It's called SYN-ACK because it's doing two things at once: synchronizing its own sequence number with the sender, and acknowledging the sender's sequence number. 
- **ACK**: The sender will send out a sequence number set to the receiver's acknolwedgement number, an acknowledgement number of the receiver's sequence number + 1, and the ACK flag set. This flag can also include data as well, while the other two messages can't, as the two host's sequence numbers hadn't been set up yet, so there would be no way to determine where the data should be stored. 

If any packet gets lost in this handshake process, the host will resend it after a timer elapses. 

![[Pasted image 20251101165406.png]]

To tear down a connection, a host sends a Fin, waits for an Ack, waits for the other host to send a Fin, and then sends an Ack. Alternatively, the host can send a reset RST to not have to deal with this ack process. 
- FINs are sent whenever a socket is closed, and received when a socket returns an EOF. 

# Basic Transfer

Let's start by considering a basic communication model, where one host sends a message and waits for an ACK from the receiver before sending again. Note that this is not how TCP works, but it's an easy mental model for introducing some fundamental concepts. One thing I will note early on: TCP is a full-duplex protocol, meaning both hosts and send and receive data. However, for simplicity, I'll only speak in terms of a sender and receiver, since the protocol functions the same if they switch. 

## Sequence numbers

We'll start by including a sequence number with every message, which specifies which byte in our sending buffer is the start of our message. This tells the receiver where to put the data we're sending, and helps prevent duplicate messages. For example, if the receiver has already been sent a packet with a certain sequence number, it can check its buffer and see that the position corresponding to that sequence number is filled, so it can send an ACK back without storing the data.   
## Timeouts

What if we don't get an ACK back? Say the receiver's ACK  (or the sender's message) were lost. Well, we can detect and prevent this by setting a timer whenever the sender transmits a message, and  retransmitting that message whenever the timer expires and we haven't received an ack with the acknowledgement number of our sequence number + data. 

But how long should this timeout be? If it's too short, we'll retransmit duplicate messages too often. If it's too long, we'll experience excessive delays when a packet's actually lost. We'll set our timeout to be based on the round-trip time (RTT) between sending a message and receiving an ACK back. 

Since we can't calculate the RTT directly for every message, we try to estimate it based on data we collect over time. Every so often (usually once a RTT), we'll pick a segment to act as a sample. We won't use any retransmitted segment, since we could receive a late ACK from the originally transmitted message, and mistake that time for the RTT, which would be a massive underestimate. In fact, we stop taking samples until we've observed a regular transmission-ack. 

We'll track when we send it and when we receive it, and derive a new estimated RTT for our timers as follows: 

$Estimated = (1- a) * Estimated + a*sample$

We'll double this interval every time we timeout and retransmit. Why? Our timers are most likely expiring because our network is congested. If we retransmit packets with the same average RTT, that congestion could get worse. By backing off, we hope the network congestion sorts itself out. 

For reference, these elements together form the **Karn-Partridge algorithm**: 
1. Stop taking RTT samples on TCP retransmit
2. Start taking samples again only after a successful send-ack
3. Double timeouts every retransmit
## Acknowledgement numbers

Acknowledgement numbers are included in ACKs the receiver send to the sender, telling it what data the receiver actually got. TCP uses **cumulative acknowledgement**, meaning its ack number is the first missing byte in the stream. This helps us detect and prevent packet loss. 

For example, suppose that Host A has received one segment from Host B containing bytes 0 through 535 and another segment containing bytes 900 through 1,000. Even though these segments are noncontiguous, we'll still store them both (for the most part, see sliding window discussions later). 

For some reason Host A has not yet received bytes 536 through 899. In this example, Host A is still waiting for byte 536 (and beyond) in order to re-create B’s data stream. Thanks to cumulative acknowledgement, A’s next segment to B will contain 536 in the acknowledgment number field.

Whenever Host A sends bytes 536 to 899 (after timing out without receiving an acknowledgement number field from B with those ranges), it'll retransmit that byte range. Assuming host B recieves that retransmission, its acknowledgement number will then be 1,000. 

# Sliding Window

Waiting for an ACK before retransmitting a message is crazy inefficient. Its utilization rate is crazy slow: 

$$U = \frac{\frac{L}{R}}{RTT + \frac{L}{R}}$$

In practice, TCP uses a sliding window to allow multiple messages (N) to be pipelined and sent simultaneously. 

$$U = \frac{N * \frac{L}{R}}{RTT + \frac{L}{R}}$$There are two forms of sliding window protocols, and TCP uses a mixture of both. 
![[Pasted image 20251109170336.png]]
Like Go-back-N, TCP sends back cumulative ACKs, and like selective repeat, TCP will only transmit unACKed packets when their timers expire (and even then, only if it didn't recieve a larger acknowledgement number). 

We call it a sliding window since we can think of our message buffer as a window of "active" segments. Every time we ACK a message, we advance our window to another segment, allowing that segment to be sent as well. 

However, using a sliding window means both the sender and receiver need to have space in their buffers for all messages that could be sent in the same window. It's fine if the reciever doesn't have all the messages that the sender is currently sending, but if the sender moves too far ahead, we could overflow the receiver's storage capacity and lead to dropped packets. 

## Fast Retransmit

With a sliding window, we can detect packet loss slightly faster. If we receive three duplicate ACKs, or ACKs with the same acknowledgement number corresponding to an earlier sent message, we know the message we sent after it wasn't received, and can retransmit it then instead of waiting for its timer to expire. 
# Flow Control

We call the process of managing the sliding window size flow control, matching the rate the sender can send with the rate the receiver and read. 

The sender keeps track of a **receive window**, referring to how much free space the receiver has. 

The receiver keeps track of an **advertised window**, which it derives from calculating
	advertised = buffSize - (lastByteReceived - LastByteRead)
Note that while the buffer size is fixed by the receiver, the other two variables are dynamic based on the connection (receieved) and internal processing (read), so the advertised window is dynamic as well. The receiver includes this in its TCP header. 

The sender grabs this advertised window out from the headers of ACKs, and compares it to the amount of unACK'd data it's sent out, or LastByteSent - LastByteAcked. Before sending any more segments out, the sender checks that 
	LastByteSent - LastByteAcked $\leq$ advertised window

There's one more thing we need to do: if the advertised window ever becomes 0 whenever the receiver is full, the sender will not know when the receiver's buffer is no longer full (unless the receiver starts sending its own messages to the sender). So, the sender will probe the receiver by sending 1-byte segments, which the receiver will acknowledge, until the advertised window is no longer 0. 

## Silly Window Syndrome

We say silly window syndrome occurs whenever the advertised window becomes "sillily" small, resulting in sending a series of small packets due to this small advertised window. This happens when the receiver consumes data slowly, or the sender sends too quickly and fills up the receiver's buffer. This leads to sending a series of small messages instead of fewer but larger messages, which in turn wastes bandwith since we get less data but have to process the same amount of TCP headers. 

We can prevent this using two strategies: 
1. **Delayed Ack**: We'll only ack every other message, or until a timeout of 0.5 seconds, and piggyback it with data our "receiver" wants to send. This helps reduce processing overhead by reducing the total number of packets (and headers) that need to be processed. In general, this isn't too useful unless our receiver plans to respond almost immediately to data sent; otherwise, we're needlessly delaying our acks. 
2. **Nagle's algorithm**: Here, we try to always send full-size TCP segments. If either our window or available data to send are less than the MSS, we'll wait for any unacknowledged data to receive an ACK. In the meantime, we'll combine this small available data together in the hopes that we can send a larger (hopefully MSS-sized message). Note that if we don't have any unacknowledged data, we'll send the message immediately, even if it is small.
	1. This doesn't mesh well with delayed ack at all: since Nagle's blocks for acknowledgements, it doesn't send out a second message for the delayed ack to acknowledge. This means that the ack will only come when the timeout expires. Then, the sender has to receive the ack before it can send all the data it's been gathering. This timeout + trip time increase to sending time is unacceptable, so we don't enable Nagle's algorithm and delayed acks simultaneously. 
