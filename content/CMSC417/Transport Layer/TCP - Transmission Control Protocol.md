TCP is a connection-oriented protocol providing reliable transport along with a LOT of other things. 

# TCP Message Structure
![[Pasted image 20251101163554.png]]
Payload data is divided into segments based on the node's local MSS restrictions. These segments are identified by the offset into the bytestream each segment represents, starting from 0 and continuing onwards until the entire message has been divided into segments. 

# TCP Segment Structure

![[Pasted image 20251101163354.png]]
There are quite a few TCP fields, but many of them aren't used anymore. 
- **Sequence number**: This is the bytestream offset of the first byte in that segment's data. So, if the first byte of a segment was at an offset of 1,000 into the message's bytestream, the sequence number would be 1,000. In practice, a random sequence number is chosen to represent the start of the first message, rather than 0. So, the offset of 1,000 would produce a sequence number of (random value) +  1,000. 
- **Acknowledgement number**: This represents the sequence number the host expects the other host to send next.
- **Flags**: All we care about is the SYN and ACK flag, indicating if a packet is trying to synchronize or acknowledge, respectively. 
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

# Data Transfer

A data field's acknowledgement number is a cumulative acknowledgement, meaning that it represents the highest sequence number such that every byte before it has been received. 

For example, suppose that Host A has received one segment from Host B containing bytes 0 through 535 and another segment containing bytes 900 through 1,000. For some reason Host A has not yet received bytes 536 through 899. In this example, Host A is still waiting for byte 536 (and beyond) in order to re-create B’s
data stream. Thus, A’s next segment to B will contain 536 in the acknowledgment number field.

Whenever Host A sends bytes 536 to 899 (after timing out without receiving an acknowledgement number field from B with those ranges), it'll retransmit that byte range. Assuming host B recieves that retransmission, its acknowledgement number will then be 1,000. 