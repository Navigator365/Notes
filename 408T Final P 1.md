
# OB Linux FTP

We'll look at our diagram and see our ip is 10.4.0.27. First, we'll run `nmap 10.4.0.27 -sV -T4`

These are our results: 
![[Pasted image 20260421185829.png]]

While all of these are worth investigating, I'll start with port 5555, since it seems the most wild. Connecting to this port with `nc`, we see it sends us "Hello" as raw bytes over a TCP stream. No higher-level protocol or anything like that. Weird. I tried sending back some input, but no luck there either. Unfortunate. 

vsFTPd is claiming it's version 3.2.5, but that doesn't exist...wondering if it's vulnerable to this backdoor https://www.rapid7.com/db/modules/exploit/unix/ftp/vsftpd_234_backdoor/
- tried that backdoor and didn't get anythign
- passwordless login enabled, BUT seems empty, /
- ok it probably works, but for a different special character



Port 44637 gives us a directory listing...
- I don't think we have root access, but can still read files
- unfortunately i crashed it
- and now there's an http port???
- looks like it periodically restarts
- DONT PRESS CtrlC
- Seems to be trying to connect to http:80 --- i wonder why
- seems to periodically freak out and send back a RST to me
- use nc , then manually type out GET /path HTTP/1.0
- then
```
printf 'GET /usr/local/sbin/vsftpd HTTP/1.0\r\n\r\n' | nc <ip> 44637 > response.raw
grep -abo $'\r\n\r\n' response.raw | head -1
dd if=response.raw of=binary.out bs=1 skip=<offset+4>
```
Boom, we have our binary.
- port is 0x61 0x21, swapping endianness and converting gives us 26401

FTP
searching for strings, we see /bin/sh, we follow the call chain and discover it's triggered whenever we input :P
Just append our username with a :P on login, and we get a root shell spawned on that port 
https://westoahu.hawaii.edu/cyber/forensics-weekly-executive-summmaries/8424-2/
VULN


So we can just do that!


THE 5555 mystery - /usr/local/bin/musicd
- from our root shell, we see that this is running musicd, which at the very least seems to allow for some paths
- https://github.com/foxbenjaminfox/musicd#playlist-format
- /home/student/Downloads/musicdaemon-0.0.3/src in binary strings, this is interesting
	- file isn't present, but it can serve as some identification
- lsof tells us it's running as root, good sign
- https://marc.info/?l=bugtraq&m=109329098806595&w=2
- This works quite nicely,
- ```
  LOAD /etc/shadow
  SHOWLIST
  ```



- explore 5555 mystery, starting by viewing source code 
VULN

# OG Compiler

Http server - apache 2.4.7
- curl is the goat
-  HTML SAYING A COPY OF THE SHADOW FILE IS THERE!!!!!!!
- just ip/shadow - BOOM SHADOW
- VULNERABILITY
- see the type's \$6\$, so we can crack it using sha-512



- root level cronjob lpe: 
	- ```
	  in script:
	  cp /bin/bash /tmp/rootbash
	  chmod u+s /tmp/rootbash
	  
	  then wait for it to run, and call
	  /tmp/rootbash -p
	  gives root shell
	  ```
- VULN

- dirtycow - server running at port 33333
	- running as a non-root user though, so still need privesc
	- Is there an advantage to writing something that can give us a root shell over the network, as opposed to LPE?
	- If there are points for bringing the compiler down, we could overwrite the compiler itself with some nonsense that would break it, or maybe python
	- echo 0 > /proc/sys/vm/dirty_writeback_centisecs <- need to do this first, ideally with the root level cron job (or resulting root shell, doesn't matter)
	- ![[Pasted image 20260422105447.png|532]]
	- note that we could use this to trigger a panic, 
- VULN


# Implant idea

poisoin binary to accept root credentials if given a
- https://launchpad.net/ubuntu/+source/openssh/1:7.2p2-4ubuntu2.4


have ta log into a windows machine (not the domain controller) as the DOMAIN CONTROLLER then run mimikatz 
- on one of the windows machine, run mimikatz (vuln as eternalblue)
- get memory dumps, you're already priviled
- figure out how to get persistence on other computers
if not -- disconnect windows computers from domain