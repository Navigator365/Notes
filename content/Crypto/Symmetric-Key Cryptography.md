
Symmetric key crypto is fast (most of the operations can be computed quickly) and involve 1 shared secret key. 


# Block Ciphers

We'll start off with the "basic" unit of symmetric-key cryptography. These are functions that take in a (message, secret key) pair to encrypt and output a ciphertext, and a (ciphertext, secret key) pair to decrypt and output the original plaintext. 

These block ciphers are constructed st that they provide some properties that can ensure encryption: 
- First and foremost, they're *deterministic*: E(k, m) will always be the same for the same (k, m) pair
- These ciphers are *pseudorandom*; each bit is equally likely to be set or not set in the binary representation of the encrypted output. This has two subproperties:
	- *Confusion*, that each bit of the ciphertext should depend on multiple parts of the key
	- *Diffusion*, that changing 1 bit in the message should lead to bigger changes in the ciphertext
- The ciphers are *one-way trapdoors*: decryption is easy with (k, c) but computationally infeasible without both. 
- The ciphers are fixed-size, taking in X bits as a message, and producing a ciphertext of length X. 


We can chain together block ciphers to produce arbitrary-length messages. However, this introduces a potential attack: 
- When decomposing our message into blocks, what if the same values are used for input across multiple blocks? 
	- As an example, say we end every sentence in a letter with "STOP" which takes up a whole block. 
- Since block ciphers are deterministic, they'll spit out the same encrypted output every time. 
- If attackers can associate encrypted output to other phenomena, they can gain insight into what's being sent without decrypting the ciphertexts themselves. 
	- Say a spy tells us every sentence ends in "STOP." Then, we can look for repeated ciphertexts of the same value, and infer they indicate separate sentences. 
	- Worse still, if an attacker is able to choose their own plaintext to encrypt, they could just encrypt "STOP" and look at the ciphertext, then substitute in that value for "STOP" in any messages they've captured. 

The solution is an *Initialization Vector (IV)*, which is either random or unique. Both are valid solutions, and each has their pros and cons. 
## Random

The icon of the random approach is Cipher Block Chaining (CBC). 
![[Pasted image 20251009200359.png]]

Here, we have a random IV we use as our initial input, and use the output of our encryption (which should be random) to encrypt our next block. 

To see how decryption works, let's do some XOR math. 
c\[0] = IV
c\[i\] = E(k, m\[i] ^ c\[i-1]) for all m starting with i = 1;

Then, decrypting any c\[i] is D(k, c\[i]) ^ c\[i-1] = m\[i] ^ c\[i-1] ^ c\[i - 1] = m\[i] (since xoring anything with itself is 0, and xoring 0 with anything is the other thing). 

Note that decrypting messages can be parallelized, assuming we've received all ciphertexts. Decrypting one ciphertext only depends on the previous ciphertext and key, both of which we already have. Encryption, however, relies on the output of encryption on the previous block, so it cannot be parallelized. 

## Unique

Alternately, we can use a unique IV each time we encrypt/decrypt, and bypass the need for chaining. The best example of this is Counter (CTR) mode. 
![[Pasted image 20251009201753.png]]

Our IV will be some value we'll randomly generate once, concatenated with a counter which we'll increment for each block of our message. 
As before, 
c\[i] = E(k, IV + i) ^ m\[i]
decryption = E(k, IV + i) ^ c\[i]  = m\[i]
But why does this work? 

This is similar to a One-Time pad, where we use a one-time key the same size of our message, and xor it with our plaintext. Doing so makes brute-forcing extremely difficult (impossible in the one-time pad's case). 

Since we're no longer chaining together encryption outputs, we can parallelize both encryption and decryption. We also only have to use 1 function (encryption). Sounds pretty good!  
# Authenticity via MAC

I want to send a message to somebody, and have them know if the message they received was the message I sent. We need symmetric crypto here: only they should know if only the messages we send are modified or not. Just like encryption and decryption, there are two stages at play here: 
- Signing: given a secret key and a message, produce a tag
- Verification: given a secret key, a message, and a tag, produce a yes/no answer for if this (message, tag) pair was sent by the person we share this secret key with

Here, our attackers are trying to read our messages and use them to construct an original (message, tag) pair that verifies as yes. Turns out, pseudorandom functions like our block ciphers can prevent these attacks. 

## ECBC

We'll base our signing function on CBC, but with a couple of changes. Our new MAC-producing strategy is called ECBC (encrypted CBC). 
To sign: 
- We take a message, and two keys k and k'
- We perform CBC with m, k, and no initialization vector. 
- We take the last ciphertext and produce a tag E(k', c\[i])

To verify: 
- We take m, k, k', and a tag, and rerun the signing algorithm and return yes if we get the tag, and no otherwise. 

Why only one ciphertext? Well, we don't care about encrypting the whole message, just a random, hard-to-forge block. Why use two keys? Well, otherwise attackers can do the following: 
Say the attacker asks for the tag for a message m. 
t = c\[n] = E(k, m\[n] ^ c\[n-1])
Then, the attacker generates a single-block message m' and asks for the tag for a message c\[n] ^ m'
t' = E(k, c\[n] ^ m') -> since we only have 1 block, it's just encryption
But notice t' = E(k, m' ^ E(k, m\[n] ^ c\[n-1])
That's the same tag as the message (m+m')!!!! Now the attacker can generate valid tags. 
By encrypting using another key, we have
t = E(k', c\[n]) = E(k', E(k, m\[n] ^ c\[n-1]))
t' = E(k', E(k, c\[n] ^ m')). Now, we have nothing to substitute, and no way to build out a valid tag from the tags and messages we've sent. 


## HMACS

An alternative approach involves hashing. Hashes have 2 important properties: 
- *Preimage resistant*: Given the hash H(m), it's hard to find m
- *Collision resistant*: Given H(m), it is hard to find other messages that produce the same hash. 
	- *Cryptographically strong* hash functions also ensure that for 2 distinct messages, the probability that their hashes have the same value for the same bit is 1/2. 

Not really sure about how they work...TODO later

# Hashing Best Practices

Hashing has its own concept of IVs - salts. These are random values concatenated with hash inputs, producing wildly different outputs for the same hash input. Good, now we won't be easily brute-forced by random tables. 

Whenever you're storing passwords, for example, store a password hashed with a salt, and store the salt separately from the hash. Also, spam hashing H(H(H(...))) to make hashing them inefficient to brute force while still easy to compute for correct inputs. 