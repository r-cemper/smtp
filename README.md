## Mail sending for beginners   
Sending information by mail is one of the oldest technologies  
in networks. The standard for mail transfer was declared in 1980.  
SMTP stands for **S**imple **M**ail **T**ransfer **P**rotocol.   
SMTP has been supported in IRIS since the beginning and also before.  
Over time, various features have evolved, especially related to  
security, encoding, authentication, multipart messages, ...   
The [related documentation](https://docs.intersystems.com/results.html?docs%5Bquery%5D=mail) is excellent in all details. You find a rich set of   
Classes, Methods, Utilities and Examples in %Net.* class documentation. 
    
This example shows a very simple starting point into the subject.   
If you send an email, you need an SMTP server that receives it.  
It could be some external server of your choice, but again, IRIS is      
well prepared to serve you. Interoperability provides all you need.   
So this package provides you with a minimal-sized Production for testing.    
Instead of a real Production, the service writes the input into a global   
that you can check immediately.  
So the demo can run in a *single* instance for sending and receiving mail.    
A slim test routine for all required steps is provided.   

### Prerequisites
Make sure you have [git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git) and [Docker desktop](https://www.docker.com/products/docker-desktop) installed.

### Installation 
Clone/git pull the repo into any local directory:   
```
https://github.com/rcemper/smtp.git
```
Build and run the IRIS container  
from the download directory with your project and check startup:    
```
docker-compose up -d   && docker-compose logs -f
```
### Testing   
Start an IRIS session in namespace USER
```
docker-compose exec iris iris session iris
```
Now load the test utility
```
USER>ZLOAD smtp PRINT
mail     ;
         s %mail=##class(%Net.MailMessage).%New()
         s %mail.From="iris@home.at"
         s %mail.Sender="iris@home.at"
         s %mail.Subject="Test from ISOS"
         d %mail.TextData.WriteLine("demo test mail from ISOS")
         d %mail.TextData.WriteLine("more text from ISOS")
         s %mail.MessageSize=%mail.TextData.SizeGet()
         d %mail.To.Insert("irisowner")
         ;;
         s %smtp=##class(%Net.SMTP).%New()
         s %smtp.smtpserver="localhost"
         s %smtp.port=30025
         q
start
         w ##class(Ens.Director).StartProduction("mail.Receive")
         q
send
         s sc=%smtp.Send(%mail)
         zw sc
         i 'sc d $system.OBJ.DisplayError(sc)
         q
view
         zw ^txt
         q
stop
         w ##class(Ens.Director).StopProduction()
         q
```
From command line prompt  
```
USER>
```
you have these 5 options to type in   
- **do mail**  ; is composing a simple mail with 2 lines   
for more text, add content to character stream  %mail.TextData    
and don't forget to also adjust the text size   
- **do start**  ; is starting the SMTP server production
```
17:27:37.285:Ens.Director: Production 'mail.Receive' starting...
17:27:37.332:Ens.Director: Production 'mail.Receive' started.1
```
- **do send**   ; is sending your mail to server
```
sc=1
```   
if an error is returned, code and details are displayed   
```
sc="0 "_$lb($lb(6031,,,,,,,,,$lb(,"USER",$lb("e^Send+42^%Net.SMTP.1^2","e^send+1^smtp^1","d^^smtp^0"))))
ERROR #6031: Unable to open TCP/IP connection.
```
- **do view**   ; shows the lines received by SMTP Production
```
^txt=1
^txt(1,1)="EHLO 5a950947449d"
^txt(1,2)="MAIL FROM: <iris@home.at>"
^txt(1,3)="RCPT TO: <irisowner>"
^txt(1,4)="DATA"
^txt(1,5)="Date: Sat, 15 Aug 2026 11:02:07 UT"
^txt(1,6)="From: iris@home.at"
^txt(1,7)="Subject: Test from ISOS"
^txt(1,8)="Sender: iris@home.at"
^txt(1,9)="To: irisowner"
^txt(1,10)="MIME-Version: 1.0"
^txt(1,11)="Content-Type: text/plain; charset=""us-ascii"""
^txt(1,12)="Content-Transfer-Encoding: quoted-printable"
^txt(1,13)=""
^txt(1,14)="demo test mail from ISOS=0Amore text from ISOS=0A"
^txt(1,15)=""
^txt(1,16)="."
^txt(1,17)="QUIT"
```
- **do stop**   ; stops the SMTP Server Production
```
17:45:48.370:Ens.Director: StopProduction initiated.
17:45:48.376:Ens.Director: No Production is running (Stopped)1
```

Access to [System Management Portal](http://localhost:52773/csp/sys/%25CSP.Portal.Home.zen?$NAMESPACE=USER) 
and [Interoperability Portal](http://localhost:52773/csp/user/EnsPortal.ProductionConfig.zen?$NAMESPACE=USER&PRODUCTION=mail.Receive)

[Article](https://community.intersystems.com/post/mail-sending-beginners)
