
# Misc 

## Yuri

We see that the files differ by specific characters, subtracting their difference produces the flag. 

```python
flag = ""

body1 = ""
body2 = ""
with open("yuri_1.txt", "r") as f1:
    body1 = f1.read()
with open("yuri.txt", "r") as f2:
    body2 = f2.read()

for i in range(len(body1)):
    if body1[i] != body2[i]:   
        flag += chr(ord(body1[i]) - ord(body2[i]))

print(f"Flag is {flag}")
```

