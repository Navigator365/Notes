Our network is a graph, with nodes as routers and edges as links. These edges are weighted with what I'll call costs. We want to compute the least-cost distance to each router for all routers in our network. 

We want this computation to be **dynamic**, **anycasted** and **resilient**: 
- Our algorithm should respond to link outages or changes in link weights
- Our algorithm should give each router a full view of the network topology. 
- Our algorithm should prevent errors or incorrect behavior in one router from cascading and affecting other routers. 
## Transmission

For each node to be capable of running Dijkstra's on its own, it needs a complete view of the network. We provide this through Link State Packets (LSP), which provide the costs of a node's neighbors, just like Distance Vector. Also like Distance Vector, we'll send these LSP's both periodically and in response to link cost changes. 

Each LSP has 4 components: 
1. The ID of the router that created it
2. The costs to each neighbor from the router that created it
3. A sequence number to distinguish it from older LSPs
4. A Time To Live (TTL) specifying how long its information should be preserved

Why those two extra components? Well, we need to pass this information along to all nodes in the graph, not just our neighbors. We call this practice *flooding*: we'll first send it to our neighbors, who will then send it to their neighbors. These extra components help ensure our information is up-to-date, and prevent LSPs from being endlessly send around the network. 

When a router receives an LSP, it performs the following process: 
- Do I have an LSP in storage with the same ID?
	- Yes! Ok, let's compare the sequence numbers of the stored and incoming LSPs. 
		- The incoming LSP has a greater sequence number. That means it's newer! I'll forward it to all my neighbors, except for the one that sent the LSP to me. 
		- The incoming LSP has a smaller sequence number. This is an older message; I won't send it
	- No! I'll store it and forward it to all my neighbors, except for the one that sent the LSP to me
![[Pasted image 20250921211412.png]]

This process helps flooding end quickly, minimizing the overhead required for Link State. To save on storage and prevent old data from persisting, the TTL of an LSP is decremented every flood. When a TTL becomes 0, that router doesn't forward the LSP. 

## Algorithm

Let's say router $A$ has LSPs for all other nodes in the graph. We'll now calculate least-cost paths from $A$ to all other nodes. We'll do Dijkstra's while taking advantage of the format of LSPs. 

$A$ stores 2 lists, $T$ and $C = \{A\}$. The elements of each are tuples of the form (Node, Cost, Next-hop), where next-hop is the next node $A$ should route to along the shortest path to the node. Then, we'll repeat the following process:

 If it's our first iteration or $T \neq \emptyset$ : 
- Take the most-recently added element of $C$ and call it $N$. Grab $N$'s LSP. 
- For each neighbor $n$ of $N$, its cost $cost_n$ = $cost(A, N) + cost(N, n)$. 
	- If $n \notin C$ AND $n \notin T$, append $(n, cost_n, NextHop)$ where NextHop is $N's$ next-hop. 
	- If $n \in C$ and $cost_n$ is less than $n$'s cost in $C$ currently, replace its entry with $(n, cost_n, NextHop)$. 
- Pick the smallest entry in $T$ and move it to $C$. 

This isn't so dissimilar to normal Dijkstra's. The main distinction comes in how we're storing neighbors. We don't need to reconstruct the path: all we really care about is who to send to next. Therefore, we only keep track of NextHops, and propagate them along distance nodes, so we don't store unnecessary information. 
## Advantages over Distance Vector 

In general, Link State routing has more predictable numbers of messages sent and speed of convergence, and its upper bounds on both are far less than those of Distance Vector routing. 

It's also more robust against router malfunctions: LSPs only contain information about a router's neighbors, and all routers are talking to each other. If one bad LSP is sent out, it won't disrupt every router's pathfinding, and will likely be corrected soon by other routers that share some of the same neighbors. In contrast, malfunctioning routers in Distance Vector can send out vectors with incorrect distances for all destinations. Not only is the scale of the error bigger, but it's potentially the only source of information about some paths for some of its neighbors, amplifying the error. In short, since Link State sends less data and talks to more routers than Distance Vector, it's harder for errors to be defused throughout the entire network. 