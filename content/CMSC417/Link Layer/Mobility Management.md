
We want to maintain the ability to talk to devices as they move to different networks. This property is called mobility, and we have a couple ways to provide it. Note that the section below is specific to Mobile IP, which iirc isn't used a whole lot. 4/5G mobility is far more common. 

First, some terminology and setup. A mobile device will be assigned a **permanent address** at its "home" network (referring to the device's "home", not imposing any specific properties), which will not change. A **home agent** at that network will handle requests related to mobility.  When a device joins a **visited network**, it contacts the network's **foreign agent**, which will perform mobility functions on behalf of that device. The foreign agent assigns the device a **care-of-address**, or an IP that it will use in the visited network. **Correspondents** are any devices that want to talk to the mobile device. 

To set up mobility, a device goes through a process called **registration**. It contacts the foreign agent and provides it with its permanent IP address, which assigns it a care-of address. The foreign agent then reaches out through the network using the permanent IP address, contacting the home agent and providing it with the care-of address. This way, both the foreign agent and the home agent know where the mobile device is. 

Then, there are two strategies for correspondents to contact the mobile device: indirect routing, and direct routing. 

# Indirect routing

In indirect routing, the correspondent uses the permanent address of the mobile device, contacting the home agent which will then reach out across the network using the care-of address the foreign agent gave it. It will encapsulate the correspondent's message with an IP datagram with a destination of the care-of address. The foreign agent will receive this datagram, decapsulate it, and forward the message to the mobile device. 

Note that we don't necessarily need an independent foreign agent; our device can do all the work itself. However, this pattern of routing from correspondent-home-foreign, called triangle routing, is inefficient if the correspondent is inside the foreign agent's network, since it can just talk to the foreign agent directly as it knows where it is. 
![[Pasted image 20251130154854.png]]

# Direct routing

Just like in indirect routing, the correspondent uses the device's permanent address to reach out to the home agent. However, the home agent will respond to the correspondent under direct routing, giving it the care-of address of the device. The correspondent will then contact the foreign agent using the address (which it knows how to reach, since it's on the same network), which will then forward the message to the device itself. ![[Pasted image 20251130155032.png]]

# Mobility across multiple agents

We can extend this process as the device moves across different foreign networks. If a network joins a new foreign network after previously joining another foreign network, that new network's foreign agent will inform the previous, or "anchor" foreign agent, of the change. That way, whenever the home agent or correspondent contacts the anchor foreign agent, it can redirect to the new foreign agent to deliver the message. 

TODO: Not super confident on this, the slides just had this picture
![[Pasted image 20251130155605.png]]