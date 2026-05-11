
We're looking for a way to get a shell in this Python server, so we'll  start looking for command injections. We find one in `option3`: 
```python
def option3(conn):
	send(conn, "Enter file to create \n")
	data = conn.recv(1024).strip().decode('utf-8', 'ignore')
	
	if os.getcwd() == '/trusted': # NOTE: We need to find a way to get to /trusted
		# NOTE: We can execute arbitrary commands here by passing the input x;<our command>
		out = subprocess.check_output("touch " + data, shell=True)  
		send(conn, out.decode()) # And we get the output of that command back to us!
	else:
		os.system("touch " + alnum(data))
		send(conn, "Created file.")
	return False
```

Ok, so how can we move our program's working directory to `/trusted?` That's where `option7` comes in. 
```python
def option7(conn, admin):
	if admin:
		send(conn, "Enter target directory \n")
		data = conn.recv(1024).strip().decode('utf-8', 'ignore')
		if data[0] == '\\' or data[0] == '~' or data[0] == '/':
			return True
	else:
	os.chdir(data # NOTE: This is what we want
	send(conn, "Changing directory...\n")
	# program continues...
```
Since we can't change to `/` directly (it's filtered), we can chain a series of `..`, moving up a directory, until we reach the root. But how will we know when we've reached the root? Thankfully, `option5` sends us our current directory, and does nothing else. So, we can call option5 once, figure out how many directory we'll have to escape by counting the number of `/` characters, and then call `option5` and send it `..` that number of times, and then call `option5` again and pass in `trusted` to get to `/trusted`. 

That's all well and good, but what about that if statement with the admin check in `option7`? We need that to be `True` to change directories, and it's initialized to `False`. Not to worry, `option9` allows us to set admin to `True`. 

```python
def option9(conn, pin):
	send(conn, "Please enter your pin. \n")
	data = conn.recv(1024).strip().decode('utf-8', 'ignore')
	# Omitting less-relevant code
	i = 0
	while i < len(pin):
		# Go through the pin
		if pin[i] == data[i]:

			time.sleep(1) # This is a long time! If it takes longer, we know our guess was correct
		else:
			send(conn, "Your pin was incorrect.\n")
			time.sleep(.1) # This is a much shorter time - if it finishes faster, we know we were wrong
			return False
			i += 1
		

	send(conn, "Your pin was correct!\n")
	return True
```
We can perform a timing attack on this: if we receive a response in `i + 1` seconds or more, we know the first `i` digits of our guessed pin match the actual pin. As we see elsewhere in the code that the pin is 10 digits, we can start with `"0"*10`, then increment each digit in the string by 1 from left to right, and check if that digit matches the pin by seeing if we get a response in `i + 1` seconds or more. If we don't, we'll increment the digit, and if we get a response in `i + 2` seconds or more, we know that digit and the digit after it are correct. 

Ok, we've got all the tools to put this exploit together: 
1.  First, we'll set `admin = True` by performing a timing attack on `option9`
2. Then, we'll change to `/trusted` through `option7` and `option5`. 
3. Finally, we'll perform a command injection through `option5`, and return the output to the user. 

Some final notes: we can't choose which option to select, they're randomly chosen for us. We can get around this by writing an output parser, which sees if the output is for the option we're looking for, sends 'y' if it is, or skips it if it isn't. Also, as this option text may print immediately after some of our functions, we may accidentally capture it while calling`recv` for  data at the end of our other steps in the exploit. This starves our option parser. To fix this, we can set a timeout for our socket in our option parser, and skip to the next option if it expires, ensuring our parser won't block forever. 