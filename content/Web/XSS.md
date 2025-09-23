At its core, XSS is a client-side attack where users unknowingly execute malicious code inserted into a webpage they visit. This code executes in the victim's browser acting as the victim, allowing access to their data, and, if the victim has sufficient privledges,  the whole application. 

By entering HTML containing our payload, we can trick the website into rendering our text as proper HTML, inserting it into the site itself. Once inserted, the code in our payload can run, either in response to user action or automatically. 

## Stored XSS

In stored XSS, an XSS payload is stored on the backend, and executed whenever a user requests the webpage containing the payload. For example, the [Samy worm](https://en.wikipedia.org/wiki/Samy_(computer_worm)) made posts on MySpace containing an XSS payload that would make the visiting user post the same payload on their profile. As you might imagine, stored XSS attacks are especially severe, as they can target a large audience at once (anyone who views the webpage containing the payload), and lend themselves to easy self-replication like in the example above. 

## Reflected XSS

In Reflected XSS, payloads are processed on the backend but are returned without being sanitized. This can happen any time user input is displayed within a website response. For example, an error message that contains user input could be vulnerable to a reflected XSS attack. These attacks are non-persistant, since the payload is only returned and inserted into the webpage in response to a specific user input, instead of being stored and made available to other users. 

These attacks require an external delivery mechanism. Often, that happens through payloads encoded as parameters in URLs. If anyone clicks those URLs, the payload will be executed on their browser. Attackers can make use of this to include malicious links targeting websites with XSS vulnerabilities, allowing them to execute scripts that grab user credentials and send them to servers the attackers control. 

### Phishing/Creds Stealing

If a website renders HTML elements in response to URL parameters, we can use a reflected XSS attack to insert our own payload. This is actually a really common method of phishing! Attackers will create links with XSS payloads that modify a website's appearance, often by adding a new login form. When users click those links, they'll be presented with the form. They'll login, since they trust the website they're on, and they'll be redirected to their account screen. But that form was actually fake, inserted by the attackers, and redirecting all information in it to a server attackers control. This allows attackers to steal user creds without arousing a user's suspicion: after all, they were on a legitimate site the entire time!
## DOM-Based XSS

These vulnerabilities happen whenever attacker-controlled data can be used as input to front-end JS functions that allow for dynamic code execution. 

Just like with reflected XSS, attackers can use DOM-Based XSS in malicious URLs, passing in '#payloads' which would execute whenever the user clicked the link. Also like reflected XSS, this is a non-persistent attack: since it's entirely client-side, there's no way for the payload to make its way to the server.  

Look for sources of user input, and then HTML or JS code elements that reference it, and see how they interpret that input. Different sources, called sinks, will perform different operations. Depending on which sink is vulnerable, you may have to structure your payload differently. 

For example, a javascript function might modify a URL to search for based on user input. You just have to find out how the HTML element controlling the URL is structured, and boom! XSS.

If that doesn't work, look for JS elements in general and see if they allow for user input, and what they do with that user input.

## Blind XSS

Blind XSS refers to XSS that's triggered on pages we can't access; we're injecting blind. As you might imagine, these are tricky: it's much easier to do these using a DOM-Based XSS vulnerability, since we can precisely construct a payload we know works. The other two techniques are essentially trial and error, as we don't know what input sanitizing the sever does. 

We'll often try to find a working XSS payload by trying a bunch of possible payloads on all fields.  Each payload will have a src=IP/field_name, where IP is the IP of a server we control. We'll log all these, and find out which payloads on which fields are successfully injected and executed on the webpage we can't access. 

## XSS Prevention

In general, we want to validate input (is this input what I expected? If I asked for an email, did I get a valid address?) and sanitize input (remove any input with code in it)

### Frontend

Never use user input directly within HTML tags, or allow user input as arguments to functions  that change the raw text of HTML fields. 
### Backend

[Output encoding](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html#output-encoding): When displaying input exactly as the user typed it in, convert any characters with special meanings in HTML ('<', '&', etc) into literal text (&lt;, &amp;). We don't want any user code to ever be executed, and this conversion helps accomplish that. 