###### тэги: [[ctf]]
###### дата: 22.11.2025
---
## bandit1 → bandit2
home directory contains file “-”, which can’t be opened by `cat file.txt` command, but opens with `cat < file.txt`.  this happens because in first case `cat` opens and reads file on its own, whereas in second case we redirect contents of file.txt to stdin and passing it to `cat` which outputs text in terminal.

## bandit2 → bandit3
file with spaces in its name is handled by escaping them `\ `. 

## bandit3 → bandit4
file is simply in another directory.

## bandit4 → bandit5
multiple files in directory, script to read them all:
```bash
for filename in ./*; do
	cat $filename;
	printf "\n";
done
```

## bandit5 → bandit6
multiple subdirectories in directory, we need to read human-readable file with size of 1033 bytes:
```bash
for directory in $(find . -maxdepth 1 -mindepth 1 -type d); do
    (printf "$directory\n"; ls -la "$directory") | grep 1033 -A 5 -B 10;
done;
```

## bandit6 → bandit7
file is somewhere on the server with specific properties, command to find it:
```bash
find / -group bandit6 -user bandit7 2>/dev/null
```

## bandit7 → bandit8
the password for the next level is stored in the file data.txt next to the word millionth.
```bash
cat data.txt | grep millionth
```

## bandit8 → bandit9
the password for the next level is stored in the file data.txt and is the only line of text that occurs only once.
```bash
sort data.txt | uniq -u
```

## bandit9 → bandit10
the password for the next level is stored in the file data.txt in one of the few human-readable strings, preceded by several ‘=’ characters.
```bash
grep '==' -a data.txt
```
## bandit10 → bandit11
the password for the next level is stored in the file data.txt, which contains base64 encoded data.
```bash
base64 -d data.txt
```
## bandit11 → bandit12
the password for the next level is stored in the file data.txt, where all lowercase (a-z) and uppercase (a-z) letters have been rotated by 13 positions.
```bash
cat data.txt | tr [a-za-z] [n-za-mn-za-m]
```
## bandit12 → bandit13
the password for the next level is stored in the file data.txt, which is a hexdump of a file that has been repeatedly compressed.
``` bash
xxd -r data.txt > data.bin # hexdump -> binary
```
then we check hexdump of data.bin and look at first bytes to determine file signature. if hexdump starts with:
- `1f 8b` – gzip (`gzip -d data.gz`)
- `42 5a 68` – bzip2 (`bzip2 -d data.bz2`)
- `6461 7461 382e 6269 6e00`  – tar (`tar -xvf data.tar`)

## bandit13 → bandit14
“the password for the next level is stored in `/etc/bandit_pass/bandit14` and can only be read by user bandit14. for this level, you don’t get the next password, but you get a private ssh key that can be used to log into the next level.”
get private key from bandit13:
```bash
scp -p 2220 bandit13@bandit.labs.overthewire.org:~/sshkey.private .
```
modify `~/.ssh/config` file:
```Bash
Host bandit14
	Port 2220
	HostName bandit.labs.overthewire.org
	User bandit14
	IdentityFile ~/sshkey.private
```
Connect `bandit14` and read `/etc/bandit_pass/bandit14`
## bandit14 → bandit15
“The password for the next level can be retrieved by submitting the password of the current level to **port 30000 on localhost**.”
Send passwrod via 
```Bash
telnet localhost 30000
```
## bandit15 → bandit16
“The password for the next level can be retrieved by submitting the password of the current level to port 30001 on localhost using SSL/TLS encryption.”
Use `openssl`:
```Bash
openssl s_client -connect localhost:30001
```
## bandit16 → bandit17
“The credentials for the next level can be retrieved by submitting the password of the current level to a port on localhost in the range 31000 to 32000. First find out which of these ports have a server listening on them. Then find out which of those speak SSL/TLS and which don’t. There is only 1 server that will give the next credentials, the others will simply send back to you whatever you send to it.”
Scan ports with `nmap`:
```Bash
nmap -p 31000-32000 localhost
```
For each port try, `-quiet` to supress `KEYUPDATE`, which occurs when response starts with “k”:
```Bash
openssl s_client connect localhost:port -quiet
```
## bandit17 → bandit18
“There are 2 files in the homedirectory: passwords.old and passwords.new. The password for the next level is in passwords.new and is the only line that has been changed between passwords.old and passwords.new”
Use `diff` to find differences, `diff -y` for side-by-side comparison in two columns.
```Bash
diff -y passwords.old passwords.new
```
## bandit18 → bandit19
“The password for the next level is stored in a file readme in the homedirectory. Unfortunately, someone has modified .bashrc to log you out when you log in with SSH.”
We get thrown out when trying to login so we need to bypass `.bashrc`:
```Bash
ssh bandit18@bandit.labs.overthewire.org -p 2220 -t /bin/sh
```
## bandit19 → bandit20
“To gain access to the next level, you should use the setuid binary in the homedirectory. Execute it without arguments to find out how to use it. The password for this level can be found in the usual place (/etc/bandit_pass), after you have used the setuid binary.”
Execute to read bandit20’s password from his name:
```Bash
./bandit20-do cat /etc/bandit_pass/bandit21
```
## bandit20 → bandit21
“There is a setuid binary in the homedirectory that does the following: it makes a connection to localhost on the port you specify as a commandline argument. It then reads a line of text from the connection and compares it to the password in the previous level (bandit20). If the password is correct, it will transmit the password for the next level (bandit21).”
We have to start server listening with `nc`, connect to it with `./suconnect`, send to `./suconnect` password of bandit20 from `nc`, receive response with next password in `nc`
```Bash
nc -l 1234
^Z
./suconnect 1234
^Z
fg 1
# send previous password
^Z
fg 2
fg 1
# here we get password
```
## bandit21 → bandit22
“A program is running automatically at regular intervals from cron, the time-based job scheduler. Look in /etc/cron.d/ for the configuration and see what command is being executed.”
We should read `/etc/cron.d/cronjob_bandit22`, it executes `/usr/bin/cronjob_bandit22.sh` file, which writes bandit22’s password to `/tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv`, so we get it from here.
## bandit22 → bandit23
“A program is running automatically at regular intervals from cron, the time-based job scheduler. Look in /etc/cron.d/ for the configuration and see what command is being executed.”
Again look into `/etc/cron.d`, find task related to bandit23, it executes `/usr/bin/cronjob_bandit23.sh` which does:
```Bash
myname=$(whoami)
mytarget=$(echo I am user $myname | md5sum | cut -d ' ' -f 1)

echo "Copying passwordfile /etc/bandit_pass/$myname to /tmp/$mytarget"
```
Determine which file we need to look into in `/tmp`:
```Bash
echo I am user bandit23 | md5sum | cut -d ' ' -f 1
```
Read this file.
## bandit23 → bandit24
“A program is running automatically at regular intervals from cron, the time-based job scheduler. Look in /etc/cron.d/ for the configuration and see what command is being executed.”
```Bash
cat /etc/cron.d/cronjob_bandit24 # view cron job
* * * * * bandit24 /usr/bin/cronjob_bandit24.sh &> /dev/null

cat /usr/bin/cronjob_bandit24.sh # view script
# script executes and deletes all scripts from /var/spool/bandit24/foo/

mktemp -d # make our own directory where we will store file for password and script
cd $temp_dir
touch password script.sh

# important: set correct rights for files and temp_dir
chmod 777 temp_dir
chmod +rw password
chmod +rx script.sh

# move file to execution directory
cp script.sh /var/spool/bandit24/foo/script.sh

# read password after a minute
cat password
```
script.sh:
```bash
#!/bin/bash

cat /etc/bandit_pass/bandit24 > /tmp/tmp.bqqdiWI5bP/password
```
## bandit24 → bandit25
“A daemon is listening on port 30002 and will give you the password for bandit25 if given the password for bandit24 and a secret numeric 4-digit pincode. There is no way to retrieve the pincode except by going through all of the 10000 combinations, called brute-forcing.”
Script that does bruteforcing:
```Bash
#!/bin/bash
bandit24_pass=""
for i in {0000..9999}; do
    echo "$bandit24_pass" "$i"
done | nc localhost 30002 | grep -v Wrong
```
## bandit25 → bandit26
“Logging in to bandit26 from bandit25 should be fairly easy… The shell for user bandit26 is not /bin/bash, but something else. Find out what it is, how it works and how to break out of it.”
Shell exits immediately after we login. 
```Bash
# let's check /etc/passwd
cat /etc/passwd | grep bandit26
bandit26:x:11026:11026:bandit level 26:/home/bandit26:/usr/bin/showtext

cat /usr/bin/showtext
exec more ~/text.txt
exit 0

# we need to use more command here
# minimize terminal window, so more will trigger
# then press 'v' to open default text editor (vi in our case)
# once in vi, type :e /etc/bandit_pass/bandit26 and it will show password
```
## bandit26 → bandit27
“Good job getting a shell! Now hurry and grab the password for bandit27!”
If you type `:set shell=/bin/bash` and then `:shell` in vi, it will throw you in bash, from which you can read password from `/etc/bandit_pass/bandit27`
## bandit27 → bandit28
“There is a git repository at ssh://bandit27-git@bandit.labs.overthewire.org/home/bandit27-git/repo via the port 2220. The password for the user bandit27-git is the same as for the user bandit27. Clone the repository and find the password for the next level.”
Just git clone repo and read README inside.
## bandit28 → bandit29
*“There is a git repository at ssh://bandit27-git@bandit.labs.overthewire.org/home/bandit27-git/repo via the port 2220. The password for the user bandit27-git is the same as for the user bandit27. Clone the repository and find the password for the next level.”*
Clone repo, no password in README,md, but we can see 3 commits in `git log`, password is inside one of them.
```Bash
git log
git checkout <commit> # check all commits to find password in README.md
```
## bandit29 → bandit30
*“There is a git repository at ssh://bandit27-git@bandit.labs.overthewire.org/home/bandit27-git/repo via the port 2220. The password for the user bandit27-git is the same as for the user bandit27. Clone the repository and find the password for the next level.”*
Clone repo and run `git branch -a` to list all branches: local and remote.  The password will be in README.md on branch `remotes/origin/dev`. Remote branches are invisible with `git branch`.
## bandit30 → bandit31
*“There is a git repository at ssh://bandit27-git@bandit.labs.overthewire.org/home/bandit27-git/repo via the port 2220. The password for the user bandit27-git is the same as for the user bandit27. Clone the repository and find the password for the next level.”*
Clone repo and check tags with `git tag`, there will be `secret` annotated tag, read it with `git show secret`, it contains password.
## bandit31 → bandit32
*“There is a git repository at ssh://bandit27-git@bandit.labs.overthewire.org/home/bandit27-git/repo via the port 2220. The password for the user bandit27-git is the same as for the user bandit27. Clone the repository and find the password for the next level.”*
README.md asks us to create key.txt file with “May I come in?” text. key.txt is in .gitignore, so add it with `git add key.txt -f`, then commit and push and server will answer with password.
## bandit32 → bandit33
*”After all this `git` stuff, it’s time for another escape. Good luck!”*
Shell doesn’t let us run any command, but we can use `${}` to access environment variables and execute their values as commands. Variable `$0` contains name of the running shell. Difference between `$SHELL` and `$0` is that first one contains script, which will be executed on user login, whereas `$0` contains name of currently running shell (`sh` in our case).