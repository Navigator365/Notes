---
title: Link Layer
---
Say you book a vacation with a travel agency (old, I know). The travel agent books a taxi to the nearest airport, then several layover flights, then an arrival at the airport of your final destination, and a taxi to a prebooked hotel. 

While the travel agent, like the network layer, governs the transportation path a packet makes from its source to its destination, the drivers and pilots of each vehicle  handle transportation process along each link in the journey, like each of the many disparate link-layer protocols, concerned with transporting information from one node to another. 

We use [[Addressing and ARP|MAC addresses]] in link-layer packets, called frames, to uniquely identify a link-layer device. We can transmit data either to one device through a singular wired connection, or to many devices through [[Multiple Access Protocols]], and combine links together into networks through [[Switched LANs|switches]]. 

Each of these processes have to consider the problem of collisions, when two frames overlap each other in the same physical space. This makes both frames unintelligible. These protocols seek to mitigate collisions or entirely prevent them from happening. 

We can also use the link layer to support [[Wireless Links|wireless]] and [[Mobility Management|mobile technologies]], though each of these require new structures in place to support them. They also affect higher-layer protocols, with higher packet loss in wireless and disruptions and changes in RTT in mobility. While the impact should be minimal since IP's best-effort service model remains unchanged by these technologies, performance, especially TCP performance, is affected, since packet loss results in congestion control, RTT changes prompt changes in timeouts, and out-of-order packets lead to poor TCP performance. The Internet was not designed with these technologies and resulting issues in mind!