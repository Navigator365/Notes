
TODO: How does TLS respond to MITM attacks? MITM during the handshake?
# Handshake
So we all know how HTTP works. You type something in, it gets resolved via DNS, and you get served with whatever beautiful content you want. 

But how do we make it SECURE? HTTPS (s is for SECURE).  We have to do a little handshake, and then all of our messages will be encrypted in transmission. 

> [!Warning] DNS Security
> While HTTPS can encrypt your data, it DOESN'T handle DNS resolution. So, don't use an unencrypted DNS sever if you don't want anybody seeing what URLs you're going to. Or you could just be boring and use a VPN. But that's no fun.

## TODO: Move this all into its own section
Also: I just realized public key encrypts with other person's public and decrypts with their private, while ds encrypts with their private and decrypts with other person's public. I think that's worth talking abt. 

![[Pasted image 20250825160826.png]]
Now about this handshake: it's in TLS (formerly SSL, now used interchangeably). Through it, the client and server use asymmetric encryption to encrypt their communications using symmetric session keys. Let's cover that handshake in a little more detail: 
![[Pasted image 20250825162614.png]]

Some notes: 
- The above diagram is not the only way to do TLS. Many possible encryption methods can be used. All TLS does is describe how the general process should look. 
- The SSL certificate the server has contains not only the CA's digital signature, but also the server's public key. 
- This enables authentication of that certificate when the server signs parameters with its private key - now the client can try to decrypt. If successful: we're talking to the right person. 
- Keyless is cool because you don't have to store your private keys on a cloud server - that's one less security risk ig. 
- This particular protocol enables **forward secrecy,** where compromise of a single session key doesn't jeopardize data in any other session. In other words, the attacker could only decrypt data from the sessions they have keys for. Forward secrecy demands 2 things: 
	- A unique session key for every session
	- A session key generated separately from the server's private key (otherwise, a compromise of one session key could expose the private key which would expose all session keys)
	- This is why RSA isn't used for TLS anymore, since the server encrypts its premaster secret with its private key and send it to the client for key generation. 