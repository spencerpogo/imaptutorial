# How to make a python email reply bot
In this tutorial we will be learning how to make a python program that can read emails and respond based on the content.  Once you have the fundamentals of the program, you can write your own functions to do anything.  I will provide a 2 examples: a weather bot and a remote commands/shell script executor (useful on your own machine rather than repl.it).  Sorry for the tutorial being so long.  If you just want to get to the bot, skip the "IMAP in Python Basics" section and the "SMTP in Python Basics" sections.  

### What is IMAP?
IMAP stands for Internet Mail Access Protocol. Almost all email clients, even web based ones, login to your email providers email server using IMAP.  With IMAP there are folders to organize emails and emails stay on the server unless explicitly deleted.  

### What is SMTP?
SMTP stands for Simple Mail Transfer Protocol.  It is the standard protocol used for sending emails across the internet.  SMTP is used to both send and receive emails, but we will only be sending.  



## IMAP in Python Basics
To implement IMAP we will be using the third-party module ```imapclient``` (docs [here](https://imapclient.readthedocs.io/en/2.1.0/)).  

### Connecting to the Server
Most servers can be found with a simple google search of "*your provider* imap settings", but here are the most common ones:  

**Gmail:** imap.gmail.com  
**Outlook.com/Hotmail.com:** imap-mail.outlook.com  
**Yahoo.com:** imap.mail.yahoo.com  
**AT&T:** imap.mail.att.net  
**Comcast:** imap.comcast.net  
**Verizon:** incoming.verizon.net  

Now you can connect to the server, the ```imapclient``` module makes this easy.  All you have to do is initialize an ```imapclient.IMAPClient()``` object using the server name, and it will connect to it on the default IMAP port using SSL. For example:
```python
import imapclient
i = imapclient.IMAPClient('imap-mail.outlook.com') # i is easy to type
```
### Logging in
Logging in is very simple.    
```python
i.login('bob@example.com', 'mySuperSecretPassw0rd')
```
The ```i.login()``` function is pretty self-explanatory, it logs in using the email and password provided.  

### Selecting a folder
Next we need to select a folder.  ```i.list_folders()``` shows the available folders.  You should choose the one you want, usually "INBOX".  
```python
i.select_folder('INBOX')
```
You can also, for debugging, pass in the keyword argument ```readonly=True``` to select_folder() to make the server not mark the messages you fetch as read.  

### Search for messages
Now we can search for a message.  There are many search criteria, here is [a full list](https://gist.github.com/martinrusev/6121028). Some of the most common ones are ```'ALL'``` which selects all message, ```'UNSEEN'``` which gets all unread messages.  The search flags behave differently which each email server, so you should first experiment with them in the shell.  You can search through the messages in the folder using ```i.search(['criteria'])```.  To get a unread messages, use the following code:
```python
i.search(['UNSEEN'])
```
This will not return the emails themselves, but a list of unique IDs or UIDs, for example ```[40032, 40033, 40034]``` which we are about to use.  
You can also use a special method if you are using Gmail, ```i.gmail_search()``` which behaves like the search box in Gmail.  

#### Size Limits
If your search matches lots of messages, you can get an ```imaplib.erorr: got more than 10000 bytes```.  This can be fixed by the following code:
```python
import imaplib
imaplib._MAXLINE = 1000000
```
This sets the limit to 10,000,000 bytes instead of 10,000.  

### Fetching emails
When you fetch an email, you download it from the server.  Unless you are using ```readonly=True```, when you fetch an email it will mark it as read.  You fetch an email like this:
```python
rawmsgs = i.fetch(uids, ['BODY[]']) # uids is the uids returned by search()
```
The object returned by fetch is complicated and hard to parse, so we will be using a second third-party module, ```pyzmail``` to parse them.  ** Important: If you are installing pyzmail using pip on python 3.6 or above, you need to install ```pyzmail36``` instead of ```pymail``` or you will get an error in pip. **

This is how you parse a message with pyzmail:
```python
import pyzmail
msg = pyzmail.PyzMessage.factory(rawmsgs[40041]['BODY[]']) # Be sure to change the uid number
```
Using this object you can get a lot of information on the message.  
```python
msg.get_subject()
msg.get_addresses('from')
msg.get_addresses('to')
msg.get_addresses('cc')
```
Sample Output:
```
Thanks!
[('Alice Doe', 'alice@example.com')]
[('Bob Smith', 'bob@example.com')]
[]
```
These methods get the subject and addresses the message was sent to.  
### Reading Messages

```python
msg.text_part != None
msg.text_part.get_payload().decode(msg.text_part.charset)
msg.html_part != None
msg.html_part.get_payload().decode(msg.html_part.charset)
```
Sample Output:
```
True
'Thanks for buying lunch yesterday. \r\n\r\n-Alice\r\n'
True
'<div style="font-family: courier;">Thanks for buying lunch yesterday. <br><br>-Alice</div>'
```
Email messages can have 2 parts: a text part and an HTML part.  In pyzmail, they
 will either be ```None``` if it doesn't exist or if it does exist it will have
 a ```get_payload()``` method which returns ```bytes``` which can be decoded
 using ```.decode()``` with the charset stored in either ```msg.text_part.charset```
 or ```msg.html_part.charset```.  The text part is plaintext, the HTML part is
 HTML to be rendered for the user.  
 
 
 
## SMTP in python basics
SMTP is similar to IMAP, but doesn't have as many commands.  To implement SMTP,
we will be using the python built in library, ```smtplib```.  
### Connecting to the server
You should first find your providers SMTP settings by finding them in the list
below or searching *your provider* smtp settings.  
**Gmail:** imap.gmail.com  
**Outlook.com/Hotmail.com:** imap-mail.outlook.com  
**Yahoo.com:** imap.mail.yahoo.com  
**AT&T:** imap.mail.att.net  
**Comcast:** imap.comcast.net  
**Verizon:** incoming.verizon.net  

Once you have your settings, you can connect to the server by initializing a new
 ```smtplib.SMTP()``` object and starting TLS:
```python
s = smtplib.SMTP('smtp.example.com', 587)
s.starttls()
s.ehlo()
```
### Troubleshooting
If you get an error while connecting or if your email provider is listed above 
as *port 465* then your email provider may not support TLS on port 587.  In 
this case, you should connect using SSL to port 465 like this:  
```
s = smtplib.SMTP_SSL('smtp.example.com', 465)
s.ehlo()
```
### Logging in
Logging in is similar to IMAP, you call the ```login()``` function of your SMTP
connection, using your email and password: **Be careful about leaving passwords
in your code!**
```
s.login('bob@example.com', 'mySuperSecretPassw0rd')
```
### Sending emails
To send emails, I have found that the ```sendmail()``` function doesn't really
work as it can mess with email headers, instead we should use the python 
built-in module ```email```'s ```email.message.EmailMessage()``` class which
makes settings the headers really easy and has many advanced capabilities which
are listed in [the docs](https://docs.python.org/3/library/email.message.html#email.message.EmailMessage).  
Here is an example of how to send a text message:
```python
text = """Dear Alice,
You are welcome.  Think nothing of it! 

-Bob
"""
msg = email.message.EmailMessage()
msg['from'] = 'bob@example.com'
msg["to"] = 'alice@example.com'
msg["Subject"] = "Re: Thanks! "
msg.set_content(text)
res = s.send_message(msg)
```
In most email providers, though not required, prefacing the subject with Re:
denotes a reply.  In this example, Re: Thanks! would show up in the same
thread as the original Thanks! message in both email clients.  To return
value of the ```send_message()``` function is a dictionary of the addresses
to which sending failed.  If successful, it should return ```{}```.  




## Putting it all together: making the bot
Using our knowledge, we are going to write the bot.  First thing we need to do
is store the configuration information.  For repl.it, we will use ```input()```
but on your own computer you can hard-code values (not passwords! ).  

### Using a configuration file
Add a new file in repl.it called ```config.py``` and add the following:
```python
import getpass

radr = input("Adresses to log in to")  # address to check and send from
imapserver = input("IMAP server domain: ")  # imap server for account
smtpserver = input("SMTP server domain: ")  # smtp server for account
smtpserverport = input("SMTP Server port [587]: ")  # smtp server port for starttls
if not smtpserverport or smtpserverport == "":
    smtpserverport = 587
pwd =  getpass.getpass("Account password: ") # password for account encoded with base64.b64encode
sadr = input("Trusted addresses to receive from: ")  # address to receive commands from
check_freq = 5
```
Here we set the config values to be used by the main program.  For security, we
will set a trusted email that is the only one that commands will be accepted
from.  This is so that no random people can email us commands.  

### Initializing the IMAP connection
```python
from config import *

def imap_init():
    """
    Initialize IMAP connection
    """
    print("Initializing IMAP... ", end='')
    global i
    i = imapclient.IMAPClient(imapserver)
    c = i.login(radr, pwd)
    i.select_folder("INBOX")
    print("Done. ")
```
Here we initialize the imap connection by import our config file
(using ```import *``` is usually a bad idea because you don't know where things
came from but we are just importing variables so it is okay).  
We also define i using ```global i``` so that it is available to the rest of our
 program.  We also login to the server and select the "INBOX" folder.  
### Initializing the SMTP connection
```python
def smtp_init():
    """
    Initialize SMTP connection
    """
    print("Initializing SMTP...")
    global s
    s = smtplib.SMTP(smtpserver, smtpserverport)
    c = s.starttls()[0]  # The returned status code
    if c is not 220:
        raise Exception('Starting tls failed: ' + str(c))
    c = s.login(radr, pwd)[0]
    if c is not 235:
        raise Exception('SMTP login failed: ' + str(c))
    print("Done. ")
```
In this block, we initialize the SMTP connection.  We use the same ```global```
technique as with IMAP and connect to the SMTP server and login, but we also
check the status codes returned by the server to make sure everything was
successful (we can't do this with IMAP).  

### Getting unread emails
The next step is getting any unread emails.  
```python
def get_unread():
    """
    Fetch unread emails
    """
    uids = i.search(['UNSEEN'])
    if not uids:
        return None
    else:
        print("Found %s unreads" % len(uids))
        return i.fetch(uids, ['BODY[]', 'FLAGS'])
```
Here we define a function to get unread emails.  This function searches the IMAP
object for any unread emails.  It returns ```None``` if it didn't find any, or
if it did, it fetches them from the server and returns them.  

### Defining commands
For our bot to work, we have to define some actions for it.  To do this, we will
define functions in ```config.py``` and add them to a commands ```dict``` which
will map the command names to the functions.  The functions will take 1
argument: the message split by lines.  For now, we will make a command that
will return "Hello, World! " every time no matter the message content:
```python
def hello_world(lines):
    return "Hello, World! "

commands = {"hello" : hello_world}
```

### Analyzing the message
Now we will analyze the message and determine what to do about it.  
```python
def analyze_msg(raws, a):
    """
    Analyze message.  Determine if sender and command are valid.
    Return values:
    None: Sender is invalid or no text part
    False: Invalid command
    Otherwise:
    Array: message split by lines
    :type raws: dict
    """
    print("Analyzing message with uid " + str(a))
    msg = pm.factory(raws[a][b'BODY[]'])
    frm = msg.get_addresses('from')
    if frm[0][1] != sadr:
        print("Unread is from %s <%s> skipping" % (frm[0][0],
                                                   frm[0][1]))
        return None
    global subject
    if not subject.startswith("Re"):
        subject = "Re: " + msg.get_subject()
    print("subject is", subject)
    if msg.text_part is None:
        print("No text part, cannot parse")
        return None
    text = msg.text_part.get_payload().decode(msg.text_part.charset)
    cmds = text.replace('\r', '').split('\n')  # Remove any \r and split on \n
    if cmds[0] not in commands:
        print("Command %s is not in commands" % cmds[0])
        return False
    else:
        return cmds
```
Let's break this down.  First we initialize the message object with the data we
are given.  Next, we make sure it is from our trusted address.  Next we set the
subject to start with Re: so that it shows up as a reply in the email thread.  
Next, we make sure we have a ```text_part``` to parse and split it by lines.  
We check that the requested command is a valid command, and if so, return the
line of the message.  

### Defining a mail function
Defining a mail function will be useful so that we can see in the logs what was
mailed to the user, and it will make the sending mail process as easy as passing
the function the mail text.  
```python
def mail(text):
    """
    Print an email to console, then send it
    """
    print("This email will be sent: ")
    print(text)
    msg = email.message.EmailMessage()
    global subject
    msg["from"] = radr
    msg["to"] = sadr
    msg["Subject"] = subject
    msg.set_content(text)
    res = s.send_message(msg)
    print("Sent, res is", res)
```

### Writing the event loop
We will write the loop our bot goes through as it waits for a message to
process. We want our bot to run until we interrupt it, so we will put it in an
infinite loop.  
```python
imap_init()
smtp_init()

while True:  # Main loop
```
Next, we will check for any unread messages and analyze them.  (You will see why
it is in a ```try:``` block later)
```python
    try:
          print()  # Blank line for clarity
          msgs = get_unread()
          while msgs is None:
              time.sleep(check_freq)
              msgs = get_unread()
          for a in msgs.keys():
              if type(a) is not int:
                  continue
              cmds = analyze_msg(msgs, a)
```
We get the messages, and if there are none, we enter a loop that won't exit
until a new message arrives, and waits for the ```check_freq``` amount of time
before checking again.  Once we have found a message, we analyze it.  
```python
              if cmds is None:
                  continue
              elif cmds is False:  # Invalid Command
                  t = "The command is invalid. The commands are: \n"
                  for l in commands.keys():
                      t = t + str(l) + "\n"
                  mail(t)
                  continue
              else:
                  print("Command received: \n%s" % cmds)
                  r = commands[cmds[0]](cmds)
              mail(str(r))
              print("Command successfully completed! ")
```
Here we check the return value from ```analyze_msg()```.  If it is None (meaning
it is not from the trusted sender or doesn't have a text part) we skip it and
continue to the next message.  If it is False, meaning it is an invalid command,
we helpfully send the user a message reminding them of the available commands.  
Otherwise, we assume we got the text to fun so we run the corresponding command.  
When it finishes, we send back the result and print a message to the console.  
The bot is complete!  

### Testing the bot
Now we can run the bot and test it!  If you run the bot and send a message to
the email it is connected to with the first line being ```hello``` then it
should reply back "Hello, World! ".  If you get any error, make sure you followed
each step and if you think the error is on my part, let me know in the comments
and I will fix it.  

## Example commands: Weather Checker
Here I will provide you with one of two example command: a program that gets the
weather using the openweathermap.org API for your local area and emails it back
to you!  

### Setup
Before you can use this program, you need to find a couple of values.  To find
your city's id, search for it at https://openweathermap.org/find and click on it
in the results.  The city id is the string of numbers at the end of the URL.  
To get your API key, follow the steps here: https://openweathermap.org/appid.  
This requires you to create a free account and wait one hour for you API key to
be activated.  Keep in mind the limit for calls with a free account is one per
10 minutes, so don't check too often or your account could be suspended.  Now
that you have your values, replace them in the code below.  Also, we have one
third-party dependency: the ```requests``` module.  Don't forget to install it
if you haven't already.  

### The code
```python
import requests
from json.decoder import JSONDecodeError
def weather(lines):
    city_id = str(5391959)
    url = "https://api.openweathermap.org/data/2.5/forecast?id="
    api_key = "your api key here"
    res = requests.get(url + city_id + "&APPID=" + api_key + "&mode=json")
    if res.status_code != 200:
        return "Oops, code was " + code
    try:
        j = res.json()
    except JSONDecodeError:
        return "JSON decode error: \n" + res.text
    try:
        main = j["city"]["list"][0]["weather"][0]["main"]
        description = j["city"]["list"][0]["weather"][0]["description"]
    except Exception as e:
        return str(e) + "\n" + r.text
    return "The weather is: \n%s: %s" % (main, description)
```
Let's break this down.  First we define our config values and get the API page
using them.  Then, we make sure the request was successful (if the status code
was 200) and send a message back if it wasn't.  Next we attempt to decode the
JSON returned by the API and send back if we can't.  Finally, we get the weather
and the description of it and catch any error that occurs from it (usually the
key not existing because of an error with the call) and send back the error and
full content of the request so that we can diagnose the error.  Finally, if all
was successful, we return the weather to the user!  

## Example command: Command and Shell Script executor
This command won't be useful on repl.it, but rather on your own server.  This is
really two commands: one that will run a single command, and one that will write
the message contents to a file on disk and execute it. (useful if you want to
run commands in the same working directory or with variables because if you only
run one command at a time, the shell will be reset each time).  

### The code: command executor
```python
import subprocess as sub
def exec_cmd(cmds):
    param = cmds[2]
    """exec_cmd: executes a command with subprocess
    """
    try:
        p = sub.run(param, shell=True, timeout=20, stdout=sub.PIPE, stderr=sub.PIPE)
    except sub.TimeoutExpired:
        return "Command timed out.  "
    return '''Command exited with code %s
Stdout:
%s
Stderr:
%s
''' % (p.returncode, p.stdout.decode(), p.stderr.decode())
```
This command isn't super complicated.  We take the command to be executed as the
second line and run it using ```subprocess``` with a timeout of 20.  If the
timeout expires, we inform the user.  Otherwise, we send the return code,
STDOUT, and STDERR.  

### The code: shell script executor
```python
def runscript(lines):
    """
    Writes input to disk and executes it.  Returns any errors.
    IMPORTANT:
    Before running, make sure file 'script' exists and is executable.
    """
    try:
        f = open("script", "w")
        count = 0
        for i in lines:
            if count is 0 or i is 1:
                count = count + 1
                continue
            f.write(i + "\n")
            count = count + 1
        f.close()
        p = sub.run("./script", shell=True, timeout=timeout, stdout=sub.PIPE, stderr=sub.PIPE)
        r = """Script finished!
Return code: %s
Stdout:
%s
Stderr:
%s
""" % (p.returncode, p.stdout.decode(), p.stderr.decode())
    except Exception as e:
        return "Error: \n" + str(e)
    return r
```
This code is extremely similar to the single command runner code, except for it
loops through the lines for the message (ignoring lines 1 and 2) and writes them
to a file named ```script``` which, as mentioned in the docstring, should exist
and be executable before this command runs.  

### Adding the commands to the bot
To add these commands to the bot, and to define you own, paste the functions
into ```emailcmd_config.py``` and update the the ```commands``` dictionary at
the bottom by putting the key as the alias for the command and the value as the
function object.  
```
commands = {"weather":weather, "exec":exec_cmd, "script":runscript}
```
