# General Techniques

A couple common disassembly methods: 
- **Linear Sweep**: Treat every byte as an instruction and try to decode it. This fails miserably when data is mixed with code (which often happens).
- **Superset Disassembly**: Try every combination of bytes that produces a valid instruction, then try to piece them all together in a way that makes sense. No false negatives this way, but prone to false positives. 
- **Probabilistic Disassembly**: Similar to superset, but just assign probabilities and choose the most likely one. While it's unlikely you'll  have false negatives this way, you'll also have some false positives.
	- We build these probabilities using a couple of hints!
	- If a series of possible instructions have control flow convergence (jumping to a common label), it's more likely they're legit. 
	- If control flows cross (a jump to A and then a jump to a lower label B)
	- [[Static Analysis#Data Flow|Def-use relationships]]
	- Once we've defined our hints for probabilities, we can establish some basic logic rules: ![[Pasted image 20250826204746.png]]
![[Pasted image 20250826203630.png]]

