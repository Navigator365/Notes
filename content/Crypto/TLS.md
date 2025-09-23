
So we all know how HTTP works. You type something in, it gets resolved via DNS, and you get served with whatever beautiful content you want. 

But how do we make it SECURE? HTTPS (s is for SECURE).  We have to do a little handshake, and then all of our messages will be encrypted in transmission afterwards. 

> [!Warning] DNS Security
> While HTTPS can encrypt your data, it DOESN'T handle DNS resolution. So, don't use an unencrypted DNS sever if you don't want anybody seeing what URLs you're going to. Or you could just be boring and use a VPN. But that's no fun.


Now about this handshake: it's in TLS (formerly SSL, now used interchangeably). Through it, the client and server use asymmetric encryption to generate a symmetric session key that will be used to encrypt the rest of the conversation. 

I'll focus on [TLS 1.3](https://tls13.xargs.org/), since it's faster than 1.2 and removes support for more insecure methods of encryption. There's definitely more to their differences than that, but let's cover the basics first. 

1. To start, the client and server will both generate their own ephemeral public/private key pairs. 
2. The client and the server will both send hellos, containing random data, cipher suites to use, and their public key. The server will decide which suite/protocol to use; the client just sends a list of ones it can do. 
3. The client and the server will both derive the same handshake secret using their private key, the other's public key, and a hash of both hellos. This shared secret is used to generate the server handshake key, which will be used to encrypt the rest of the conversation. 
4. The server sends its certificate (containing a public key associated with the cert along with a CA's signature), and a certificate verifier, proving that the server has the private key associated with the certificate's public key. This verifier is a hash of previous handshake messages signed by the certificate private key. Finally, the server sends a message verifier, a hash of all previous messages (including the certificate and certificate verifier) signed by the handshake secret, confirming nothing in the entire message was tampered with. 
5. The client confirms the CA's signature, the certificate verifier, and the message verifier, showing that the certificate is valid and the server actually owns it. 
6. Then, both client and server generate a session key from the handshake secret. This key will be used to encrypt/decrypt all future messages. 

The certificate is essential to the whole process. Since the handshake setup uses asymmetric encryption, MITM attacks can't just passively listen to communications and decrypt them. They need to actively be involved: setting up TLS sessions of their own between client->MITM and MITM->Server. This means that the MITM must have its own certificate! 

Well, what if it just uses the server certificate? While it can obtain the server certificate, and can't replicate the certificate verify, since that's formed from the certificate's private key signing all previous messages, which include random data from that particular handshake. This means it's impossible for the MITM to reuse that certificate verify on the client, since the contents of what the private key will be signing will be different. Since the MITM doesn't have the server's private key, it can't recreate that message, and therefore can't use the server's certificate. 

What if the MITM gets its own certificate? To be trusted, that certificate needs to be signed by a CA. But with smart CA's, that will never happen, and attackers can't MITM. 

### Forward Secrecy

So about those encryption methods TLS 1.3 removed - what made them bad? Many lacked forward secrecy, where compromise of a single session key doesn't jeopardize data in any other session. Forward secrecy demands 2 things: 
- A unique session key for every session 
- A session key generated separately from a non-ephemeral private key (otherwise, a compromise of one session key could expose the private key which would expose all session keys)
By only using ephemeral private keys, TLS 1.3 makes forward secrecy a lot more achievable. 


