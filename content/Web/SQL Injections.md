sql both query and boolean return
OR Injection: leverage that OR evaluates after AND to bypass auth
	- show example
	- also abuse quote placement to close quotes, put sql code, then reclose missing quote somewhere later down the line
	- comment abuse

UNION: leak data but have to ensure same colcount
	- solution: pad w junk data to balance out
	- seems like you have to enumerate all cols from table -> requires some prev knowledge
		- good when we can see query results so we know num of column
	- actually we can detect with ORDER BY/UNION
	- can leak system data too rather than just other table data

INFO_SCHEMA
	- figure out which cols are displayed first
	- Leverage info, display with union commands, get the data we want

Can r/w files 
	- can install my own shells/malware onto server
		- abuse union -> don't want to display anything, just write something but still have to follow column enforcement
		- need to find file path - try reading from configurations
		- may sound dumb, but important: remember where you are when writing, and try there first -> so if you're in /dashboard, write to /user/www/..../dashboard...
mitigations: sanitization (esc) validate (expected pattern), parameterized queries to separate query logic from user input