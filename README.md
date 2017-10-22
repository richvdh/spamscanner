spamscanner
===========

This script looks for emails which have been moved into or out of a 'spam'
folder on an Cyrus mailserver, and will feed them into spamassassin to train
its bayesian filters.

Operation
---------

The script assumes that your mail delivery system uses Spamassassin to check
for spam at delivery time, and files spam into a folder named accordingly. It
also assumes that Spamassassin adds a message header to any detected spam (by
default, Spamassassin will add `X-Spam-Flag: YES`).

On each run of the script, it looks for messages which have been manually added
to the spam folder (i.e. those which do not have the "spam" header), and
reports them to Spamassasin as spam. It also looks for messages which have been
manually moved from the "spam" folder to another folder, and reports them as
ham.

Prerequisites
-------------

* The script expects to authenticate to the IMAP server as an admin user. By
  default, it authenticates as the `spamscanner` user. You should create a user
  on the IMAP server and add it to the `imap_admins` list in `/etc/imapd.conf`.

  Once you have done this, `lm` under `cyradmin` (using the spamscanner user
  credentials) should give a list of all mailboxes on the system, with user
  mailboxes prefixed `user.<username>`.

* The script will connect to the spamd daemon, which must be (a) running, and
  (b) configured to allow spam/ham training. To do the latter, set
  `--allow-tell` on the `spamd` commandline. (For Debian, this can be done by
  editing the `OPTIONS` setting in `/etc/default/spamassassin`.)

  (Note that this will allow anybody who can connect to spamd to train its
  Baysian filter. You should probably ensure that shell access to the server
  hosting Spamassassin is restricted.)

* The script its state in a file. Its location can be changed via the
  `--statefile` parameter, but by default it is
  `/var/lib/spamscanner/spamscanner.pickle`. You should create
  `/var/lib/spamscanner`.

Installation
------------

The script is configurable via a number of commandline arguments; for more
information, run the script with ``--help``. In general though, the defaults
should be largely sufficient, except the imap password.

On first run, the script should be run with the `--init` argument. This will
cause it to initialise the state file, and scan all folders for misfiled ham or
spam. It is suggested that you use the `-dd` debug level to give some
information on what it is doing:

    spamscanner -p <password> -dd --init

Once this completes successfully, you should set up a cronjob to run the script
periodically (hourly or daily). Modulo any local configuration options, thhe
command to run is simply:

    spamscanner -p <password>
