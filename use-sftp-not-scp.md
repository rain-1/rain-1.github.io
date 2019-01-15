# Recommendation: sftp for copying files between machines.

`sftp` is a better tool than `scp` for copying files between machines. It doesn't (mis)interpret filenames and has a more secure design overall. For example it has a built in chroot system.

Disadvantages of sftp: "SCP is usually much faster than SFTP at transferring files, especially on high latency networks". From [https://superuser.com/questions/134901/whats-the-difference-between-scp-and-sftp]. I timed a copy across my local network of a 104MB file - sftp: 37 seconds. scp: 34 seconds.

A discussion about scp vs sftp from curl dev. [https://ec.haxx.se/usingcurl-scpsftp.html]

## Why shouldn't I use scp?

It interprets the filename as a shell command *on the remote host* before trying to copy that file. e.g. ``scp pi:'`touch wow`' .`` creates a file on the 'pi' computer.

Some people have called this a "feature" rather than a bug and discussed how you can do things like `scp pi:/db/backup-$(date +%s)`. I don't really buy this, do you know anybody that makes use of this feature on purpose?

So you have to do a bit of awkward/difficult shell sorcery with `"` and `'` to copy filenames with spaces or sigils. It can be complex when there are 4 different possible levels of quoting and interpretation in a single command (local shell processing, command line options processing, parsing of inputs, remote shell processing).

Also there was some interesting research finding even more nasty bugs with scp: [https://blog.f-secure.com/weak-scp-security-vulnerabilities/]. (Security note: This is very minor in terms of security, it can only be exploited if you ssh into a malicious host. For that to happen you would have to accept a bad fingerprint. On the other hand it illustrates the bad design of scp). It's such an old protocol that it's really outdated in the way it's designed. They are sort of forced to keep the unpleasant features in because changing anything would break millions of scripts.

## What about rsync

rsync might have some nice theory to the transfer algorithm: [https://rsync.samba.org/tech_report/] but the actual implementation leaves a lot to be desired. It's got a long long history of quite serious security vulnerabilities involving pathnames. I don't recommend using this tool at all.

## How do I use sftp

### copy a file from my local computer to my remote computer

```
echo put 'sheep.txt' | sftp pi
echo put 'sheep.txt' here\\\'s-sheep.txt | sftp pi
echo put 'sheep.txt' "'heres sheep.txt'" | sftp pi
```

Note: You do need to escape the filenames in your 'put' commands.

### copy a file from my remote computer to my local computer

```
sftp pi:file-i-want.txt
```

or

```
sftp pi:file-i-want.txt mine-now.txt
```

note: this will go into interactive mode if the filename actualy designates a directory on the remote.

### copy a folder from my local computer to my remote computer

```
echo get -r "'the folder'" | sftp pi
```

or

```
echo get -r "'your folder'" "'my folder'" | sftp pi
```

Note: does not follow symbolic links

### copy a folder from my remote computer to my local computer


```
echo put -r site-backup/ | sftp pi
```

Note: does not follow symbolic links

### what commands does sftp let me use?

```
sftp> help
Available commands:
bye                                Quit sftp
cd path                            Change remote directory to 'path'
chgrp grp path                     Change group of file 'path' to 'grp'
chmod mode path                    Change permissions of file 'path' to 'mode'
chown own path                     Change owner of file 'path' to 'own'
df [-hi] [path]                    Display statistics for current directory or
                                   filesystem containing 'path'
exit                               Quit sftp
get [-afPpRr] remote [local]       Download file
reget [-fPpRr] remote [local]      Resume download file
reput [-fPpRr] [local] remote      Resume upload file
help                               Display this help text
lcd path                           Change local directory to 'path'
lls [ls-options [path]]            Display local directory listing
lmkdir path                        Create local directory
ln [-s] oldpath newpath            Link remote file (-s for symlink)
lpwd                               Print local working directory
ls [-1afhlnrSt] [path]             Display remote directory listing
lumask umask                       Set local umask to 'umask'
mkdir path                         Create remote directory
progress                           Toggle display of progress meter
put [-afPpRr] local [remote]       Upload file
pwd                                Display remote working directory
quit                               Quit sftp
rename oldpath newpath             Rename remote file
rm path                            Delete remote file
rmdir path                         Remove remote directory
symlink oldpath newpath            Symlink remote file
version                            Show SFTP version
!command                           Execute 'command' in local shell
!                                  Escape to local shell
?                                  Synonym for help
```

### Alternatively

```
tar cf - . | ssh host 'cd path && tar xf -'
```

cheers Taylor.
