---
title: Declouding My Email Workflow, Part I
layout: post
date: 2016-12-04 17:56 EST

categories: mutt offlineimap gmail cloud mssmtp
---

# (in which I start the transition to UNIX-style mailing habits)

## Why?
Long story short: I would like to consolidate all of my data onto my local machine if I can afford to. If I have full control, I would ideally like to minimize the amount of my data that lives on servers out of my control.

My reasons are twofold:

One purpose is political as I grew up during a high level of optimism for the GNU Public License, Free Software[^1] as termed by Richard Stallman, the similar-but-not-identical concept of Open Software[^2]. I am also a bit disquieted by the recent role of data centralized by third-parties in daily life (esp. social media), which has been both an opportunity for great social good but has also been a source of great potential harm.

The other purpose is to salve my own ego. I have been a full-time software developer for almost ten years, now. I am more of a wilting flower developer, concerned with what I don't know rather than the more popular (and more dangerous, and more recommended, I'd say) form in which people celebrate their capabilities.
I remember being disoriented and unfamiliar with software and tools, so I scrabbled towards interfaces and technologies that didn't overwhelm me. This usually came in the form of cloud-based platforms or OS X platforms that streamlined functionality for ease of use.
I have valued this time in my life, because this streamlining has enabled me to focus and do a great many things inf my personal and professional development.

But I feel it is time to consider myself a veteran developer. It is time for me to go through my day not looking for the best workflows others have written, but to start thinking about what I can write to hasten my own workflow while keeping an eye out for possible improvements others have created.
As a developer I also now feel I have the capability to add what I need to a platform based primarily around open source unixes. Anything that doesn't exist, I am able to write. And the things that need writing are probably the things best written by me because they should be shaped to fit my needs exactly.

One of my current coworkers who most inspires me really pushed the benefits of shell scripts and aliases, saying that ideally every person's computers would be configured to fit its own exactly. I would like to see how far I could go with their mentality.

## How (overview)
**[ETA: BTW I've started a working glossary [primer](/email/2016/12/04/what_little_i_know_of_email_systems.html) on what I think I know about email systems. It may or may not be correct]**

Right now my mail is hosted in Google Mail. This makes it slightly different from the IMAP-based systems that are the standard way for mail to be hosted and handled nowadays, but it is compatible with IMAP and there are ways to work with it. I would like to keep and work with the labels and filters I already use, and we'll see if I move things locally as I get more comfortable. If you want to sync mail from Gmail, you will have to enable IMAP support.[^3]

I do not know if ultimately I want to download all of my mail locally, but it will come in handy should I choose to transfer all my mail to a different mail system.

I do not yet know what mail client I want to use locally. I use Emacs as my editor so an Emacs-based mail reader would make sense, but perhaps there are reasons to use something like Mutt[^4] or Alpine[^5]. I will start with Mutt because I don't know if using Emacs for both code and mail will break my mental workflow.

Here is the basic flow of how mail will work on my computer.
0. [OfflineIMAP](http://www.offlineimap.org) will fetch all new mail from Gmail, and will update Gmail with any label changes I have made.
1. [Mutt](http://www.mutt.org) will be the program I use to read mail, for now, because I found the most resources for this.
2. [mssmtp](http://msmtp.sourceforge.org) will be the program that sends new mail, via Gmail's SMTP server, off to people I want to communicate with.

Here is my overall outline of attack.
0. Explain some basic concepts which always seem to come up when dealing with unix-based mail systems.
1. Use the software `OfflineIMAP` to download all of my mail from work.
2. Configure `Mutt` to be a little bit Gmail friendly.
3. Configure a cross-platform way of sending mail, `mssmtp`.

I will admit that this would be a lot easier if I used something like mutt to do all of this for me. But there are technologies I want to look into that make this split necessary. Also, some of these allow me to change my day-to-day mail-reader without struggling with a lot of repeat configuration.

## A brief explanation of Gmail and IMAP
Historically, email was organized as if it was a file system; every email message was like a file, and was placed in a folder just like the files you have on your hard drive. Organization was dealt with by placing each mail into a category and coming up with some kind of organizational scheme. If you had a fancy mail system, you could have sub-folders!

Gmail's major selling feature is that it chose to eschew traditional folders for an interface in which you just *searched* for mail whenever you wanted to recall it. You could *label* mail if you want, and mail can have multiple labels if necessary, but generally all mail lives in a universal "all mail" bucket and have 0 or more labels if desired.

Gmail also doesn't immediately delete mail, but applies a "Trash" label so that you can undo that deletion if necessary. After 30 days it will automatically delete trashed email, or you can delete it permanently from within that label if you want. There is a similar folder for Spam.

Traditionally, a different server is used for sending mail than for storing or keeping mail. If you wanted to keep copies of all sent mail, you wold have to both sent it out via one system and place a copy of that mail in some "sent" folder. Gmail doesn't depend on client software manually; its sending infrastructure automatically copies any sent mail to a "Sent Mail" folder. If your mail client does this, you will wind up with multiple copies of the same email.

## OfflineIMAP

There are two major components to OfflineIMAP. One is a pair of python functions that translate Gmail's builtin folders into single-word lower-case mail. This is because as a keyboard-driven person I want to make my life a bit more convenient.

This code is located in the file `~/.offlineimap.py`

	import re

	""" Translate local folder names to the canonical Gmail labels. """
	def nametrans_local2gmail(folder):
		return {
			'drafts':    '[Gmail]/Drafts',
			'chats':     '[Gmail]/Chats',
			'flagged':   '[Gmail]/Starred',
			'important': '[Gmail]/Important',
			'spam':      '[Gmail]/Spam',
			'trash':     '[Gmail]/Trash',
			'sent':      '[Gmail]/Sent Mail',
			'archive':   '[Gmail]/All Mail',
			'inbox':     'INBOX'
		}.get(folder, folder)

	""" Translate gmail labels to local folder names """
	def nametrans_gmail2local(folder):
		return  re.sub('\[Gmail\]\/Drafts', 'drafts',
				re.sub('\[Gmail\]\/Chats', 'chats',
				re.sub('\[Gmail\]\/Starred', 'flagged',
				re.sub('\[Gmail\]\/Important', 'important',
				re.sub('\[Gmail\]\/Spam', 'spam',
				re.sub('\[Gmail\]\/Trash', 'trash',
				re.sub('\[Gmail\]\/Sent Mail', 'sent',
				re.sub('\[Gmail\]\/All Mail', 'archive',
				re.sub('INBOX', 'inbox', folder)))))))))

It is entirely possible that I could write both functions in forms of dictionaries or in regular expressions; I found some promising code snippets on the internet so that is what I used.

Generally configuring offlineimap requires configuring three sections ... something defining the remote source, the local store, and an "account" which ties them together. This configuration is found in `~/.offlineimaprc`

	[general]
	accounts = personal  # comma separated list of accounts to synchronize
	pythonfile = ~/.offlineimap.py  # If you specify this, you can leverage python functions
	maxsyncaccounts = 1  # If using autorefresh, this should equal the number of accounts defined

	[Account personal]
	localrepository = PersonalLocal  # Link this local mail store...
	remoterepository = PersonalGmail  # ... to this remote source
	synclabels = yes  # (see explanation below)
	autorefesh = 60  # Sync this account every 60 seconds

	[Repository PersonalLocal]
	type = GmailMaildir  # Gmail has some special handling, use its plugin
	localfolders = ~/mail/personal  # Where is this mail stored locally?
	nametrans = nametrans_local2gmail  # Use the mapping I defined above in .offlineimap.py

	[Repository PersonalGmail]
	type = Gmail # Again, there is a special plugin for gmail vs. regular imap
	remoteuser = gdcosta@gmail.com
	nametrans = nametrans_gmail2local  # Use the mapping I defined above in .offlineimap.py
	sslcacertfile = /usr/local/etc/openssl/cert.pem  # (see explanation below)
	oauth2_client_id = (gobbledygook)  # see explanation below))
	oauth2_client_secret = (gobbledygook)  # see explanation below)
	oauth2_refresh_token = (gobbledygook)  # see explanation below)
	# readonly = True  # Don't change remote source to reflect local store, useful for testing

### synclabels
Syncing all of my mail was slow enough, but the `synclabels` option is a gmail-specific thing that a) slows the process and b) messes up gmail a bit.

So there's a standard way of handling labels, there are a bunch of mail headers that tend to hold these keywords or labels. Gmail does not use it since it plays games with pretending to be folders. This option sets the mail headers (`X-Keywords`, or `X-Labels`, which might matter to someone but not to me) on every mail remotely. This is slow but it also **unfortunately** changes the date your mail was last modified. So right now, all my "All Mail" mail shows itself as having the date I first synchronized my mail with OfflineIMAP.

This might be worth it as I get more savvy, but I would personally keep it off until you decide you want to leverage it. I don't know how to yet.

### sslcacertfile
Gmail requires an SSL connection. At least on OS X, I had to generate this file myself by installing openssl. This file might also be different on different unix systems.

### Google OAuth
Normally I could just supply a username and password to my IMAP clients using the `remoteuser` and `remotepassword` options. For some reason Google Mail decides this isn't secure enough (even though Google has "application specific passwords which I thought were for this reason")

Instead I had to do this super nerdy thing[^6] to get mail working. I am not sure if it is better or worse from a security POV to have three different credentials, involving a rotating authentication.

## Mutt

I am currently using Mutt as my mail client because it is the most "familiar" (it's what I used in my university days) and my head works better when my mail client does not look like my editor.

I actually have _two_ accounts that I use regularly and I would like to show off how I navigate between both of them.

This is the contents of my `~/.muttrc`

	set mbox_type=Maildir  # OfflineIMAP uses the Maildir format for local storage
	set folder="~/mail/"  # As seen in my OfflineIMAP config, this is where all my mail is stored

source ~/.mutt/muttrc.mailboxes  # We will take about this later
set spoolfile = +personal/inbox  # This is my default inbox, we'll talk about naming formats later

set sort = threads  # I like to see my emails threaded
set sort_aux = reverse-last-date-received  # Within my threads, I like to see them in reverse temporal order

set mail_check = 60  # Refresh my view every 60 seconds
set timeout = 60  # Bail out of a prompt if 60 seconds go by and I don't do anything

folder-hook personal/* source ~/.mutt/muttrc.personal  # Set some things if I'm inside ~/mail/personal
folder-hook other/*     source ~/.mutt/muttrc.other  # Set some things if I'm looking at mailboxes for my other account

macro index,pager y "<save-message>=$my_prefix/archive<enter><enter>" "Archive"  # use the 'y' key to quickly move mail from the inbox to the archive. We'll talk about variables later.
macro index,pager gp "+personal/inbox" "Go to Personal Inbox"  # Use multi-key shortcuts for oft-accessed folders
macro index,pager go "+other/inbox" "Go to Other Inbox"

auto_view text/html                                      # view html automatically
alternative_order text/enriched text/plain text/html     # prefer plaintext over html (see caveats below)

#### mailbox name formats, variables
You'll notice a lot of forms like `+personal/inbox` being thrown around. This is basically a shortcut to say "starting with the folder referenced by `folder`, access these subfolders. You can use '=' or '+' as the shortcut prefix.

You'll notice in one place I have a `$my_prefix` defined. Any variable starting with `$my` is a user-defined variable, and I will show you later how I set that variable in a way that allows most of my shortcuts to leverage a common file folder scheme. The benefit is I can write the macro above once and not need to be written almost identically over and over.

#### Auto-generating mailboxes
Mutt is a bit silly in that normally one specifies, manually, _every single mailbox_.

OfflineIMAP has a solution for this ... it can generate a text snipped that lists every local mail folder it knows about. I configure mutt to source a file called `~/.mutt/muttrc.mailboxes` which is actually defined inside `~/.offlineimaprc` and is written whenever the latter program runs. This is what that section looks like:

	[mbnames]
	enabled = yes
	filename = ~/.mutt/muttrc.mailboxes
	header = "mailboxes "
	peritem = "+%(accountname)s/%(foldername)s"
	sep = " "
	footer = "\n"

I do not fully know the syntax, but I assume it takes the `header`, spits out something in the form of `peritem` via some builtin OfflineIMAP variables and python interpolation, separated by the `sep` value. It then ends with the `footer` value. Ultimately this generates something in mutt's config syntax that my `.muttrc` sources.

Other mail clients may not be as silly.

### Viewing HTML emails

I have unfortunately discovered that a lot of places now send all of their messages in HTML content without having an alternative plain-text form. Some of them add insult to injury by including a plain-text form that says "go look at the HTML version"

Mutt supports embedding HTML-rendered views, as you can see above. You will have to configure an additional file, though, called `~/.mailcap` which seems to be some kind of mime -> handler syntax. I am using the links console browser, I have also heard good things about w3m. This is what my file looks like:

	# for mutt to view html e-mails
	text/html;    links %s; nametemplate=%s.html
	text/html;    links -dump %s; nametemplate=%s.html; copiousoutput

I feel like only the second line is necessary, but these were the instructions I found, so it goes.

### Folder hooks.
As you can imagine, if I have two or more mail accounts I check, then each account will have its own inbox, it's own (Gmail) archive, its own trash-bin, etc... It would be embarrassing and annoying to have files get copied between two different accounts, wouldn't it?

The way to handle this is via **folder hooks**: As seen above, whenever we change into a qualifying directory, (in this case one of the format `<account>/<label>`) we source some account-specific configuration. In my case, for my personal mail it looks like the following:

	set spoolfile = +personal/inbox  # Set the inbox for this account
	set postponed = +personal/drafts  # Set the drafts folder for this account
	set mbox = +personal/archive  # If 'move' is set to on, all inbox email you ignore gets moved here.
	set trash = +personal/trash  # Set the trash folder for this account
	set move = no  # (optional) Don't move mail in my inbox if I ignore it.
	set record = +personal/sent # If not on gmail I would set this. I don't actually set it myself.
	set copy = no  # Don't move mail I create into any folder into the folder specified by $record.

	set sendmail="/usr/local/bin/msmtp -a personal"  # This is the program that actually sends mail.

	color status cyan default  # I use different status bar colours to indicate which account I am in.

	set my_prefix = "personal"  # You saw how I use this custom variable to define reusable hooks.
	set from = "myname@isp.com"  # This is the email address I put in the From: field.

I have one of these for every account I use. I suspect that built-in variables need to be set, while custom key bindings can take advantage of custom variables.

## msmtp

In my case I've chosen to use an external program to send mail: I believe mutt can do it itself, but for flexibility reasons I don't use the builtin functionality. Anyway, it is configured via the file `~/.msmptrc` (which has to be set to the unix permissions of 600 a.k.a. only the user can read and write to the file, because it has password information inside it which the program is paranoid about. If only mutt would be so paranoid itself >:)

	# Set default values (defined by the program)
	defaults

	# As with OfflineIMAP, Gmail demands use of SSL/TLS certificates
	auth on
	tls on
	tls_trust_file /usr/local/etc/openssl/cert.pem

	# Personal
	account personal
	host smtp.gmail.com
	port 587
	from myname@gmail.com
	user myname
	password <applicationspecificpassword>


	# Set a default account
	account default: personal

You can copy that personal stanza for any other gmail account you need, and I'm sure it is very similar for non-gmail accounts as well.

## Conclusion

Anyway, this is enough to give me a minimum viable workflow for email. There are additional things I'd like to try out which necessitate the configuration and architectural choices I'm made (esp. downloading local mail instead of using Gmail's servers directly) which I will explain once I figure out how things work.

See you later.

[^1]: <https://www.gnu.org/philosophy/free-sw.en.html>
[^2]: <https://opensource.com/resources/what-open-source>
[^3]: <https://support.google.com/mail/answer/7126229?hl=en>
[^4]: <http://mutt.org/>
[^5]: <https://www.washington.edu/alpine/>
[^6]: <https://github.com/OfflineIMAP/offlineimap/blob/master/offlineimap.conf#L829>
