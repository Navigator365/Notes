We'll start by talking about two broad categories of static analysis: **control flow (CFA)** and **data flow (DFA)**. First, a quick note on terminology: static analysis usually takes one of two forms, **offline** (prior to execution) and **online** (concurrent with execution). 

> [!Note] Online vs dynamic
> Though they both have *something* to do with program execution, we tend to describe dynamic analysis as getting from actually executing a program, and then analyzing that data after the fact. Online analysis is more in-line: the analysis is simultaneous with execution. 


# Control Flow

Control flow analysis focuses on how control flow changes are made, and is essential to computations for predicate conditions (ex jne x, y). We can enforce this with **Control Flow Integrity** (CFI), which is an online analysis that ensures that the program follows the control flow patterns we'd expect. This can help prevent against ROP attacks, which are all about disrupting control flow to build out new code to execute. 

# Data Flow

Data flow analysis focuses on how data is propagated, and is essential to almost any computation or assignment. For example, **Data space randomization** (DSR) is an online analysis that tries to prevent memory corruption through buffer overflow attacks, encrypting any data written to buffers, and copying and decrypting it when reading from buffers (we copy so that any exploitation relying on location fails). This mitigates the dangers from storing potentially malicious code in memory, because at least in its current state, the data can't be read as code. Of course, there are several mitigations to overcome this, but we won't worry too much about that for now. 

However, to enable DSR, we need to know *when* to decrypt/encrypt. This in turn requires us to identify and instrument every array read/write. We can accomplish this through **definition** and **use** chains. Formally, 
- Definitions are statements assigning a value to a variable
	-  A definition is **killed** if there is another assignment
	- A definition is **reachable** at a point p if data can flow from the definition to the point without encountering a kill. 
- Uses are references to the value of a variable

To compute these definitions, we'll get a Control Flow Graph (NOT Context-Free Grammar!!) of the program. Inside each block, we'll determine what new definitions have been generated, and what old definitions have been propagated. Then, we'll be able to characterize what definitions are alive inside the block, and what definitions will be passed to subsequent blocks. 
![[Pasted image 20250827113619.png]]
For loops, we'll recalculate all values inside/descendant from the loop until the loop is over. 

> [!Note] SSA
> Many systems that perform data flow analysis simplify things with Static Single Assignment (SSA). Each variable is assigned only once; when that variable is reassigned, a new variable is created. This makes linking together use-define chains easier, as for each use there will only be one possible assignment (assuming a program that respects control flow)

 Let's use this to optimize DSR. Instead of encrypting all buffer reads, let's only do it when it's possible for the input to be untrusted, or entered from an outside source. 
 We can throw in some further optimizations by seeing if randomization is even necessary: we can use a constraint solver for a given byte size, and determine if any potentially malicious shellcode could possibly exist within that size. 