# Q1 

The program is statically linked, and is not stripped. 

# Q2 

The string is
```
runtime: sp=abi mismatchwrong timerstrace bufferillegal seekinvalid slothost is downnot pollablegotypesaliashttpmuxgo121multipathtcprandautoseedtlsunsafeekmHello, World!3814697265625wakeableSleepprofMemActiveprofMemFuturetraceStackTabexecRInternaltestRInternalGC sweep waitsynctest.WaitSIGQUIT: quitSIGKILL: killout of memory is nil, not value method heap metadata span.base()=bad flushGen created at: 
```
which is obviously not the length I would have expected. 

# Q3

This function's name is main.sayHello. 
# Q4

fmt.Fprintln is used to print "'Hello World"

# Q5

The prologues and epilogues seem to function similar to normal assembly: 
```asm
# Prolog
  main.go:7		0x498220		55			PUSHQ BP				
  main.go:7		0x498221		4889e5      MOVQ SP, BP				
  main.go:7		0x498224		4883ec38	SUBQ $0x38, SP				
# Epilog
  main.go:9		0x498265		4883c438	ADDQ $0x38, SP				
  main.go:9		0x498269		5d			POPQ BP					
  main.go:9		0x49826a		c3			RET		

```

# Q6

These are used for arguments: ```RAX, RBX, RCX, RDI, RSI, R8, R9, R10, R11```

# Q7

```  
  main.go:11		0x498248		b801000000		MOVL $0x1, AX // Set up registers	
  main.go:11		0x49824d		bb02000000		MOVL $0x2, BX		
  main.go:11		0x498252		b903000000		MOVL $0x3, CX		
  main.go:11		0x498257		e8c4ffffff		CALL main.FuncAdd(SB)	
  main.go:11		0x49825c		4883c041		ADDQ $0x41, AX // Get result from (R)AX	
```

# Q8

AX is used to return the error, and multi-argument returns are applied, and in some cases stretched across registers (Simple2 zeros out BX and CX to create the nil)

# Q9

CX has it, loaded via MOVQ 0x18(AX), CX

# Q10

The ITab struct is passed as the first argument to the saySomething function, which is distinct for each object. This struct contains a function pointer to that struct's implementation of the Speak interface, which allows the saySomething function to call the correct implementation of that function. 

# Q11

This binary uses the Zap go logging library, and some libraries provided by the go language itself, like crypto, net, sys, and text.

# Q12

This is used for configuring command-line flags, setting default values, and printing out what flags are intended to do. From this, we can see that this is a program designed for some kind of network connection and logging.  

# Q13 

There are 3 such functions, base64EncodeHandler, addUserHandler, and getUsersHandler. 

Since strings in error messages in bas64EncodeHandler seem to refer to a POST request, this is likely base64-encoding data sent in the payload of a post request. Strings mentioning Content-Type suggests that it also attaches headers as well, producing a full payload to be sent. Finally, it also prints out the base64 encoding of the data. 


addUserHandler seems to read in a json file in the body of a post request. This seems to have the fields age, name, and height based on some strings related to what I guess is logging. However, there's another segment of json generated previously that only has the message and name fields. I don't think this is related to the previous segment, so maybe it is a truncated version of the data contained in the body. Based on some output strings, it also writes this to another json object. 
![[Pasted image 20251112182600.png]]
getUsersHandler seems to retrieve some list of users using 2 http get requests, one with the "format" parameter and the other with the "sort" parameter. Depending on the response from the "sort" parameter, it does some sort of slicing and sorting on some data (possibly data returned from the response to the get request, but it's never parsed like it is in other spots).  After that, depending on the response from the request with the "format" parameter, it writes the data into either csv or json format. 

Based on sort:
![[Pasted image 20251112182107.png]]

Based on format:
![[Pasted image 20251112182135.png]]
# Q14

Just from the title alone, this sounds like some kind of (de)/multiplexer. With calls like redirectToPath, this seems to take incoming http requests and direct them to designated handlers (essentially, demultiplexing). 
![[Pasted image 20251112182738.png]]

I don't see any calls that build out URLs or POST request bodies or anything like that, so I don't think this function is sending out any traffic, just handling incoming traffic.

# Q15

I had difficulty triggering most of the functions. Since they're all oriented around responding to requests, that makes sense: whatever should be sending us these requests just doesn't exist. 

The only one I could break consistently on was `net/http.(*ServeMux).Handle`. First, I wanted to see how many times it was called, and what args it was called on, so I made a breakpoint and continued until the program ended (or in my case, just hang). 

![[Pasted image 20251112184903.png]]

This is the results of my first run, which was interesting, since these seem to allude to the other functions I couldn't hit through breakpoints (encode to base64, the 2 /api/users to the 2 user-related handlers). Next, I wanted to inspect the handlers themselves and see if I could somehow resolve them to something more useful than just a generic Handler name. 

These all seem to imply links to a website. Given what I learned about the ServeMux function, this is probably taking in requests, and parsing them based on how the url they sent to reach our server ends. Depending on the ending, the sender has requested a particular service, so we pass that over to the function that can handle that request. 

I couldn't really find anything more meaningful, or resolve the arguments into something specific. 