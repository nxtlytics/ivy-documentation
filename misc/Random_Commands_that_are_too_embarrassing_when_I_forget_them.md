# Random Commands that are too embarrassing when I forget them

# Apache

## Common errors

If you get a libGeoIP.so.1: cannot open shared object file: No such file
or directory error, add /usr/local/lib to /etc/ld.so.conf then run
/sbin/ldconfig /etc/ld.so.conf

## Test apache syntax

```shell
apachectl configtest
```

# AWS CLI

## Get latest ami for X

**Amazon Linux 2 example:**

```shell
aws --profile your-profile ec2 describe-images --owners amazon --filters "Name=name,Values=amzn2-ami-minimal*-ebs" --query 'sort_by(Images, &CreationDate)[].{Name: Name, ID: ImageId}  | [::-1] | [0:5]'
[
    {
        "Name": "amzn2-ami-minimal-hvm-2.0.20191217.0-x86_64-ebs",
        "ID": "ami-0e2c4438b8dc5b48b"
    },
    {
        "Name": "amzn2-ami-minimal-hvm-2.0.20191217.0-arm64-ebs",
        "ID": "ami-08f942e7c38d6b8b4"
    },
    {
        "Name": "amzn2-ami-minimal-hvm-2.0.20191116.0-x86_64-ebs",
        "ID": "ami-0d0e4c663fcb087ff"
    },
    {
        "Name": "amzn2-ami-minimal-hvm-2.0.20191116.0-arm64-ebs",
        "ID": "ami-0d9d3f76da27f4a30"
    },
    {
        "Name": "amzn2-ami-minimal-hvm-2.0.20191024.3-x86_64-ebs",
        "ID": "ami-05b6c00fc86fac8f6"
    }
]
```

## Get all IPs for ec2 instances in a given region

```shell
aws --profile your-profile --region=us-east-1 ec2 describe-instances --query "Reservations[*].Instances[*].NetworkInterfaces[*].PrivateIpAddress" --output=text
```

## Terminate instance in auto-scaling group

```shell
function asgdie() {
 INSTANCE_ID="i-${1}"
 DECREMENT=$2
 REGION=${3:-"us-west-2"}
 #REGION="us-west-2"
 if [ "${DECREMENT}" == "-dec" ]; then
   echo "terminating ${INSTANCE_ID} and decrementing..."
   aws autoscaling terminate-instance-in-auto-scaling-group  \
     --should-decrement-desired-capacity                     \
     --region ${REGION} --instance-id ${INSTANCE_ID}
 else
   echo "terminating ${INSTANCE_ID} and replacing..."
   aws autoscaling terminate-instance-in-auto-scaling-group    \
     --no-should-decrement-desired-capacity                    \
     --region ${REGION} --instance-id ${INSTANCE_ID}
 fi
}
```

# Bash

## Send a running process to the background and leave it running after ssh disconnects

1.  "Stop" the process: CTRL + Z

2.  Send it to the backgroung: bg

3.  Confirm it is running: jobs

4.  Disown it so SIGHUP doesn't kill it: disown -h %\<\# of job\>

5.  Done!

## List all programs available in your session

```shell
compgen -c
```

## List all programs with the same name

```shell
type -a [program]
```

## Order in which programs/alias/functions are used/available

There are four things that can be invoked when you type a command on the
command line. They are used in this order:

1.  alias
2.  function
3.  builtin
4.  file(s)

Source:
https://stackoverflow.com/questions/17639831/what-happens-if-two-command-line-programs-share-the-same-name

# dig

## Reverse DNS lookup in linux

```shell
for i in $(cat file_with_ips_separated_by_a_newline.txt); do dig -x ${i}; done
for i in 54.72.56.232 34.251.236.6 34.202.176.40 54.152.16.151; do dig -x ${i}; done
```

## Reverse DNS lookup in windows

```shell
$array = @("54.72.56.232", "34.251.236.6", "34.202.176.40", "54.152.16.151")
foreach ($element in $array) {
  [System.Net.Dns]::gethostentry($element).HostName
}
```

# Docker

## "SSH" to docker for mac

```shell
screen ~/Library/Containers/com.docker.docker/Data/com.docker.driver.amd64-linux/tty 
```

# flock

## Do not duplicate a process if it is already running

```shell
/usr/bin/flock -n /tmp/script.lock /usr/local/bin/script.sh > /dev/null
```

# git

## Test ssh key against github

```shell
$ ssh -T git@github.com
Warning: Permanently added the RSA host key for IP address '192.30.255.113' to the list of known hosts.
Hi nxtlytics! You've successfully authenticated, but GitHub does not provide shell access.
```

## How to solve a merge conflict with vimdiff

http://www.rosipov.com/blog/use-vimdiff-as-git-mergetool/

## vimdiff cheatsheet

https://gist.github.com/mattratleph/4026987

## More on git rebase and merge

https://medium.com/@porteneuve/getting-solid-at-git-rebase-vs-merge-4fa1a48c53aa#.2gzkxzv2m

## Set push to your fork (avoid pushing to original project)

```shell
git remote set-url --push origin <repo url>
```

## Sync your fork from upstream (original repo)

1. git fetch upstream2. git checkout master3. git merge upstream/master

## Mirror upstream to fork

1.  git fetch -p origin

2.  git push --mirror

## Remove Remote

git remote rm

## Rename Remote

git remote rename

## Review las commit

git log --name-status HEAD^..HEAD

## Restore your fork back to the upstream state (FORCE)

1.  Make sure you have upstream setup (see above), also git stash your
    changes before doing this because it will delete everything that has
    not been accepted by the upstream repo

2.  git fetch upstream

3.  git checkout master

4.  git reset --hard upstream/master

5.  git push origin master --force

## git cheatsheet

```shell
-- Basic
# Create a local branch
git checkout -b herpaderp
# Switch branches
git checkout existing-branch
# Merge master into my current branch
git merge master
-- Comparing Branches
# Diff files between two branches
git diff master...existing-branch <path/to/file>
# List all changed files between two branches
git diff --name-status master...existing-branch
-- Resolving Conflicts
# Abort the merge entirely (or any other terrible action)
git reset --hard
# Prefer my current branch’s file
git checkout --ours <path/to/file>
git add <path/to/file>
# Prefer the remote branch’s file
git checkout --theirs <path/to/file>
git add <path/to/file>
# Resolve conflict caused by a removed file “deleted by us/them” # -- remove the file
git rm <path/to/file>
# -- keep the file
git add <path/to/file>
# Complete merge after resolving all conflicts (no arguments!)
git commit
-- Misc Awesome
# Pluck a file or directory from another branch
# (an alternative to handling merge conflicts before they conflict) git checkout <remote branch> -- <path/to/file>
git commit
# Remove untracked files and directories
git clean -fd
```

# Journalctl

## Filter by path

```shell
journalctl /usr/local/bin/some-binary
```

## Send to stdout

```shell
journalctl --no-pager
```

# less

## less with color support (?)

```shell
less -Rf <file>
```

# lsof

## See al ports that are listening

```shell
lsof -Pi TCP -s TCP:LISTEN
```

# netcat

## netcat using proxy

```shell
nc -X connect -x <Proxy IP or Host>:<Proxy Port (eg. 3128)> google.com 443
```

# netstat

## List incoming connections

```shell
watch "netstat -ant | grep <port>"
```

# nfs

## List connected clients

```shell
showmount -a
```

# Regular expressions

Go regex tester: https://regoio.herokuapp.com/

# sed

## Match newline

```shell
sed ':a;N;$!ba;s!line with newline\n!replacement line with newline\n!g' <file> | less
```

# SSH

## openssh private key to RSA key

```shell
$ cp /path/to/private/ssh/key /path/to/private/ssh/key-rsa
$ ssh-keygen -p -N "" -m pem -f /path/to/private/ssh/key-rsa
Key has comment 'user@host.local'
Your identification has been saved with the new passphrase.
```

# Systemd

## Reload Systemd after modifying units

```shell
systemctl daemon-reload
```

# tar

## keep permissions for easy transfer

### On the source computer:

```shell
cd /path/to/folder/to/copy
tar cvpzf put_your_name_here.tar.gz .
```

## Copy `put_your_name_here.tar.gz` to the other computer

### On the destination computer:

```shell
cd /path/to/destination/folder
tar xpvzf put_your_name_here.tar.gz
```

tar will recreate the archived folder structure with all permissionsintact.
Those commands will archive the contents of the source folder and then extract 
them into the destination folder. If you want to copy the folder

itself, then you should, at step 1:

```shell
cd /path/to/parent/folder
tar cvpzf put_your_name_here.tar.gz folder_to_copy
```

The same mechanism could be used for single files.

# tcpdump - tshark

## Suggested page

http://www.rationallyparanoid.com/articles/tcpdump.html

## DNS query and resp addr

**Command:**

```shell
tshark -i eth0 -f "src port 53" -n -T fields -e frame.time -e ip.src -e ip.dst -e dns.qry.name -e dns. resp.addr
```

**Example:**

```shell
[root@hostname ~]# tshark -i eth0 -f "src port 53" -n -T fields - e frame.time -e ip.src -e ip.dst -e dns.qry.name -e dns.resp.addr
Running as user "root" and group "root". This could be dangerous.
Capturing on eth0
 
Jun 13, 2017 21:01:47.985835000 10.51.6.40 dnsvip.server.tld
Jun 13, 2017 21:01:47.985852000 10.51.6.40 dnsvip.server.tld 10.51.0.201
Jun 13, 2017 21:02:03.442282000 10.51.6.40 dnsvip.server.tld 10.59.191.39
Jun 13, 2017 21:02:03.452753000 10.51.6.39 dnsvip.server.tld 10.59.191.41
Jun 13, 2017 21:02:03.464685000 10.51.6.40 dnsvip.server.tld 10.58.191.15
Jun 13, 2017 21:02:03.527581000 10.51.6.39 dnsvip.server.tld 10.58.191.17
Jun 13, 2017 21:02:03.590627000 10.51.6.40 dnsvip.server.tld 10.59.255.31
Jun 13, 2017 21:02:03.600425000 10.51.6.39 dnsvip.server.tld 10.59.255.33
Jun 13, 2017 21:02:03.611783000 10.51.6.40 dnsvip.server.tld 10.58.255.23
Jun 13, 2017 21:02:03.681430000 10.51.6.39 dnsvip.server.tld 10.58.255.25
Jun 13, 2017 21:02:04.538753000 10.51.6.40 dnsvip.server.tld 10.51.0.201
Jun 13, 2017 21:02:04.538768000 10.51.6.40 dnsvip.server.tld
10.51.73.15 10.51.73.15 10.51.73.15 10.51.73.15 10.51.73.15 10.51.73.15 10.51.73.15 10.51.73.15 10.51.73.15 10.51.73.15 10.51.73.15 10.51.73.15
ew2-nfs- ew2-nfs- eu4a-buff- eu4b-buff- eu5a-buff- eu5b-buff- eu6a-buff- eu6b-buff- eu7a-buff- eu7b-buff- ew2-nfs- ew2-nfs-
```

## HTTP host and user agent based on http requests

Note: tshark only accepts default http ports, if you're using custom
ports you should use tcprewrite prior to this

**Command:**

```shell
tshark -i <Network Device> -Y http.request -T fields -e http.host -e http.user_agent
```

**Example:**

```shell
Ricardos-MacBook-Pro:Downloads ricardo$ sudo /Applications/Wireshark.app/Contents/MacOS/tshark -i en0 -Y http.request -T fields -e http.host -e http.user_agent
Capturing on 'Wi-Fi: en0'
239.255.255.250:1900    Google Chrome/79.0.3945.117 Mac OS X
239.255.255.250:1900    Google Chrome/79.0.3945.79 Mac OS X
239.255.255.250:1900    Google Chrome/79.0.3945.79 Mac OS X
239.255.255.250:1900
239.255.255.250:1900    Google Chrome/79.0.3945.79 Mac OS X
239.255.255.250:1900    Google Chrome/79.0.3945.79 Windows
239.255.255.250:1900    Google Chrome/79.0.3945.88 Mac OS X
239.255.255.250:1900    Google Chrome/79.0.3945.79 Mac OS X
239.255.255.250:1900    Google Chrome/79.0.3945.79 Windows
239.255.255.250:1900    Google Chrome/79.0.3945.88 Mac OS X
239.255.255.250:1900
^C11 packets captured
```

## DHCP

### Based on network device

```shell
tcpdump -i eth0 -n port 67 and port 68
```

### tcpdump filter to match DHCP packets including a specific Client MAC Address:

```shell
tcpdump -i br0 -vvv -s 1500 '((port 67 or port 68) and (udp[38:4] = 0x3e0ccf08))'
```

Note: tcpdump only allows matching on a maximum of 4 bytes (octets), not
the 6 bytes of a MAC address. So, in the above example, we match the
last 4 bytes (presumably the most unique) - our original MAC address was
`00:16:3e:0c:cf:08` , so we match on `3e0ccf08` 

### tcpdump filter to capture packets sent by the client (DISCOVER,REQUEST, INFORM):

```shell
tcpdump -i br0 -vvv -s 1500 '((port 67 or port 68) and (udp[8:1] = 0x1))'
```

Source:

<https://gist.github.com/mattm7n/1405067>

<http://sysadmin.wikia.com/wiki/DHCP_debugging_with_tcpdump>

# Terraform - Terragrunt

## format check .tf and .tfvars files

```shell
terraform fmt
```

## format check .hcl files

```shell
terragrunt hclfmt
```

# Vim

## Disable tabs when pasting

```shell
:set paste
:set nopaste
```

## Set backgroun dark - change vim colors

```shell
:set background=dark
```

Set tab to be changed to 2 spaces (or 4)

source:
http://stackoverflow.com/questions/1878974/redefine-tab-as-4-spaces

### 2 spaces

```shell
:set tabstop=8 softtabstop=0 expandtab shiftwidth=2 smarttab
```

### 4 spaces

```shell
:set tabstop=8 softtabstop=0 expandtab shiftwidth=4 smarttab
```

## Scrolling synchronously

```shell
:set scb
:windo set scb
:set scb! #To disable it
```

# YUM

## List all available versions of a package

```shell
yum list available redis --showduplicates
```

# Zookeeper

## mntr

### mntr output for follower

```shell
$ echo mntr | nc 127.0.0.1 2181
zk_version      3.4.6-1569965, built on 02/20/2014 09:09 GMT
zk_avg_latency  0
zk_max_latency  4
zk_min_latency  0
zk_packets_received     46
zk_packets_sent 45
zk_num_alive_connections        2
zk_outstanding_requests 0
zk_server_state follower
zk_znode_count  277
zk_watch_count  2
zk_ephemerals_count     16
zk_approximate_data_size        260553
zk_open_file_descriptor_count   31
zk_max_file_descriptor_count    4096
```

### mntr output for leader

```shell
$ echo mntr | nc 127.0.0.1 2181
zk_version      3.4.6-1569965, built on 02/20/2014 09:09 GMT
zk_avg_latency  0
zk_max_latency  29
zk_min_latency  0
zk_packets_received     2809237
zk_packets_sent 2809249
zk_num_alive_connections        10
zk_outstanding_requests 0
zk_server_state leader
zk_znode_count  277
zk_watch_count  18
zk_ephemerals_count     16
zk_approximate_data_size        260553
zk_open_file_descriptor_count   41
zk_max_file_descriptor_count    4096
zk_followers    2
zk_synced_followers     2
zk_pending_syncs        0
```

## ruok

### rouk zookeeper ?

```shell
$ echo ruok | nc 127.0.0.1 2181
imok
```
