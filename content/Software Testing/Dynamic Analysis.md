
![[Pasted image 20250829175318.png]]

While static analysis can provide coverage of the whole program, its view is often blurry. It can't determine some parts of the program that rely on dynamic behavior. Dynamic analysis provides more limited coverage, but provides us with much more accurate analysis: we know for sure this is how the program behaves. 

Ideally, we'd want to analyze all possible program states that could occur dynamically, and verify that the program is secure throughout all of them. How can we do this? The easiest solution would be to determine what dynamic inputs are necessary to trigger all possible control flow paths, and walk through those by running the program a bunch of times. However, that strategy has a couple limitations. 
- It limits us to only walking along paths that we can find statically. It may be possible for new control flow patterns to emerge during program execution (say, as a result of user input). 
- It doesn't guarantee we'll encounter all program states that are possible along each control flow path. 
	- Consider an if statement based on user input, where one case has a buffer overflow vulnerability based on the same user input. If all we care about is simply visiting all control flow paths, we could miss the vulnerability and the new program state that results from it. For example, we could input a single character to satisfy the if statement, and never actually overflow the buffer. 

# Fuzzing

TODO: Read https://www.fuzzingbook.org/html/Tours.html
# Forced Execution

Another variation of fuzzing is called **forced execution**. We'll take no input, and run sequentially. Whenever we meet a predicate, we'll explore all possible outcomes regardless of whether the predicate is satisfied or not. This may lead to uninitialized variables, or variables with values we don't expect. We'll try to recover from any crashes or errors this may produce, and fuzz any variables that are the source of those errors. 

![[Pasted image 20250829194858.png]]

This allows us to leverage insights about program structure without fuzzing things we don't have to. 

# Concolic Execution