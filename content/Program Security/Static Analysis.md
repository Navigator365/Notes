We'll start by talking about two broad categories of static analysis: **control flow (CFA)** and **data flow (DFA)**. First, a quick note on terminology: static analysis usually takes one of two forms, **offline** (prior to execution) and **online** (concurrent with execution). 

> [!Note] Online vs dynamic
> Though they both have *something* to do with program execution, we tend to describe dynamic analysis as getting from actually executing a program, and then analyzing that data after the fact. Online analysis is more in-line: the analysis is simultaneous with execution. 


# Control Flow

Control flow analysis focuses on how control flow changes are made, and is essential to computations for predicate conditions (ex jne x, y). We can enforce this with **Control Flow Integrity** (CFI), which is an online analysis that ensures that the program follows the control flow patterns we'd expect. This can help prevent against ROP attacks, which are all about disrupting control flow to build out new code to execute. 

We can enforce CFI by pregenerating a call graph, or a graph of all functions and their callers. Then, we can say a callee should only ever return to one of its callers. Of course, this doesn't really work for langauges that allow for function pointers to execute arbitrary functions (as then the allowed scope of functions expands to everything the fp could possibly be, including some functions that shouldn't be called in that context). We also need to keep track of our current context: if we call a function, we should return to the next line after that call, not another, later place that function is called (to prevent attempts to work around CFI).

CFI needs to be implemented through a program's assembly itself, as any other screening attempt (say, inspecting LLVM's IR), runs into a TOCTOU vulnerability. A program can have CFI when compiled, but data corruption could still happen during executions. Scans don't stop that: only programmed assembly-level checks that be generated statically but occur during execution. 

As a fun aside, when implementing CFI it's better to jmp {register w address} to return to a caller rather than ret'ing. Even if we have a perfectly secure program, a context switch to an insecure program could give an attacker ways to corrupt our process's memory. It's very difficult to corrupt a register file, but far easier to corrupt the stack. Since ret relies on the stack, we'll prefer jmping. 

# Data Flow

Data flow analysis focuses on how data is propagated, and is essential to almost any computation or assignment. For example, **Data space randomization** (DSR) is an online analysis that tries to prevent memory corruption through buffer overflow attacks, encrypting any data written to buffers, and copying and decrypting it when reading from buffers (we copy so that any exploitation relying on location fails). This mitigates the dangers from storing potentially malicious code in memory, because at least in its current state, the data can't be read as code. Of course, there are several mitigations to overcome this, but we won't worry too much about that for now. 

However, to enable DSR, we need to know *when* to decrypt/encrypt. This in turn requires us to identify and instrument every array read/write. We can accomplish this through **definition** and **use** chains. Formally, 
- Definitions are statements assigning a value to a variable
	-  A definition is **killed** if there is another assignment
	- A definition is **reachable** at a point p if data can flow from the definition to the point without encountering a kill. 
- Uses are references to the value of a variable

To compute these definitions, we'll get a Control Flow Graph (NOT Context-Free Grammar!!) of the program. Inside each block, we'll determine what new definitions have been generated, and what old definitions have been propagated. Then, we'll be able to characterize what definitions are alive inside the block, and what definitions will be passed to subsequent blocks. 

For loops, we'll recalculate all values inside/descendant from the loop until the loop is over. 

![[graph.png]]


> [!Note] SSA
> Many systems that perform data flow analysis simplify things with Static Single Assignment (SSA). Each variable is assigned only once; when that variable is reassigned, a new variable is created. This makes linking together use-define chains easier, as for each use there will only be one possible assignment (assuming a program that respects control flow)

 Let's use this to optimize DSR. Instead of encrypting all buffer reads, let's only do it when it's possible for the input to be untrusted, or entered from an outside source. 
 We can throw in some further optimizations by seeing if randomization is even necessary: we can use a constraint solver for a given byte size, and determine if any potentially malicious shellcode could possibly exist within that size. If so, we'll encode all data that is passed into that buffer. 


# Symbolic Execution

Despite the "execution" in the name, symbolic execution is still very much static. We're not running the actual code, we're running variables in a symbolic domain. We'll represent code as equations to solve, allowing us to formally reason about programs. 



