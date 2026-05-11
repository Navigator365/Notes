
# Part 1

Since we're using the 3.13 linux kernel, we can use [this](https://www.exploit-db.com/exploits/37292) overlayfs exploit which works on all kernels from 3.13 to 3.19. Executing it is simple: we just compile the script and run the resulting binary, and get a root shell:

![[Pasted image 20260326211454.png]]

This takes advantage of an overlayfs function's failure to check privileges. Overlayfs uses 2 distinct filesystems: an upperdir which is read/write, and a lowerdir which is read-only. As a result, when we require write-access to a file in lowerdir, that file is first copied to upperdir. The functions performing this copy (ovl_copy_up_\*) only check that the owner of the modified file has write permissions to upperdir, instead of checking if the actual writer has write permissions to upperdir. Then, we can put this modified file in a read-only directory (say, /etc) by swapping upperdir and lowerdir. We can use this to write to /etc/ld.so.preload, a root-owned system-wide configuration that forces the linker to load specific shared libraries before any others, overriding functions in the C standard library with code that spawns a shell. We can then execute any binary owned by root with the setuid bit set that uses any of these overriden functions: during the linking process, the binary will invoke our shell-spawning code, and since it's owned by root, the shell that spawns will be a root shell. 

# Part 2


DirtyCow allows us to write to any file we can read from. We'll use it to overwrite a root-owned setuid binary with shellcode. Before we overwrite the binary, we'll stash a copy of it in /tmp/mysuid, which we'll use to restore the binary later. 

While the setuid bit means the binary is run with the permissions of its owner, it does not set our uid to root by itself: that's why we set it ourselves first in our shellcode, just before invoking a shell. The call will succeed thanks to the suid bit, so the shell we invoke will be a root shell. Since our shellcode will overwrite starting from the beginning of the file, we'll have to structure this setuid(0)+/bin/bash shellcode inside an elf header so the binary can execute properly. Once we've successfully overwritten the binary with our payload, we execute the binary to get a root shell. 

When that shell exits, we'll use DirtyCow again to overwrite the setuid binary, replacing our shellcode with the original bytes of the binary, which we'll gain by reading bytes from 0 up to the length of our shellcode in /tmp/mysuid. After that overwrite succeed's, we'll have successfully restored the suid binary to its original state. 

Since our shellcode is relatively small, DirtyCow usually succeeds quite quickly. However, we still need read-write-access to the /tmp directory (or any directory on the system) so that we can restore the binary to its original state. We also need to be able to open a suid-binary on the system. For this attack, I chose pkexec and generated shellcode for an x86-64 system; this exact shellcode would not translate for different architectures. 

To use the exploit, invoke the build script, then invoke the generated body, use the root shell produced, then exit from it for the cleanup to take place. 