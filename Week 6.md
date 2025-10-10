# Q1

VirusTotal detects the file as malicious, indicating that is a trojan that's part of the bodegun/convagent/nekark families. 

# Q2

There are some strings of an xml file, and others referencing either dynamically loaded dlls or functions in those dlls, including a reference to a directory and a web domain.  Finally, there are some strings indicating that this is a WannaCry 3.0 virus. 

# Q3

There are references to connecting to the Internet via libraries like "InternetOpen*" libraries. There are also some libraries that could be used for fingerprinting, like GetOEMCP, and some file reading/writing imports.

# Q4

Some sigs: 
Haha! You've been infected by WannaCry 3.0! Well... the beta version at least.
http://evildomainsdfheifoweifjksdflijsd.com


# Q5

Fakenet redirects to a fake page, indicating it intercepts and redirects network traffic while simulating legitimate network services. 


# Q6

The procmon output indicates the malware is loading an image, creating files and loading more images. 

# Q7

The malware makes some network connections, one to the hostname.aces.umd.edu:domain network (no DNS resolution at first), and another to to 192.0.2.123:http. 

![[Pasted image 20251008183831.png]]

# Q8

Yes, now the malware accesses (opens/reads/writes) my newly-created test.txt file. Also, the dns does not resolve at all this time, and no references to the 192.0.2.123 ip. 

# Q9

Regshot shows a SOFTWARE registry entry that belongs to the wannacry virus was set to the same value, indicating this is a representation of its key. 

![[Pasted image 20251008190149.png]]