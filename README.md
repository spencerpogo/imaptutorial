# How to make a python program to read and respond to emails
Hey guys!

In this tutorial we will be learning how to make a python program that can login and read emails from our email account using IMAP and send messages using SMTP! 
The possible applications of this are endless, from email bots to an email shell! 

### What is IMAP?
IMAP stands for Ineternet Mail Access Protocol. Almost all email clients, even web based ones, login to your email providers email server using IMAP.  With IMAP there are folders to organize emails and emails stay on the server unless explicitly deleted.  

### What is SMTP?
SMTP stands for Simple Mail Transfer Protocol.  It is the standard protocol used for sending emails across the internet.  SMTP is used to both send and receive emails, but we will only be sending.  

## IMAP in Python Basics
To implement IMAP we will be using the third-party module ```imapclient``` (docs [here](https://imapclient.readthedocs.io/en/2.1.0/)).  

### Connecting to the Server
Most servers can be found with a simple google search of *<your provider> imap settings*, but here are the most common ones:  

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
The object returned by fetch is complicated and hard to parse, so we will be using a second third-party module, ```pyzmail``` to parse them.  **Important: If you are installing pyzmail on python 3.6 or above, you need to install ```pyzmail36``` instead of ```pymail``` or you will get an error in pip. **
This is how you parse a message with pyzmail:
```python
import pyzmail
msg = pyzmail.PyzMessage.factory(rawmsgs[40041]['BODY[]']) # Be sure to change the uid number
```
Using this object you can get a lot of information on the message.  
```
>>> msg.get_subject()
'Thanks! '
>>> msg.get_addresses('from')
'bob@example.com'
>>> msg.get_addresses('to')
