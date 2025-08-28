---
tags:
  - htb
---
# Handshake
So we all know how HTTP works. You type something in, it gets resolved via DNS, and you get served with whatever beautiful content you want. 

But how do we make it SECURE? HTTPS (s is for SECURE).  We have to do a little handshake, and then all of our messages will be encrypted in transmission. 

> [!Warning] DNS Security
> While HTTPS can encrypt your data, it DOESN'T handle DNS resolution. So, don't use an unencrypted DNS sever if you don't want anybody seeing what URLs you're going to. Or you could just be boring and use a VPN. But that's no fun.

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

## Digital Certificates

Every digital certificate contains a BUNCH of things: the names of the subject and issuer, a public key associated with the subject, and CRL/OCSP info (more on that later). It also contains a digital signature signed by the CA of that certificate. That signature is formed from encoding the certificate itself. To verify a certificate is legit, a so-called verifier needs to obtain the CA's public key. The verifier expects to decrypt the digital signature and get the encoded value of the certificate; if it doesn't, that means there's no way to prove the CA actually issued the certificate.

Not all CA's are implicitly trusted by web browsers, so clients must build a chain of trust to a root CA the browser implicitly trusts. That means getting a certificate for the CA, grabbing its public key to verify the previous certificate,  then reaching out to the issuer of the CA's certificate to verify the CA itself, and so on and so forth until the browser gets a self-signed certificate issued by a root CA it implicitly trusts. 

## OCSP and OCSP Stapling

Ok, but how do we know the certificate is still valid? Maybe it was revoked. The canonical solution to this was to maintain Certification Revocation Lists (CRLs) that a client would download periodically, (along with signatures confirming their validity) but having to check that list every time you set up a TLS handshake is really slow. 

So along came OCSP (Online Certificate Status Protocol). Clients could now contact CAs and get a signed statement on the certificate's status. BUT this slowed down clients and put pressure on CA servers to handle a large amount of requests...a better solution was needed. 

Introducing OCSP stapling! Now, the certificate holder checks the status of its own certificate, and "staples" the CA's signed response to TLS handshake. That way, clients only have to verify the CA's response (which requires getting the CA's public key, which they already have to do anyway to verify the certificate). This also helps keep client browsing habits more private, since nobody needs to ask CA's for the validity of specific site signatures anymore. 
# Requests

Each HTTP request provides a method for accessing a resource. Though there are others, the two big ones are GET and POST. 

**GET** requests resources, and can pass data via `?=`. You can also pass authorization into it and use it as a (rather stupid) login system with the `Authorization` tag. We can specify tags with `-H` in curl. 

**POST** sends data (ex, uploading a file to a server). Instead of putting parameters in the head like GET, POST puts them in the body.  That way, less data has to be encoded as letters, and more data can be sent without clogging up the URL.  We can send POST requests via `-X` in curl, data with `-d`, and cookies with `-b`. 

## CRUD

CRUD APIs are VERY common, and have 4 operations: Create (POST reqs), Read (GET), Update (PUT), and Delete (DELETE). 

# Responses

Here's a nice little table of response codes. 

| Classes | Meanings                                                                                                                          |
| ------- | --------------------------------------------------------------------------------------------------------------------------------- |
| 1xx     | Provides information and does not affect the processing of the request                                                            |
| 2xx     | Returned when a request succeeds                                                                                                  |
| 3xx     | Returned when the server redirects the client                                                                                     |
| 4xx     | Signifies improper requests **from the client**. For example, requesting a resource that doesn't exist or requesting a bad format |
| 5xx     | Returned when there is some problem **with the HTTP server** itself                                                               |


