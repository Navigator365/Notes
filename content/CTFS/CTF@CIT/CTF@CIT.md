
# Web

## A Massive Problem

Looking at the source, we need to meet the following conditions to reach `/admin`: 
```python
@app.route('/admin')
def admin():
    if 'username' not in session:
        return redirect(url_for('login_page'))
    if session.get('role') != 'admin':
        return redirect(url_for('dashboard'))
    return render_template('admin.html', username=session.get('username'), flag=os.getenv('FLAG', 'CIT{test_flag}'))
```

By default, our role is set to `standard`, so we'll need to update it. Thankfully, there's a way to do so using `/api/profile`, but it requires a session cookie for authentication. So, we can log in on the web, copy our session cookie and use it in curl along with a post request setting our role to admin. Then, we'll log in again and navigate to `/admin`. 

## Debug Disaster

We're greeted with an empty webpage. We start by directory enumeration with gobuster, and find `\admin`. Going there, we see a debug stack trace. Clicking around it, we can see code snippets referring to a `\flg_bar` Flask route. Visiting there gives us the flag. 

## Temporary Destruction

We're given the opportunity to send input, and have it echoed back to us. This is a prime target for SSTI! First, we'll figure out what template language it uses; turns out to be Jinja. 

After a while of searching for a working payload, I found [this](https://medium.com/@freyaroselle/breaking-a-hardened-jinja2-ssti-ctf-write-up-d2104af8cae4). I had to adapt it a bit as a I searched for the flag's location on the machine. The lesson: ALWAYS USE FIND FIRST!!. It turned out to be in /tmp of all places. 

## Intern Portal

Here, we see a notes app with an insecure GET parameter, allowing us to access other people's notes. We manually search a while for notes until we find one containing the flag. 

## Hit Your Limit

There's gotta be a better way other than just suffering through ratelimiting like this???

THREADING!!!!

## Sign up and enjoy

Flask tokens aren't just random: they're encoded, so they can be decoded, and are signed by a secret 

## VERY INTERESTING:

ln -s /home/jimbo/flag2.txt /srv/ftp/flag2
curl ftp://ftp:@localhost:10921/flag2
^ this allows the ftp server (running with higher privileges) to access things we can't, using a symlink we created

https://medium.com/@shazop/ctf-cit-2026-writeups-153fb0447fc1