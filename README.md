# How to make a python email reply bot
In this tutorial we will be learning how to make a python program that can read emails and respond based on the content.  Once you have the fundamentals of the program, you can write your own functions to do anything.  I will provide a 2 examples: a weather bot and a remote commands/shell script executor (useful on your own machine rather than repl.it)

### What is IMAP?
IMAP stands for Ineternet Mail Access Protocol. Almost all email clients, even web based ones, login to your email providers email server using IMAP.  With IMAP there are folders to organize emails and emails stay on the server unless explicitly deleted.  

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
i.login("myemailaddress@outlook.com", "mySuperSecretPassword")
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
The object returned by fetch is complicated and hard to parse, so we will be using a second third-party module, ```pyzmail``` to parse them.  **Important: If you are installing pyzmail using pip on python 3.6 or above, you need to install ```pyzmail36``` instead of ```pymail``` or you will get an error in pip. **

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
Email messages can have 2 parts: a text part and an HTML part.  In pyzmail, they will either be ```None``` if it doesn't exist or if it does exist it will have a ```get_payload()``` method which returns ```bytes``` which can be decoded using ```.decode()``` with the charset stored in either ```msg.text_part.charset``` or ```msg.html_part.charset```.  The text part is plaintext, the HTML part is HTML to be rendered for the user.  
## SMTP in python basics
SMTP is similar to IMAP, but doesn't have as many commands.  To implement SMTP, we will be using the python built in library, ```smtplib```.  
### Connecting to the server
You should first find your providers SMTP settings by finding them in the list below or searching *your provider* smtp settings.  
## **TODO: ADD SMTP SETTINGS LIST**

Once you have your settings, you can connect to the server by initializing a new ```smtplib.SMTP()``` object and starting TLS:
```python
s = smtplib.SMTP('smtp.example.com')
s.starttls()
s.ehlo()
```
## TODO: finish tutorial

## Putting it all together: making the bot
Using our knowledge, we are going to write the bot.  First thing we need to do is store the configuration information.  For repl.it, we will use ```input()``` but on your own computer you can hard-code values (not passwords! ).  

Add a new file in repl.it called ```config.py``` and add the following: 
```python
############
# ACCOUNTS #
############
radr = 'bob@example.com'  # address to check and send from
imapserver = 'imap.example.com'  # imap server for account
smtpserver = 'smtp.example.com'  # smtp server for account
smtpserverport = 587  # smtp server port for starttls
pwd = b'bXlwYXNz'  # password for account encoded with base64.b64encode
sadr = 'alice@example.com'  # address to receive commands from
```
