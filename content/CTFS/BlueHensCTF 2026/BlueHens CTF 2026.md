
# Forensics
## Name Calling

![[Pasted image 20260417133118.png]]

We're given a pcap with the following objects: 
![[Pasted image 20260417133216.png]]
The zip is encrypted, and I couldn't extract any useful metadata from any of the images, so this seems like a dead end. 

# Web

# Magic Link 1

![[Pasted image 20260417134450.png]]

Again, I was pretty terrible here. We can see via the source that we're making a POST req to /login, which isn't accessible to us through normal GETs. Simulating the post ourselves, we get a json with a uuid field returned...
![[Pasted image 20260417134608.png]]
...but I couldn't use this as a cookie when trying to make a get request to the site, so that didn't work. Maybe the solution is gobuster...

NO! It was right in front of me the whole time. Check robots.txt, and we'll see a reference to a .env file, and checking that will find our flag. 
![[Pasted image 20260417135206.png]]
## Magic Link 2

Following the above screenshot, we travel to the INBOX_URL and find another flag. 

## Magic Link 3

Reaching that page, we can see we login through `login\sometoken`. We can generate a token for the admin email using a post request, then login using that token to get the flag. 

# pwn

## Temporal

We can use the stat command to find that the flag is at /flag, along with an inode associated with it. We can also leak (a) libc base address, for a particular note. Not sure how we can chain this together into an exploit, am looking forward to seeing the actual solution. 

