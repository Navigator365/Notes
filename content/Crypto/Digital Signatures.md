We want to create something that we know was signed by exactly one person, and cam be decrypted by arbitrarily many people. Therefore, we want to sign with our private key: if we signed with our public key, we don't know if we actually signed it, or somebody else who has our public key. 

So, we sign with our private key, and verify (decrypt) with the signee's public key. 