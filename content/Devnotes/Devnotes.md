Given as command line arguments:
- Path to shared object
- Function to call shared object
- Arguments for function

Safely sandbox calls to dlopen->dlysm->function execution. 

Since all of the tools require some compilation, we can...
- Write a program that will initialize and compile a sandbox
- Execute some code on it
- Communicate 
- Terminate

Advantages over the dumb fork->execute->rlimit way:
- Already scripted
- A LOT MORE FEATURES
	- Like if we ever cared about trying to profile our binaries this would nuke the other way
- Potentially slower since we have to compile a miniature program just to act as the sandbox


The real limitation is IPC BECAUSE sandbox libraries (sometimes) provide nicer IPC structures than me programming on my own
- And safer too

Is that worth the cost of compiling?

How does Google's sandbox api send back resources?
- Non-primitive types

FOR NOW, do not worry about efficiency reduction on repeated dlopen->dlsym->function call paths


Two options
- "Do-it-yourself" with fork
	- Later on, see if I can work in [pledge()](https://justine.lol/pledge/) (see fork part for the cool stuff)
- Compile a mini program thats just going to handle our dlopen sequence, and use existing libraries (like google Sandbox API) for that

WAIT...CODEJAIL???
https://www.comp.nus.edu.sg/~liangzk/papers/esorics12.pdf
> "If dynamic loading with dlopen() is used, our wrapper library will be opened when
a relative path is used. If an absolute path is used, which is uncommon, the original
library will be opened, calling its API has an exception as it is not executable in the
main context."


I feel like Codejail's insistence on compile 


https://dl.acm.org/doi/pdf/10.1145/3618305.3623604
^ Seems to be the blueprint




