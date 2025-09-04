Just for reference, each hex character can represent 4 bits, so 2 hex characters (0xFF) can represent 1 byte. 

We often use bytewise addressing, where one bit in the address represents a 1-byte region in memory. For example, consider the following 

```
00000000: 504b 0304 1400 0808 0800 a49c e74e 0000
00000010: 0000 0000 0000 0000 0000 1400 0400 4d45
```

We can see for each entry, there's 16 hex pairs, or 16 bytes. That's because the offsets are 0x10, or 16 apart. This tell us that the offsets are using bytewise addressing. 

