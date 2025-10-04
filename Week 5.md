# Lab 1

## Q1

`68.183.56.196` is hosting the malicious site, which grabs Microsoft Live credentials and pretends to be Adobe. 

## Q2
The string 'OR''=' as the value of each url parameter was used to trigger this injection. 

## Q3

The attacker logged in as jake. 

## Q4

The attacker used the whoami command first, but this was unsuccessful as it was incorrectly formatted. 

The secret message was "This message should not be accessible to the outside world!"


## Q5

The most common application layer protocol was DNS. The TLD was `.xyz`. 

# Lab 2

## Q6

The victim machine's hostname was DESKTOP-USER1PC, and and requested `10.5.7.101`. 

## Q7

The victim's mac address was `00:08:02:1c:47:ae`. 

## Q8

Some hostnames are google.com, parisyoungerfashion.com, 

## Q9 

Yes, there are requests to urls ending in fre.php, which is a known technique of lokibot. 

## Q10

Some domains are msftncsi, and an extremely long and strange domain niten..... used to make a request to an exe download. 

## Q11

The domain nitenrgystdyluncstgw.dns.army waas used to download an invoice file, and was later used to download an exe. 

## Q12 

The file is a PE32 executable, and its magic bits match (4d5a). 

## Q13

The sums are as follows: 

```
30325168f0fffaf2d28d44826667d5178ff2788cb93a0449897800c2f2ba2e6e  regasm.exe
b14395003e5efba733d717f89486aee8222abf00b33190ea2d34e7b68d2bca73  fre.php
6137f8db2192e638e13610f75e73b9247c05f4706f0afd1fdb132d86de6b4012  ncsi.txt
5b2c34b3c4e8dd898b664dba6c3786e2ff9869eff55d673aa48361f11325ed07  favicon.ico
24c52778e78ed9e324f3ae88d3f1b929c825d2d671ffa149129dca572ce250ea  invoice_908866.doc
```


## Q14 

`172.16.1.103, 51.104.164.114, 104.64.158.58, 40.122.160.14, 52.114.32.25, 204.79.197.200, 178.33.183.53`, and the 172 IP uses ephemeral ports. 

## Q15

11 denotes certificate. 

## Q16

A normal certificate has a sequence with a country name, an organization name from a known organization, and a descriptive common name. The malicious cert instead uses id fields like stateOrProvinceNames filled with obviously incorrect information (thefyfrer5), with an organization name of `likenessS.r.l.` which is obviously not the name of any organization, and the final common name appears to be the name of a `.java` program, also with a gibberish name. 