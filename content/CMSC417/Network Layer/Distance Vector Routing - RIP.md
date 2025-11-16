Our network is a graph, with nodes as routers and edges as links. These edges are weighted with what I'll call costs. We want to compute the least-cost distance to each router for all routers in our network. 

We want this computation to be **dynamic**, **multicasted** and **asynchronous**: 
- Our algorithm should respond to link outages or changes in link weights
- Our algorithm shouldn't require each router to communicate with every other router. 
- Our algorithm should function regardless of the order a router receives distance updates from other routers
## Algorithm

To perform distance-vector routing, each router $x$ will need to store the following: 
- A distance vector $D_x(y) = d$ where $y$ is every other router in our network and $d$ is the distance between $x$ and $y$. 
	- We also need to keep track of the next-hop router, or the router that's immediately after $x$ in the shortest path to $y$. While not strictly necessary, I'll store this in the distance vector as well. 
- A list of $cost(x, n)$ between $x$ and all neighbors $n$.
- All of its neighbors' distance vectors. 

First, we initialize our costs and distance vectors: 
- $D_x(*) = \infty$ 
- $D_x(n) = cost(x, n) \forall n \in$ neighbors
- Broadcast the distance vector to our neighbors. 

Then, we'll loop forever doing the following for any given router $x$ on [[Distance Vector Routing - RIP#Updates|triggered update or periodic update]]:
- $\forall y, D_x(y) = min_v(cost(x, v) + D_v(y))$ 
	- In English, we'll calculate the least-cost path to every router $y$ by, for each $y$, checking the distances using each neighbor $v$ as our next-hop. We'll pick the minimum, and store the chosen $v$ as our next-hop. Finally, we'll update our distance vector with the new least-cost path for each $y$.
	- This formula is called the Bellman-Ford formula, and is a big thing in DP for you leetcode lovers. 
- If ANY entry in our distance vector changed, broadcast the distance vector to our neighbors. 

Eventually, the distance vectors in each router will converge with the actual least-cost distances in the network. 
## Updates

We recalculate our distance vectors on two categories of updates: 
- **Triggered updates**: Any event. For example, a link cost to a neighbor might change, or we might recieve a distance vector from our neighbor. 
- **Periodic Updates**: Updates sent over a given interval. These are often used to detect links that are down: if you don't see your neighbors' periodic updates, you know the link between you two is dead. The only other way to detect links that are down is to constantly ping your neighbors, which could be expensive or distracting. 

## Counting to Infinity

While distance-vector routing generally works quite well, there are some situations that can put the algorithm in a loop thanks to its asynchronous nature. 

Let's look at a network like this, where one of the costs changes. 

![[Pasted image 20250914184924.png]]

Let's say $y$ detects the cost change first, and runs Bellman-Ford. It does so, and it gets    $D_y(x) = min(cost(y, x) + D_x(x), cost(y, z) + D_z(y)) = min(60, 6)$. Why 6? Well, $z$ hasn't updated yet: prior to this, the least-cost path between $z$ and $x$ was through $y$, so $y$ still has that old info on hand. 

Since $y$ has its distance vector changed, it'll send its new vector to its neighbors. $z$ picks it up, and calculates $D_z(x) = min(50, 1 + D_y(x)) = min(50, 7).$ Since it updated, $z$ will send its distance vector to $y$, and this process will keep on repeating!

Eventually, it'll stop when $D_y(x) > 50$, since $D_z(x)$ will select a minimum and no longer change on every triggered update, but that's still an awful lot of wasted time. 

This problem is called the **count-to-infinity problem.** While we don't always count to infinity (we usually stop once we hit a min condition, unless our only other option is infinity in the case of a down link), we want our distance vectors to quickly converge to the actual least-cost paths, not this meandering process. Luckily, we have a couple of mitigations: 

- **Hard cap**: Define infinity as a small enough number, and we won't waste too much time. In modern implementations of this protocol, infinity is 16. 
- **Split Horizon**: When sending routing updates, don't send routes back to our next-hop neighbor. For example, if after running our algo router $A$ has router $B$ as a next-hop to go to router $C$, we won't send $D_A(C)$ back to router $B$ when sending $A$'s distance vector. 
	- In our example, this would stop us from counting-to-infinity, but wouldn't encourage convergence: $y$'s and $z$'s distance tables would still contain inaccurate results. 
	- This is not a foolproof technique: if a link is down, two routers may independently decide their least-cost path is to point to each other, creating a **routing loop.**
- **Poison Reverse**:  A stronger version of split horizon where we send $D_A(C) = \infty$ back to $B$ when sending $A$'s distance vector.  
	- This helps us converge faster - still not super fast, but as fast as we can. 
	- In this example, $y$'s distance table entry for x would be set to 60, a closer representation to its actual value. Working through this, you can see that the vectors converge faster (try it! it's good practice). 
	-  This strategy prevents routing loops between pairs of routers, since they'll never choose to point to each other. However, it's not foolproof: some loops can persist involving 3+ routers, though we don't need to worry too much about this for the class. 
