# Lab 1

## Q1

apropos users
## Q2

argv is char** at address 0x7fffffffdb98, pointing to string /home/bdodge1/Downloads/tmp_hw1/xor_c_debug.bin
## Q3

I see other environment variables of my system displayed. 
## Q4

The start of `.text` is 0x401100. 
## Q5

The stack ranges from 0x7ffffffdd000 to 0x7ffffffff000
## Q6

This likely follows the standard calling convention, with rdi as the first argument and rsi as the second, then rdx as the third. 
## Q7 

The return value is stored in RAX at address 0x00000000004052a0. 
## Q9

These are heap-allocated structs. 
## Q10

fopen64

# Lab 2

## Q1 

strace would be better suited for statically compiled binaries, since there are no dynamic libraries for ltrace to capture. 
## Q2

strace gives more information, while ltrace is a little easier to understand (use of malloc instead of mmap). As for the 4: fopen corresponds to openat, malloc to mmap, fread to read, free to munmap, fwrite to write, and fclose to close. 

## Q3

Read takes in a file descriptor first, in this case reading from a user-allocated ELF file (since it's not 0, 1, or 2), and read 832 bytes from it. 

## Q4

The file /lib/x86_64-linux-gnu/libc.so.6 was the file read by the first read syscall, and file descriptors are not unique once the file has been closed, since that fd number can now be reused. 

## Q5

ltrace is much larger, including all std functions called. The `new` operator is used to allocate dynamic memory. 

## Q6

Since the cpp file loads more dynamic libraries (std and gcc), it has to perform more read syscalls to read/load those files. 

## Q7 

The basic structure and order of syscalls remains the same across both languages, but specific argument values are different (st_mode)

## Q8

dropbear requires sudo permissions to run; reading from /etc/dropbear/dropbear_ecdsa_host_key is privileged. 


## Challenge

Using ltrace lets us see a strcmp that contains a comparison of our user input to a string, so we'll take that and use it as the password, which we use to get a flag. The flag is produced by decoding the input password to produce HACS408eIsTheBestClass@UMD.  

Flag: ACES{HACS408eIsTheBestClass@UMD}