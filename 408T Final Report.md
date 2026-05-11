
# Linux FTP

An nmap scan tells us there's a service active on port 44637. Connecting to it with `nc`, we see that we can send GET requests, and be served up directory listings, and even files, on the device. Unfortunately, we don't have root permissions, as we can't see root only files (ie, shadow). 

However, this allows us to investigate other services active on the device. Our nmap scan earlier told us about a vsFTP 3.2.5 service running on port 21. This is a little suspicious, since there is no 3.2.5 version of that software. We can browse our filesystem until we find a likely candidate for the binary running that service, then download that using port 44637 as follows `printf 'GET /usr/local/sbin/vsftpd HTTP/1.0\r\n\r\n' | nc <ip> 44637 > response.raw`. 

Searching for strings in this binary, we see `/bin/sh`. Very suspicious! Time for a reverse-engineering job. Opening up ghidra, we search for that string, and find it in this function: 
![[Pasted image 20260423214150.png]]
This is constructing a socket and invoking a shell - this seems like our device is hosting a shell on a port. We can find the port by looking at sa_data\[0-1]: this is 0x61 0x21, swapping for endianness (I'm assuming a htons call here) would give us the port 26401. 

But what triggers this? Finding references to this function points us to this, which seems to trigger when our last 2 input characters are :P
![[Pasted image 20260423214353.png]]
We'll test this out! 
![[Pasted image 20260423214523.png]]
Would you look at that!! A root shell. That's a VULN (1/5)

Ok, moving on. Using this rootshell, we find that the service running on port 5555 as ROOT is musicd. Grabbing that binary using a similar process above, we notice the string `/home/student/Downloads/musicdaemon-0.0.3/src` in it.  Looking this version up, we see it has an arbitrary file read vulnerability, allowing us to view any system file over the network without any authentication, which we can demonstrate below: 

![[Pasted image 20260423214738.png]]
That's a VULN (2/5)

# Compiler

We see an http server running on port 80: curling it tells us there's a file somewhere nearby that might help us inside. We can curl `/shadow`, and find the system's `/etc/shadow` file! Great!

We can crack this and get a list of passwords for most of the usernames - that's a VULN (3/5). 

Great, now we can ssh into the system. When we do, we're periodically met with the banner `Alert to all users: running WIP defragging script at /etc/degraggulator.sh.`. Since our crontab is empty, this means someone else is running it. Since root owns this file, it's a good chance this is a root-level cronjob. We have write permissions to it, so we can append `cp /bin/bash /tmp/bash; chmod u+s /tmp/bash` to give us a bash binary with the setuid bit set. When this script runs as part of the root's cronjob, that binary is created, and we can get an effectiveuid=root shell using `/tmp/bash -p`: 
![[Pasted image 20260423215423.png]]

That's a VULN (4/5). 

Ok, great! We're on a roll. Running `uname -r` tells us we're running on a kernel old enough to be vulnerable to the DirtyCow vulnerability. Using my freshly-stolen creds, I transferred over my dirtycow homework to this machine, ran it, and got a root shell. ![[Pasted image 20260423215620.png]]
Note that before doing this, I could use the `/tmp/bash` binary to set `dirty_writeback` to 0 to avoid a kernel panic. In any case, that's a VULN (5/5). 
`