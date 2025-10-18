# Q1

The average entropy is 6.14583, and die does not think this program is packed. The entropy may not be uniform throughout the binary as some sections may have smaller values in their regions, while other sections may have more functions/data inside them. 

# Q2

BinaryNinja shows that most of the binary has a medium amount of entropy, which matches the die analysis that this program was not packed. 

# Q3

There are no strangely named sections; all the sections present are those I'd expect to be in a PE binary. 

# Q4

The average entropy is 7.74363, and the entropy graph is much different, showing a consistently higher value throughout the binary. 

# Q5

The majority (the middle, except for some stubs at the end) of the program shows high entropy for packed.exe, unlike the consistent low entropy of explorer.exe. 
# Q6 

There are less sections than normal in a Windows executable, and they have unusual names (UPX0/1/2). 

# Q7 

I see some strings that reference text printed to the terminal, like "Sorry too low." Malware authors might want to use a packer to change their program's layout to evade naive signature-based detection, and legitimate authors might use packers to prevent reverse-engineering of proprietary platforms and reduce program size. 

# Q8

I used upx -d packed.exe to unpack the binary. 

# Q9

The file that fails to open is MyPuts.dll. 

# Q10

LoadLibrary will load a DLL module, and GetProcAddress retrieves the address of a function in a loaded dll. 


# Q11

This library wants the function myPuts from the dll. 

# Q12

There is a BinaryNinja TTDRecordCPU.dll library that was not present in the previous execution. 

# Q13

The base address of the newly loaded module is 0x1fd731d0000. 

# Q14

The address of the myPuts function is 0x1fd731d1000

# Q15

I didn't see anything when running the program. 


# Q16

The hash calculation is here: 

![[Pasted image 20251015191627.png]]


# Q17 

Wininet.dll, Kernel32.dll, and Advapi32.dll are used to look up the functions. 

# Q18



# Q19

CreateProcessW, GetComputerNameW, etc

