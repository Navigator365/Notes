
How can we connect devices together at the link layer? 
- Starting simple, we can connect any two devices together with point-to-point links. 
- Next, we can use a bus, or a shared cable linking together devices. This isn't that good, since we have a single point of failure that's not easily fixable: if the cable is bad, we'll have to tear it out of all the devices and rewire from scratch. 
- Moving up, we have the **hub**, a physical layer device, using a star topology. Every device sends their frames to the hub, which broadcasts them to every other device. While this has a single point of failure, it's easier to replace. However, it can still cause collisions if two devices transmit to the hub at the same time. We also can't support multiple LAN technologies, and we're limited to a maximum number of nodes and distance. 

We can join LANs together through **repeaters**, which can amplify electrical signals passing through them. 

Finally, we have the magic of switches. They can support concurrent connections, allowing hosts to talk to each other simultaneously, all without any collisions. This happens thanks to switch buffers, which are similar to router queues: the switches  can store incoming communications in these buffers to ensure that only one message is being transmitted at a time. They can expand the size of our networks, support heterogeneous links and LAN protocols, and have improved management options. Finally, they're really easy to configure, thanks to self-learning. 

# Self-Learning

While a forwarding table on a router has to be prepopulated by a networking administrator, switches can learn how to route a frame all by themselves. It works like this: 

Every switch has a switch table, storing a mac address, the interface where that mac address should be forwarded to, and the time when that entry was placed in the table. 

Every time a switch receives a frame, it tries the following 2 things: 
- Check if the source MAC is in the switch table. If not, put the source MAC in along with the interface it came from. We'll use the timer to eventually forget this mapping. 
- Is the destination MAC in the table? If not, broadcast the frame across all interfaces except the one where it came from. If it is, then forward the frame along that interface. 

# Switches and Routers

Switches do have some limitations: to prevent broadcast messages from cycling forever, they're restricted to a spanning tree topology. Larger networks would also require larger ARP tables and substantial ARP traffic to maintain switching. Finally, if one device goes crazy and starts spamming Ethernet frames, that could be enough to take down the entire network. While routers aren't plug-and-play, and take longer to process packets, they do prevent endless forwarding through TTLs, allowing for diverse topologies, don't need any ARPing, and are immune to layer-2 spam. In general, if a network is using more than a few hundred hosts, use routers. 


