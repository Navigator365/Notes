
TODO
- smth abt how they can be installed at the os level to build up that layer of trust
	- Certificate transparency logs to ensure good behavior
- smth abt how forging a cert to have your own public key destroys the CA sig


## Digital Certificates

Every digital certificate contains a BUNCH of things: the names of the subject and issuer, a public key associated with the subject, and CRL/OCSP info (more on that later). It also contains a digital signature signed by the CA of that certificate. That signature is formed from encoding the certificate itself. To verify a certificate is legit, a so-called verifier needs to obtain the CA's public key. The verifier expects to decrypt the digital signature and get the encoded value of the certificate; if it doesn't, that means there's no way to prove the CA actually issued the certificate.

Not all CA's are implicitly trusted by web browsers, so clients must build a chain of trust to a root CA the browser implicitly trusts. That means getting a certificate for the CA, grabbing its public key to verify the previous certificate,  then reaching out to the issuer of the CA's certificate to verify the CA itself, and so on and so forth until the browser gets a self-signed certificate issued by a root CA it implicitly trusts. 

## OCSP and OCSP Stapling

Ok, but how do we know the certificate is still valid? Maybe it was revoked. The canonical solution to this was to maintain Certification Revocation Lists (CRLs) that a client would download periodically, (along with signatures confirming their validity) but having to check that list every time you set up a TLS handshake is really slow. 

So along came OCSP (Online Certificate Status Protocol). Clients could now contact CAs and get a signed statement on the certificate's status. BUT this slowed down clients and put pressure on CA servers to handle a large amount of requests...a better solution was needed. 

Introducing OCSP stapling! Now, the certificate holder checks the status of its own certificate, and "staples" the CA's signed response to TLS handshake. That way, clients only have to verify the CA's response (which requires getting the CA's public key, which they already have to do anyway to verify the certificate). This also helps keep client browsing habits more private, since nobody needs to ask CA's for the validity of specific site signatures anymore. 
