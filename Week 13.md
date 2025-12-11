# Q1 

This name mangling occurs because compilers have to provide unique symbol names for functions that support function overloading or other language features; otherwise, two distinct functions with the same name would only ever resolve to one of the functions, leading to incorrect behavior. 

# Q2

Since many of the symbols are tokio functions, this binary was likely written in Rust. 
# Q3

Some packages: tokio (used for async/threading), io (in the name, just io), sync, http_body (representing async http handling), hashbrown (hashing). 

# Q4

The symbol `_ZN3std7process7Command6output17h43c2bcdedd19a7a1E`, or `std::process::Command::output`, is called when interacting with the webserver. 

# Q5

This might be because strcasecmp takes 2 strings, which can easily be rendered as arguments, while fstat64 takes in an int file descriptor and a pointer to a struct, which require considerably more effort to parse and represent dynamically. This is reflected in the handlers, which include utf-8 strings as arguments in the log for strcasecmp, but don't try to use any of fstat64's args in the log. 

# Q6

Note: The frida-trace command didn't work for me, so I just wrote a script to iterate through all the symbols frida caught. I hooked all those containing the string "Command", since that's responsible for executing commands in Rust  (of particular use are Command.output)

UI Elements that can trigger commands: To my knowledge, basically all of the 6 main widgets, from my experimentation. 

The injection is on the "Internet Connectivity" field, which appears at first not to accept arbitrary user input, but since its input is provided as a GET request via its url, we can! We can start our command with a valid IP followed by our injection (ex 1.1.1.1 && touch /home/xyz/myvirus.txt). First, I have to urlencode this value, which can be accomplished online via converters, and then it's just a matter of interacting with the field to figure out what its parameters are, and then adding that urlencoded string to the parameters to trigger the injection on the machine running the server. 

Ex: `uri=/api/internet-test?target=1.1.1.1+%26%26+touch+%2Fhome%2Fbdodge1%2Fobama.txt` interacts with the internet connectivity field to put the file obama.txt on the user's /home/bdodge1 directory. 
# Q7

A Premaster secret log file can be used to read a TLS key in Wireshark, decrypting HTTP/TLS packets that were encrypted using that session key. This file can be provided to Wireshark through Edit>Protocols>TLS>Premaster secret log name. 

# Q8

`SERVER_TRAFFIC_SECRET_0` encrypts data sent to the client after the handshake (like the website), and is present in the log file. 

# Q9

The script runs the program up to the first conditional statement, and then uses Z3 SMT solver to find the set of valid inputs that allow the statement to be taken vs not taken. Then, looking at those two input sets (taken vs not taken) one of them should be "SOSORRY", so we'll continue running the program at the corresponding location (which in our case is taken). 

# Q10

Command to run : `python3 solve_r100.py | ./r100`
Script: 
```python
import angr

def main():
    p = angr.Project("r100", auto_load_libs=False)
    simgr = p.factory.simulation_manager(p.factory.full_init_state())


    # Code used to find the input 
    simgr.explore(find=lambda s: b"Nice" in s.posix.dumps(1))
    
    return simgr.found[0].posix.dumps(0).strip(b'\0\n').decode("utf-8")

if __name__ == '__main__':
    print(main())
```