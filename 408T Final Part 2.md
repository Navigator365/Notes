
Team: Orange
# Big Picture

I created a backdoored version of openssh that allows logins with a secret password ("wehate417butlove408t'). As part of the implant's configuration, this binary replaces the normal openssh binary, appearing identical to normal ssh, except for allowing backdoored access without ssh-based logging.  

I inserted a `backdoor.c` file that computes a secret string (so that string won't be easily viewable, ie calling `strings` on the binary), and checks if a given string is equal to it. I invoke this function in `auth-passwd.c`. If the given password is equal to the secret string, the program accepts the login, even on the root user. In fact, I modified auth.c to ensure that logging in as the root user is always allowed, with `auth_root_allowed` always returning true. 

Finally, once we're logged in, I set an `extern`-declared variable in `auth-passwd.c` that's checked in `sshlogin.c`. If it's set, no actions, logins, or logouts will be logged by ssh, as this variable will cause them all to immediately return. 

I specifically built from the same ssh version that the machine normally uses, so users running port scans or checking service versions won't notice anything askew. 

To use: after installation, ssh to the machine as normal, and enter the password "wehate417butlove408t", which will grant a shell for whatever user you logged in with. 

# How I got this stupid thing installed

NOTE: You don't have to do this part! I'm just documenting how I got some of my files. Also, this process was annoying enough that I want to keep some documentation around in case I have to do something like this again. First, stand up a docker container: 
```
docker run -it -rm -v $(pwd)/deb:/deb ubuntu:16.04 bash
apt-get update && apt-get upgrade
```
Then, run 
```
apt-get install openssh-server
```
After that, go into /etc/apt/sources.list and uncomment the first deb-src link, and `apt-get update`. Then, run 
```
apt-get source openssh-server
apt-get install dpkg-dev
apt-get build-dep --print-uris openssh
```
This'll get our openssh source to edit, and a list of urls to debs of openssh's dependencies. But first, let's add our exploit! We'll make the following edits: 


# Installation Instructions

Login to the OB Linux FTP machine with an account with sudo privileges. 



Transfer the following submitted files to the machine: 
packages.tar.gz
cracked-ssh.tar.gz

Run install.sh, which will perform the following steps: 
- Extract both packages
- install the packages individually with `sudo dpkg -i <packagename>.deb`. 
- `cd openssh-7.2p2`. 
- Run the following command: 
`./configure --prefix=/usr --sysconfdir=/etc/ssh --with-pam`

Then, modify the Makefile as follows (sorry, I couldn't get the install script to easily do this part): 

In SSHOBJS: right after ssh.o, add backdoor.o
In SSHDOBJS (right below SSHOBJS): right after sshd.o, add backdoor.o. 

Then run `make` and `sudo make install`.  Edit `/lib/systemd/service/ssh.service` as follows: under the `[Service]` heading, change `Type=notify` to `Type=fork`. Also, in `/etc/ssh/sshd_config`, change the last line to `UsePam no`

Then, run `sudo systemctl daemon-reload`, then `sudo systemctl restart ssh; sudo systemctl restart sshd; sudo systemctl enable ssh`


# Intended Use

I'll use this as a backdoor to ensure we'll always have access to blue's network. 

# Testing
1. I installed this on RG Linux FTP. Here, I logged in as a root user using the secret password. The machine's systemctl logs don't report a root user login, showing we've effectively hidden our backdoor. ![[Pasted image 20260425200400.png]]
2. I've rebooted the machine, and the backdoor works. Since it's the only version of ssh running, it's highly stable (after all, it is openssh with some slight modifications)