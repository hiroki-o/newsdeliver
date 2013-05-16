NewsDeliver, A Simple NNTP to Email Gateway
====
Copyright (C) 2002-2005 by Matt Roper <matt@mattrope.com>


Introduction
------------
NewsDeliver is intended for people who need to closely monitor a newsgroup
and want to get a notification whenever a new message is posted.  NewsDeliver
is a simple solution to a simple problem:  put NewsDeliver in a cron job that
runs every 10 minutes or so and it will email you a copy of any new messages
that get posted on the newsgroup.  If cron isn't available on your system,
NewsDeliver can also run as a daemon that will wake up periodically and check
for new messages.

NewsDeliver isn't meant to be an earth-shaking program; it's just a simple
Python script that I wrote so that I wouldn't miss any announcements from
professors on my class newsgroups.  Other college students will probably find
it useful for the same purpose.

Why did I write NewsDeliver in Python?  NewsDeliver is a very simple program
that probably could have been coded in just about any language in five minutes.
I happened to be teaching myself Python at the time and when my Internet
searches for a program like NewsDeliver failed, I decided to use that as an
excuse to actually write one in Python.  I originally wrote NewsDeliver for my
own personal use, but a couple of other people have expressed interest in it,
so I have decided to release it under the GPL just in case anyone else would
find it useful.  If you find NewsDeliver useful or feel that I should add any
features, please drop me an email and let me know.


Requirements
------------
Python is required by NewsDeliver (it has been tested on both Python 1.5 and
Python 2.x).  NewsDeliver is intended for use on *nix systems although I have
had one report of success from a Windows user running it under Cygwin.

The one feature that is known to require a relatively up-to-date version of
Python (i.e., 2.2 or later) is SMTP authentication.  If you must provide a
username and password to send mail through your SMTP server, then your Python
installation must be at least version 2.2.


Installation
------------
NewsDeliver installation is simple.  Just copy the 'newsdeliver' script to a
directory that is in the path of all users that need to use it (e.g.
/usr/local/bin).  Multiple users can all use NewsDeliver on the same system;
configuration files and "last seen message" lists are kept in each user's home
directory to prevent conflicts.  If only a single user on a system needs to run
NewsDeliver, he can keep the 'newsdeliver' script somewhere under his home
directory instead of placing it in a global bin directory.

WARNING:  **DO NOT** install NewsDeliver as setuid!  The configuration files
    for NewsDeliver are interpreted directly by the Python interpreter, so
    NewsDeliver should always run with the privileges of the current user.


Setup
-----
In order to start using NewsDeliver, run the command

    newsdeliver --newconf

This will create a new configuration file for you in ~/.newsdeliverrc.  Some of
the configuration values will be auto-detected for you, but you'll need to edit
the file to set some of them (such as the NNTP server that your newsgroups are
on).

Now that your configuration file is set up, you're ready to tell NewsDeliver
what groups you're interested in monitoring.  For each newsgroup you want to
get email copies of, run the command

    newsdeliver --addgrp my.newsgroup.name

(Replace "my.newsgroup.name" with the name of the actual newsgroup.)
NewsDeliver will assume that you've seen all of the messages already
posted to that newsgroup and will start monitoring from this time forward.

Finally, if you intend to run NewsDeliver via cron (instead of as a daemon)
you need to add an entry to your crontab file so that it runs periodically.
For example, if you wanted NewsDeliver to run every 20 minutes, you could
run the following commands to add the necessary cronjob:

    crontab -l > tempcrontab
    echo "0,20,40 * * * * /path/to/newsdeliver > /dev/null 2>&1" >> tempcrontab
    crontab tempcrontab
    rm tempcrontab

If you want NewsDeliver to run more often than every 20 minutes, add other
minute numbers to the 0,20,40 list.  Also be sure to change the
/path/to/newsdeliver to point to the location where you actually installed the
newsdeliver script.

If you don't want to use cron, simply start NewsDeliver with the following
command:

    newsdeliver --daemon &

and it will sleep in the background between checks.


Usage
-----

    newsdeliver [--verbose]
        Checks for new posts in all of the newsgroups that you're monitoring.
        If the --verbose switch is present, lots of debugging information will
        be sent to stdout.

    newsdeliver [--verbose] newsgroup1 newsgroup2 newsgroup3 ...
        Checks for new posts only in the listed newsgroups.  If the --verbose
        switch is present, lots of debugging information will be sent to stdout.

    newsdeliver [--verbose] --daemon &
        Starts NewsDeliver in daemon mode; NewsDeliver will sleep for the
        amount of time specified by the DAEMONSLEEP config option
        (default is 60 seconds) after each run and will only terminate if
        serious errors are encountered.

    newsdeliver --newconf
        Generates a new configuration file for NewsDeliver.  This only needs to
        be run once unless your configuration somehow gets corrupted.

    newsdeliver --addgrp grpname
        Starts monitoring "grpname" for new posts.

    newsdeliver --remgrp grpname
        Stops monitoring "grpname" for new posts.

    newsdeliver --list
        Displays a list of all newsgroups being monitored, along with any
        subject filters that may be in place.
    
    newsdeliver --catchup grpname
        Marks all messages in "grpname" as "seen" so that they will not be
        delivered.

    newsdeliver --catchup all
        Marks all messages in all monitored newsgroups as seen so that they
        will not be delivered.

    newsdeliver --filter grpname=filter
        Tells NewsDeliver to only retrieve messages whose subject lines
        contain the string 'filter'.  To remove the filter from a group,
        just set it to an empty string (e.g. "--filter foo.bar=")

    newsdeliver --setaddr grpname=addr
        Tells NewsDeliver to send messages from the specified newsgroup
        to an alternate email address (this overrides the global email
        address setting).

    newsdeliver --subjprefix grpname=prefixstr
        Tells NewsDeliver to prepend the value of prefixstr to the subject
        line of the emails it sends for grpname instead of using the global
        default value from the config file.

    newsdeliver --help
        Displays a list of legal command line parameters.


Dealing With Binary Posts
-------------------------
As of version 1.2, NewsDeliver now tries to detect articles that are binary
files.  When it encounters such an article, it will proceed according to
the BINARIES variable in the user's config file:

    IGNORE -- Silently ignore (do not send) this message.
    NOTIFY -- Send a short text message indicating that a binary post
              was encountered.  The user can use his regular newsreader
              to check it out.
    SEND   -- Send the post just like a normal text article.

Binary detection is currently limited to yEnc and uuEncode formats.

The Spam Problem
----------------
Spam messages can cause problems for NewsDeliver in some situations.  If the
mail server that newsgroup articles are delivered to has SMTP-time mail
filtering, it's possible for spammy newsgroup articles that NewsDeliver tries
to deliver to be rejected by the SMTP server.  Unfortunately, it isn't possible
for NewsDeliver to determine the cause of the rejection -- the failure might be
caused by a mail filter, or it might just be a symptom of a temporary mail
server glitch.  In the first case, the problematic message's delivery failure
should be ignored and NewsDeliver should move on to later messages.  But if the
failure is just a temporary glitch, then moving on will simply cause valid
articles to not be delivered.

NewsDeliver provides a MAILERROR option in the user config file that can be
used to select the appropriate behaviour for this situation.  If your mail
server does not perform SMTP-time mail filtering and rejection (most don't
unless the administrator has specifically configured them that way), you should
set the MAILERROR setting to "FAIL" so that valid messages will not be missed
if there is a temporary mail server or network glitch.  On the other hand, if
you do have SMTP-time mail filtering and if you monitor newsgroups that could
potentially receive spammy messages, you should set MAILERROR to "IGNORE" so
that a single spammy message will not terminate the NewsDeliver process and
prevent delivery of subsequent good news articles.

Group File Format
-----------------
Rather than using the command line switches to alter the list of monitored
newsgroups, it is safe (and easy) to modify the newsgroup database directly.
The format is very similar to the INI file format.  An example group file
might look like the following:

    <NewsDeliver 1.3>

    [news.group.name]
    lastseen=402
    addr=alternate.email@company.com

    [another.news.group]
    lastseen=100
    filter=foobar
    subjprefix=[NEWS: %g]

The important thing to note here is that the first line of the file must
start with the string "<NewsDeliver ".  After that, information about
each newsgroup is listed in a block -- the name of the group appears
between brackets and attributes of the group are key/value pairs separated
by "=".  Every group must have a "lastseen" attribute, but all other attributes
are optional.  Possible keys that can be set are:

    addr
        An alternate email address that messages from this group should be 
        mailed to.  Overrides the EMAILTO setting from .newsdeliverrc.
        This attribute can be set from the command line with the --setaddr
        switch.

    filter
        Only messages whose subjects match the specified filter will be
        emailed.  This attribute can be set from the command line with the
        --filter switch.

    subjprefix
        A string that will be prepended to the subject line of all messages 
        before they are sent; this overrides the global SUBJPREFIX value
        set in .newsdeliverrc.  If the string "%g" is found inside the
        subject prefix, it will be expanded to the name of the newsgroup.
        This attribute can be set from the command line with the --subjprefix
        option.

    uploadto  (experimental)
        The name of a news server that we should try to upload articles
        to instead of emailing them.  This uses the NNTP IHAVE command.
        I have no server that I can test this feature on, so it may or
        may not work properly.

    smtpfrom
        By default, the value of EMAILTO from .newsdeliverrc will be used
        as the sender address when communicating with the SMTP server
        (i.e., it will be used on the 'MAIL From:' line of the transaction).
        Since this value isn't actually visible as part of the email in
        most mail clients, this value works fine and ensures that we have
        a valid address to provide (and sometimes helps work past spam
        filters).  However some tools (such as the Unix 'from' command)
        do display this sender address so it may be desirable to override
        this value with a different address.  This option allows the
        sender address to be changed for a single newsgroup, and the
        SMTPFROM option in ~/.newsdeliverrc allows it to be changed
        globally.

License
-------
NewsDeliver is distributed under the GNU General Public License.  See the
file LICENSE from the distribution for details.

