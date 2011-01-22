---
layout: post
title: Lamson Project
tags: email smtp server lamson python
---

If you are dealing with mail systems you will probably know [qmail](http://www.qmail.org/) or [Postfix](www.postfix.org). These are *MTA*s (Mail Transfer Agents) they get and forward mails. It is simply a router for your mails. Usually is the *MTA* playing the biggest part in the email system. He gets a mail from an *MUA* (Mail User Agent) and transports it over multiple *MTA*s to the *MDA* (Mail Delivery Agent) which will store the mail in some way. The *MUA* of the recipient will then pull the Message using IMAP/POP except if it is stored locally.

    Sender
     MUA       MDA ----> Store (Messages)
      |         ^          ^
      |  SMTP   |          |    POP/IMAP
      V         |          |
     MTA ----> MTA       Server <----> MUA
                                     Reciever

In todays systems these components are mostly tied together very closely. For example, the *MTA*, *MDA* and *IMAP/POP* server have to know if the user exists, maybe username and password, store location, quota and many more. These information are in most cases distributed using a *LDAP* or a *SQL* database. In smaller environments maybe text files are appropriate.

If you have a bigger setup, where you check for spam and virus and using a web frontend you may end up with architecture like this:
     
    Sender     MTA ---------------------> MDA ----> Store (Messages)
     MUA        ^                          ^
      |         |                          |    POP/IMAP
      |    S   MTA <----> Spam-            |
      V    M    ^         Assassin       Server <----> MUA (Reciever)
     MTA   T    |                          ^
      |    P   MTA <----> ClamAV           |    POP/IMAP
      |         ^                          |
      V         |                     Web frontend
     MTA ----> MTA

Then if you add mailing lists and much more the whole thing gets pretty complex and somewhat slow. The Messages have to be parsed at each point in the chain. You can easily scale the whole system because everything is working on *SMTP*, *IMAP*, *POP* and *DNS* but you pay for it.

One project which aims to make the whole *MTA* stuff a little bit simpler is the [lamson project](http://lamsonproject.org). It is written by Zed Shaw, the author of the fast and robust web server [mongrel](http://mongrel.rubyforge.org/). Lamson is written in *python* an well documented (even the source code). I have read the whole source code and it is fun to do...

Lamson has a few interesting design principles, that want to help you making *MTA*s more reliable, intelligent and flexible:

- It inherits a project structure comparing to *Ruby on Rails*, *Django* and others. Including configuration, routing and **test**. You can test your routes! I love this idea!
- The routing is done through python using a *FSM* (Finite State Machine), which makes it a very powerful queuing system.
- Creating custom queues is very simple. Just implement the three methods: get, set, clear.
- The whole process (with spam/virus protection) can be configured/implemented using the internal routing system. Therefore the parsing of the *SMTP* mail can be minified, which improves performance.
- A simple setup is setup in a few minutes.
- It supports the maildir format out of the box.
- It sends either unicode (*UTF-8*) or ascii mails and makes assumptions on what is a good mail. Every broken mail will be tried to fix otherwise rejected.
- It is easy to customize and extend. For example if you have to support email distribution lists you can easily request a database or something similar and create new mails right within the queue.

If this sounds interesting for you try it out. For me it is a very good project which opens my mind for new ways to think about mail and *MTA*s. 
