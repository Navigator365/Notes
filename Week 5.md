# MD5 

I began by finding a large password database: [Seclists Pwdb](https://github.com/danielmiessler/SecLists/blob/master/Passwords/Common-Credentials/Pwdb_top-10000000.txt) worked well. 

We can write these to a file and run `hashcat -m 0 -a 0 md5.txt Pwdb_top-10000000.txt

1.  a7cb53633cd94b14fe96b292600e76a8:cutie12 
2. e0dfe2c1c5938a83a12eb0eeacf2725c:kurtcobain               
3.  f25a2fc72690b780b2a14e140ef6a9e0:iloveyou                 
4. 0d107d09f5bbe40cade3de5c71e9e9b7:letmein                  

# SHA1

We write these to a file and run `hashcat -a 3 -m 100 sha1.txt ?u?d?d?d?d?s?l?l?l`
1. 7f32469e06ed761e74956d5fad871eb15aea0861:I2018!son   
2. 756ccabcab3468b15721d45d11c573d9a61fab48:R1596&hjf        
3. 4a82c647f206a2fc576a6b6e30aabca8542ecbc9:G1407^erg      
4. 6208ed7e69f459dfef60a816b282720a7752f7ba:K3063\<evi        
5. db5942fce82dd937b2b4f714c025472acb58b3a3:U1970@nix        


# LM

We will pass these to a file and run `hashcat -a 0 -m 3000 lm.txt rockyou.txt`. Note that since LM splits hashes in two, we'll have to piece together our passwords by combining the results of both parts of our hashes. This gives us the following: 
1. SUPERMA+N
2. COMPUTE+R
3. DRAGON + empty

# NTLM

Run `hashcat -a 0 -m 1000 ntlm.txt Pwdb_top-10000000.txt`. 
1. 4c090b2a4a9a78b43510ceec3a60f90b:babygirl                 
2. b7e0ea9fbffcf6dd83086e905089effd:tigger                   
3. ed009a5dc9ad1848d4fc077205315aed:lovely                   

# SHA512

We first run `cewl https://digi.ninja -w digiwordlist.txt`, then `hashcat -a 0 -m 1700  md5rounds.txt digiwordlist.txt`

1. 84d38f97acd047a134a7c42df5a6156fefad58f440118e59e33f6884a3a3518c43d596995587950670191687b439082a8999eeea0eddfdfc155a9b9c89a7d8a7:digininja
2. c2c6e691755730379ff2ed4b7da8824bb8ce45d2ce71b521156dd45daf921ff1c6dc501a5abcabef8a477b4ca526624fd76550e3642cb8c1eb259118f943dc5c:Addonics
3. f8375b735c5b6d222083092bffcc33bc5d78a635446a4a046b40e9e0cfdf8fa2d09be7d2c77a8ec8efbba1e4b6763d80865eb6795ae9d663f9f6218dfad04ab9:ModSecurity
4. ab8cc51b74f7532cca958d09406e322251ec253cf2ed051bcc92bcf1c37d6e0b89aad342fcc353b0fa65cc3c945655391a3b3667ab22ce4e4e7140cdf57a4d92:Interceptor
5. 1a5f5111831f0acc3e940c602299b9ffcb15a21c045466b7280a08bef028b2bd01f84d373a7001f1c58d6eb5950ef1cdcab00888712cec45caa0064bd89e595b:SocketToMe

# Rules

I used a combination of open-source tools for this. [RSMangler](https://www.kali.org/tools/rsmangler/#rsmangler) can generate permutations of a given word, while [Single-Seed Wordlist Generator](https://github.com/StayPirate/Single-Seed-Wordlist-Generator?tab=readme-ov-file#Rule-files) is a more exhaustive (and expensive) version, with support for l33t-speak style substitutions as well. Using both tools (`./rsmangler.rb -m 5 -x 15 --file ../manchester.txt --output mangled.txt; echo manchester | ../../hashcat.bin -r 03-l33t/l33t_micro_1234.rule  | sort -u > example_wordlist.txt)` gives us wordlists to pass to hashcat `../hashcat.bin -a 0 -m 1400 sha256manr.txt example_wordlist.txt`, giving us the hashes

1. 4254581aa1717464c53d49cfc81428939c21774a098295d783b44828312598f3:manchestering (found with rsmangler)
2. c7c938b91e7e00473ee1bfa98544bbad10637d0a6a6e27ced560e0d221e999d2:manchestered (found with rsmangler)
3. 5efcaa1b8e139877c6e7c0ef1e6b7151ecd1d220e4d174f3dfca4488d13f64d2:manchester. (found with single-seed)
4. 630c24a90799fba5d29ba113b5585e0eb2b194618492694af05e16b3376a82a2:m@nchester (found with single-seed)
5. edfe3f86037802cf2e428f9d369836cb09fc026a2217936e9fa835091c55edd2:mnchstr (found with single-seed)
# Rounds

For this, we need a tool that lets us easily specify repeated rounds of a hash algorithm. Unfortunately, hashcat isn't good with this, so we turn to [mdxfind](https://www.techsolvency.com/pub/bin/mdxfind/#download) which works quite nicely with `./mdxfind -h '^MD5$' -i 100 -f md5rounds.txt ../rockyou.txt`

MD5x100 a40b32952e9c8b5103361a3981e80d94:hacs44421436
MD5x100 099c04d09c92276920076223baa0df11:abc123
MD5x100 e7865117d480bd92d3147a93982036a7:bubbles
MD5x100 aaebd8bb50bd81c90aae018dcf5d609e:angel
MD5x100 4ee6c7a7d43b953c655daee3d2730406:butterfly

#  Unknown Hash

Python's most popular hashing library by far is hashlib. As our hash is 128 hex characters or 64 bytes, we're looking for algorithms that produce 64 byte outputs. There are 2 possibilities: sha-512 (doesn't work) and Blake2b. We use hashcat again as `hashcat -a 0 -m 600 python_hashes.txt Pwdb_top-10000000.txt`

1. f6d9537d5cc8fc8e42181113adf778f266d34277029a35c1d51b1e8cd1a9570d5e8d4908c7a280342fe16cbd70ce1f8b85525e489b4240a6f565babefe956977:mommy1
2. db314cd58993e42026b658a31b81023d636afb834b39d834acb1cabd41948364fa5f1aa9451406ad5f2ffa69ca73a70febad30efae033ed16a9eb729cb8096b3:teddybear
3. a56ed0aa6b1c0e7ee2e7450ce09d78483cbf75819525f23e4643020762be69a95907bae5413edba524f6e7fea348cf1ca496ee3818fb83d5f368dc30413e151b:12345678910

#  Word

Ok, time for jumbo johntheripper. We'll use the `office2john.py` script to get our hash to crack via `python3 john/run/office2john.py ~/Downloads/crackme.docx > doc_hash.txt`, then crack that hash via `./john/run/john --wordlist=Pwdb_top-10000000.txt doc_hash.txt`

1. 123456789

# WPA 

We'll first convert this into a format johntheripper likes via `./john/run/wpapcap2john wpa.pcap > wpa_hashes.txt`, then build a wordlist via `cewl https://en.wikipedia.org/wiki/Mathematical_logic -d 3 -w math_words.txt`, then `./john/run/john --wordlist=math_words.txt wpa_hashes.txt` to crack the hash and find the password is Induction, then import it into Wireshark as `Induction:Coherer`to decrypt the network conversations [via these instructions](https://wiki.wireshark.org/HowToDecrypt802.11). 

![[Pasted image 20260303132240.png]]

# KeePass (kdb)

We'll first use the johntheripper tool keepasstojohn to convert it into a readable format for johntheripper via `./john/run/keepass2john keepass.kdb > keepass_hash.txt`, then we'll run johntheripper on the output via `./john/run/john --wordlist=../rockyou.txt keepass_hash.txt`to get the password iloveme. 

keepass.kdb:iloveme

# Zip2

We'll use a similar structure as above: call a tool to transform to a john-friendly format, then call john. In this case, we'll do `./john/run/zip2john secret_zip2.zip > zip2_hash.txt; ./john/run/john --wordlist=../rockyou.txt zip2_hash.txt` to get that the password is `password`. 

# LastPass

We'll use a database viewer to grab the second line of the data col where type = key. This is base64 encoded, so we'll decode it and convert to hex. We'll then pass this into hashcat using the LastPass type via `hashcat -a 0 -m 6800 cmd_lastpass.txt wordlist_to_attack_lastpass_vault.txt`. 

![[Pasted image 20260303223602.png]]
![[Pasted image 20260303223614.png]]

This gives us b2c589099ebe7a22bf080021afaa9b17:600000:valtoandre123@gmail.com:fireboltXB12r , so we have our password. Now we'll use a [lastpass-vault-parser tool](https://github.com/cfbao/lastpass-vault-parser?tab=readme-ov-file) and pass in the information we've gathered to produce an output. 
![[Pasted image 20260303224045.png]]
This produces a csv revealing a password for a website, showing we've cracked the vault. 
![[Pasted image 20260303224224.png]]
So, the user's password to dsw.com is B@h3mian_Rh@psody!
# EC 1 

Thankfully, jumbojohn takes care of this nicely, so we can run `./john/run/zip2john secret_pkzip2.zip > pkzip_hashes.tx; ./john/run/john pkzip_hashes.txt --wordlist=Pwdb_top-10000000.txt` to get the password koolkevin, unlocking the file and getting us 
```
Congratulations! You got the extra credit :) 

~ Your favorite TA (sorry Sruj)
```

# EC 2

For this, we'll extract the passwords in the keepass database. I used KeePassXC and exported my results as a CSV, before isolating just the passwords. Then, I wrote a python program to try every password in the list on each of the images using `steghide extract -sf {photo} -p {passwd}`. I'd post the program here but it's a massive array and the structure is simple enough. 

The results are as follows: 
crime_scene.jpg - x0L4LM7duVyeITIRy58M -> HACS408T{d0nut_cr0$s}

hackerman.jpg - zQUTpifcLlnPxboSwhVT - HACS408T{h4ck3r_m4n}

hackerman.jpg - 3cueUpCRnpEmcUWGa26f -> HACS408T{r341_mr_r0bot}