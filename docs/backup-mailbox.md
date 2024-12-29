Backup and migrate mailbox
==========================
I'm using AWS WorkMail and would like to backup my mailbox locally.

*Note: Tested on Linux Mint 21.3, circa Dec 2024.*

Install the `isync` package:

```bash
# may need to apt-get update first
sudo apt-get install isync
which mbsync
```

Now create and edit `~/.mbsyncrc`. Modify `User`, `Pass` and `Host` as appropriate. **Optionally**, modify 
`Path` and `Inbox` to match your email user name.

```
# ensures that dates of messages will be set correctly
CopyArrivalDate yes

IMAPAccount src
Host imap.mail.us-east-1.awsapps.com
Port 993
User john@example.com
Pass "MySecret"
SSLType IMAPS

IMAPStore src-remote
Account src

MaildirStore dst-local
Path ~/mail/john/
Inbox ~/mail/john/Inbox

# mkdir -p mail/journal && mbsync backup-to-local
Channel backup-to-local
Far :src-remote:
Near :dst-local:
Patterns *
Create Near
SyncState *
Sync Pull
```

You can start the download now. This can take quite a while depending on your mailbox size.

```bash
mkdir -p mail/john
mbsync backup-to-local
```

You can run `mbsync backup-to-local` multiple times and it will only sync the difference.

After full download, simply archive the entire directory:

```bash
find ~/mail/john -type f | wc -l
du -hs ~/mail/john
tar caf john.tar.gz -C ./mail/john .
ls john.tar.gz
```


Restore
-------
This assumes that the remote email account is empty.

Repopulate mail directory from archive:

```bash
mkdir -p ~/mail/john
tar xf john.tar.gz -C ./mail/john
```

Modify `~/.mbsyncrc`:

```
# ensures that dates of messages will be set correctly
CopyArrivalDate yes

IMAPAccount dst
Host imap.mail.us-east-1.awsapps.com
Port 993
User john@example.com
Pass "MySecret"
SSLType IMAPS

IMAPStore dst-remote
Account dst

MaildirStore src-local
Path ~/mail/john/
Inbox ~/mail/john/Inbox

# mbsync export-to-remote
Channel export-to-remote
Far :dst-remote:
Near :src-local:
Patterns *
Create Far
SyncState *
Sync Push
```

Remove metadata files specific to `mbsync`:

```bash
find ~/mail/journal -type f -name '.mbsyncstate' -delete
find ~/mail/journal -type f -name '.uidvalidity' -delete
```

Rename the email files to strip off `,U=##`, for example: `1735123456.135246_2.ubuntu,U=2:2,S` should 
be renamed to `1735123456.135246_2.ubuntu:2,S`:

```bash
find . -type f -name '*U=*' | { while read f; do mv "$f" "$(echo $f | sed -e 's/,U=[0-9]*//')"; done }
```

You can now push local to remote:

```bash
mbsync export-to-remote
```


References
----------
Further references:

  - https://isync.sourceforge.io/mbsync.html
  - https://aaronweb.net/blog/2014/11/migrating-mail-between-imap-servers-using-mbsync/
  - https://stackoverflow.com/questions/73248641/mbsync-near-side-box-unable-to-open
    - make sure `Path` ends with a `/`!
  - https://docs.aws.amazon.com/workmail/latest/userguide/using_IMAP.html
