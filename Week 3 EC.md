
We can simply append our script to the end of the existing script, allowing us to run malicious code without altering any of the original code. Since our iptables rules set up for NetfilterQueue only apply to forwarded packets, we can request the script from the server ourselves, grab its contents, and put it before our malicious script in the body of the http response we send. 

Here's the server's setup (ip 10.10.0.5):
![[Pasted image 20260221214112.png]]

Here's the run:
![[Pasted image 20260221214030.png]]

And here's what a client downloading from what it believes is a legitimate server experiences:
![[Pasted image 20260221214149.png]]
We've injected our own malicious script right at the end of the original script!