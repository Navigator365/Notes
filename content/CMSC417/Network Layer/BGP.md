How can we connect the Internet together? The Internet is a network of networks; we covered some routing protocols to handle communication within networks. We use BGP, or Border Gateway Protocol, to communicate **between** networks. 

Here, we're routing in terms of CIDR prefixes. At the end of the day, our router wants to put an entry (x, I) in its forwarding table, where x is a prefix and I is the interface it should send it to, so that the interface reaches its proper destination. BGP helps us do that. 

First, we organize networks into Autonomous Systems (AS's), each with a unique ASN. Routers inside these ASN's each have their own forwarding table, which is built by both intra-ASN routing algorithms and inter-ASN routing algorithms. 
- **Intra-ASN** protocols, like OSPF (LS, authenticated, multiple same-cost paths allowed, hierarchical, configurable costs) or RIP, help routers learn about routes inside their own ASN. 
- **Inter-ASN** protocols help routers learn about routes that span beyond their own ASN (though they can still include stops along routers inside the ASN). BGP uses two mini-protocols to accomplish this: 
	- **eBGP** obtains subnet reachability information from neighboring ASes
	- **iBGP** propagates reachability information to all AS-internal routers

So, when communicating with a router of a different AS, we'll use eBGP, and when communicating with a router inside our AS, we'll use iBGP. 

# Advertising Information

Now that we know the basic structure of BGP, let's talk about how we can share routing information through it. Gateway routers in an AS will advertise routes they know about, allowing neighboring AS's to learn about available paths. These advertisements, along with all BGP messages, contain the destination prefix along with a series of attributes. Two fundamental attributes are the AS-PATH and NEXT-HOP attributes. 
- **AS-PATH**: This is a sequence of ASes that will allow a given AS to reach a path. Think of it like a path the advertiser knows about. 
- **NEXT-HOP**: This is the IP address of the gateway router that should be used as the next-hop in the path towards the destination prefix (or more simply, the IP of the router beginning the AS-PATH). When a message is sent across ASes, this is set to the IP of the interface of the sending router. 

When a gateway router receives a BGP advertisement from another ASN over eBGP, it will communicate that route to all the other routers in its ASN via iBGP. Then, assuming the routers choose that route based on the ASN's policies, they will figure out how to forward messages to the next-hop router based on their intra-AS routing algorithm (like OSPF or RIP). 

## Route Prioritization

What if we learn about multiple routes to a prefix? Then, we'll chose one based on the following criteria, in order. 
1. Use the local preference value attribute, which can be set by a router or learned from another router inside the AS. This is an internal value specific to an AS,  set by network administrators. This allows admins to favor one route (say, choosing to route through a primary route instead of a backup route, if both are available). 
2. Choose the route with the smallest AS-PATH. 
3. Choose the route with the closest NEXT-HOP router (this is called **hot potato routing,** where we try to get rid of external data from our AS as quickly as possible, without thinking about the cost within other AS's)
4. Use some other tiebreakers/BGP attributes. 

# Routing Policies

While those rules are nice, we often need additional policies for our specific AS's use cases. 

![[Pasted image 20251210164143.png]]

**Customer/Provider Relationships**
For example, say we're an ISP access network AS X. We want all of our incoming traffic to be destined to us, and all of our outgoing traffic to have originated from us. In other words, we're here to provide networking services to our customers, not forward or transmit other people's messages. So how to we prevent other people from trying to send messages through us? 

We'll only advertise routes that have ourselves as a destination. For example, we know about the path XCY, but we won't advertise it to B. Since B never knows about our path, it'll never send us traffic that's meant to be sent to Y; just the traffic that should end with us. 

**Peering Relationships**

Providers can negotiate agreements called **peering**, where they agree to provide transit between their respective customers (so traffic can go from me to you, but not from you to someone else unless we've negotiated an agreement with them too). This allows for some efficient hierarchical peering, which can scale globally (if you're trying to send a message to a very distant destination, go up the chain of providers through peering agreements). But how do we implement this? 

For example, say B learns of the path A W, and advertises B A W to its customer X. Should it advertise the path B A W to C? If it does, then C could route traffic to W through B. Now, C can freeload off of B, not having to make a direct connection to A to transmit messages destined for W. While there's no firm policy in place to prevent this, in practice, any traffic flowing across a backbone network of an ISP must have either a source or destination IP in a network of a customer of that ISP; otherwise, we're giving out free rides. 


We can also use importing/exporting policies. We'll accept routes from everyone, but when exporting, we'll be more careful: 
- Advertise all routes to customers
- Advertise customer and ISP routes to peers and providers

There's much more we can do, but as you can see, we have more control over outbound traffic (choosing which routes we'll take), than incoming traffic (choosing who will form paths that involve us). 

**Interdomain Loop Prevention**

To prevent loops between AS's, we'll never accept a route with an AS-PATH containing our ASN. 

**Inbound Traffic Control**

While local preference values help us control outbound traffic, we can control inbound traffic similarly with AS-PATH padding. If we as AS 2 have a primary link and a secondary link, we'll advertise the path 2 on the primary and 2 2 2 2 2 2 on the secondary. Since our secondary link is advertising a longer AS-PATH, other ASes are less likely to choose that path. However, they still could! Another AS could have a local preference set that causes the secondary path to still be chosen. To control this, we can use BGP community attributes. These attributes are structured as ASN:value, where the value is whatever that particular AS chooses it to mean. We can include these community attributes in our advertisements and make the value mean a local preference value, telling a specific AS to value our secondary route less. 

**Minimizing Inter-AS Path**

With hot potato routing, we're choosing the path that's the lowest-cost inside our own AS. But we may not want this! Say we're a provider, and we're responding to a request from a customer. It's a huge request. Our lowest-cost path inside our own AS may lead to a higher-cost path inside their AS. Since we're a provider, we likely have a higher capacity network than them, and they may want us to share more of the burden. That means that we need to choose the route with the lowest-cost route inside the **customer's** AS. To fix this, we can use the MED (Multi-Exit Discriminator) attribute. The customer will advertise the MED cost in annoucements from each gateway router, representing the cost to get to the advertised prefix from that router inside their AS. Then, we as the provider can choose the route with the minimal MED cost. This means that MED values must be considered before hot potato routing, but after AS-PATH length, if we choose to use them. 


# BGP Attacks

**Prefix hijacking**: Start advertising yourself as the destination of an already-existing prefix route. That way, neighboring ASes will choose to route through you, since you will have the shortest AS-PATH. 

**Sub-prefix hijacking**: Originating a more-specific prefix of an existing advertised prefix. Due to longest-prefix matching, every AS is going to choose your advertised routes to get to the target prefix, meaning the entire Internet will route destined for that prefix traffic to you. 

**Bogus AS-PATHS**: you can add an AS as a final hop to a path. If it's legitimate, you can mask your bogus route, and it's hard for other people to realize your path is nonsense. Similarly, you can remove ASes from the path, generating shorter paths and encouraging more people who wouldn't normally route through an AS to route through it. Finally, you can add yourself in the middle of a path, to make your path seem more well-connected or to DoS the AS you added. Finally, you can choose to export your prefix at only one gateway router, which could be a crude form of enforcing MED requirements described above. 