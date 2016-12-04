---
title: What Little I Know of Email Systems
layout: post
date: 2016-12-04 17:55 EST

categories: email
---

As someone who has spent much of his life using Gmail or Thunderbord or some other kind of all-in-one email client, I don't really think much about how my email gets to me. When I have tried, it's come in the form of heavily complex documentation from mail server softwaare (like Postfix or Sendmail) which claims that email is best handled by a number of moving parts but then provides all of those parts in one or two messy mega-pieces of software.

So I am going to try listing off terminology as best I understand to form a working glossary. I'm probably wrong. Here goes.

*. **SMTP - Simple Mail Transfer Protocol** - The protocol for _sending_ mail to other people
*. **POP3 - Post-Office Protocol version 3** - A protocol for _fetching_ mail from a server. The idea is it only holds new mail, you download it and probably wipe it from the server. It doesn't have any sense of organization of structure.
*. **IMAP4 - Internet Mailbox Access Protocol** - Another protocol for reading stored mail on a server. Unlike POP3 it can be used to keep mail stored on a remote server, and it supports hierarchies like folders and subfolders

In a modern world where we tend to think of mail servers as being like ... Gmail or your office mail system and your computer (or phone or web browser) as having a mail client like Outlook or Thunderbird, you might think that there are just two things, mail servers and mail clients.

This isn't the case on traditional unix systems, which broke this functionally down even further. Let's assume someone sends a mail to you:

1. A **MTA - Mail Transfer Agent** gets that email, and decides if it can handle it itself or forwards it to a server that is more appropiate. For example you send an email to your mailserver's MTA, but it's destined for someone on a totally different mail server. Your MTA will then forward the email over to that service's MTA.
2. A **MDA - Mail Delivery Agent** recieves the mail from a local MTA, might do things like spam-filter it, and then stores it locally for the user.
3. A **MUA - Mail User Agent** is the program that you actually use to read and write mails. It might be a fancy GUI or it might be commandline tools. This of course works if you are running it on the same system where your mail is stored. But what if you're using it at home?
4. A **MRA - Mail Retrieval Agent** fetches mail from a remote server, either via the aforementioned POP3 or IMAP protocols
5. A **MSA - Mail Sending Agent** takes newly composed mail and sends it to an MTA so it can actually get to where it needs to go.

As you can imagine, something complex like Outlook or Thunderbird winds up being an MUA, MRA and MSA.

#### How is mail stored on a filesystem?

If you think about unix-based[^Windows or other OSes might do something very different, for all I know Microsoft Exchange stores everything in a database] systems, it's probably going to be stored as a bunch of files in a directory you are allowed to access (or maybe not ... maybe you are forced to access everything via IMAP) Way back when you either logged into a remote server to read your mail, or you fetched it ... modern systems have sort of blurred the line, and webmail has basically destroyed the line completely.

At the very least if you're using a computer at home, your mail app has to store all your mail somehow. Again, proprietary and complex pieces of software like Thunderbird or Outlook might use a private database, but the standard forms are one of the following:

* **mbox**: The oldest format, this is basically one file with all of your email appended. It's convenient and easy to process, but can't have multiple programs reading it and manipulating it without corrupting all the data.
* **Maildir**: Maildir splits out each email into one file, but then stores it in a complex way that's easy for programs to understand but not humans.

There are other options, but they are far less common. The above two are the de-factor standards. Thunderbird I believe actually uses mbox internally.

#### What about gmail?
Gmail can pretend to be an IMAP or POP server and work with those technologies' limitations, but it tweaks things a little bit. Obviously it's proprietary so I have no idea what goes on behind the scenes. But unlike IMAP which assumes a typical filesystem interface (folders containing messages, so a message can only be in one folder), gmail treats everything like labels so that you can have multiple labels on any mail. *ALL** mail has the 'All Mail' label attached, because Gmail is built around the idea that everything is in one giant indexable/searchable bucket.
Gmail also does some magic to intercept all mail you send and stick the 'Sent Mail' label on it; because sending mail is a different process than storing/retrieving it, this does not happen by default on standard SMTP/IMAP combinations. (Back when Gmail was the only service with "unlimited" email, this was a game-changer; most systems didn't expect you to save all of your email because disk space was finite)

#### Spam filtering?
Email is a notoriously insecure process, one of the easiest ways to get hacked or become a conduit for nasties spreading viruses and spam is to try to run your own mail-server. Email, one of the earliest mechanisms on the internet, was very trusting and assumed that everyone had good intentions. It is only relatively recently that email systems have had a wealth of tooling (spam filters, virus scanners, policies for not forwarding emails on behalf of people who aren't confirmed users of your service, certificates that you are in fact a "legitimate" mail server) that I don't really want to get into because I don't understand it at all.

Anyway, if you want to get into the magical world of pretending it is the 80s/90s and running all of your email on a unix system, hopefully I've made things slightly less confusing.
