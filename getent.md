# getent – get entries from administrative database
*getent* is Unix command which helps you query one of the following administrative databases in Unix: passwd, group,hosts, services, protocols, or networks.

## Administrative databases in Unix

As you can probably see from their names, the administrative
databases are here to help you gather the most vital
information about your environment:
- passwd – can be used to confirm usernames, userids, home directories and full names of your users
- group – all the information about Unix groups known to your system
- services – all the Unix services configured on your system
- networks – networking information – what networks your system belongs to
- protocols – everything your system knows about network protocols

## How To Use getent
My home Linux PC has a hostname of centos7. If I ever need to double-check which IPs this hostname points to, here’s how I
can use getent:
```shell
greys@centos7:~ getent hosts centos7
127.0.1.1 centos7
192.168.0.2 centos7
```

## getent passwd

Without additional parameters, this command will show you information about all the users available on your system. Most
often this means “users locally created on your server”, but if NIS or LDAP are configured, this might show you a really
long list of all the users in your organisation.

This is an example from my CentOS 7.4 setup:
```shell
greys@centos7:~ $ getent passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
operator:x:11:0:operator:/root:/sbin/nologin
games:x:12:100:games:/usr/games:/sbin/nologin
ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin
nobody:x:99:99:Nobody:/:/sbin/nologin
s y s t e m d - n e t w o r k : x : 1 9 2 : 1 9 2 : s y s t e m d N e t w o r k
Management:/:/sbin/nologin
dbus:x:81:81:System message bus:/:/sbin/nologin
polkitd:x:999:997:User for polkitd:/:/sbin/nologin
postfix:x:89:89::/var/spool/postfix:/sbin/nologin
chrony:x:998:996::/var/lib/chrony:/sbin/nologin
s s h d : x : 7 4 : 7 4 : P r i v i l e g e - s e p a r a t e d
SSH:/var/empty/sshd:/sbin/nologin
ntp:x:38:38::/etc/ntp:/sbin/nologin
t e c h s t a c k : x : 1 0 0 1 : 1 0 0 1 : T e c h S t a c k
Solutions:/home/techstack:/bin/bash
greys:x:1002:1002:Gleb Reys:/home/greys:/bin/bash
influxdb:x:997:993::/var/lib/influxdb:/bin/false
telegraf:x:996:992::/etc/telegraf:/bin/false
g r a f a n a : x : 9 9 5 : 9 9 1 : g r a f a n a
user:/usr/share/grafana:/sbin/nologin
dd-agent:x:994:990:Datadog Agent:/opt/datadog-agent:/bin/sh
rpc:x:32:32:Rpcbind Daemon:/var/lib/rpcbind:/sbin/nologin
rpcuser:x:29:29:RPC Service User:/var/lib/nfs:/sbin/nologin
n f s n o b o d y : x : 6 5 5 3 4 : 6 5 5 3 4 : A n o n y m o u s N F S
User:/var/lib/nfs:/sbin/nologin
```


## Using getent to find a UID by username

getent accepts various keys when searching in databases. For the passwd one, you can user either username or user id (UID)
to search the database.
```shell
greys@centos7:~ getent passwd greys
greys:x:1000:1000:Gleb Reys,,,:/home/greys:/bin/bas
```

## Using getent to find a username by UID
Like I said, the opposite will work as well:
```shell
greys@centos7:~ getent passwd 1000
greys:x:1000:1000:Gleb Reys,,,:/home/greys:/bin/bash
```
## getent group
Without parameters, this will show you all the groups found on your Unix or Linux server:
```shell
greys@centos7:~ $ getent group
root:x:0:
bin:x:1:
daemon:x:2:
sys:x:3:
adm:x:4:
tty:x:5:
disk:x:6:
lp:x:7:
mem:x:8:
kmem:x:9:
wheel:x:10:
cdrom:x:11:
mail:x:12:postfix
man:x:15:
dialout:x:18:
floppy:x:19:
games:x:20:
tape:x:30:
video:x:39:
ftp:x:50:
lock:x:54:
audio:x:63:
nobody:x:99:
users:x:100:
utmp:x:22:
utempter:x:35:
ssh_keys:x:999:
input:x:998:
systemd-journal:x:190:
systemd-network:x:192:
dbus:x:81:
polkitd:x:997:
postdrop:x:90:
postfix:x:89:
chrony:x:996:
sshd:x:74:
ntp:x:38:
cgred:x:995:
docker:x:994:
techstack:x:1001:
greys:x:1002:
influxdb:x:993:
telegraf:x:992:
grafana:x:991:
dd-agent:x:990:
rpc:x:32:
rpcuser:x:29:
nfsnobody:x:65534:
printadmin:x:989:
vboxusers:x:988:
```

## Use getent to show Unix group
If you know a Unix group name, getent will help you confirm its numeric group ID and show members (usernames).
In this example below, system group mail has group ID of 12, and system user postfix as its member.
```shell
greys@centos7:~ $ getent group mail
mail:x:12:postfix
```
## See also:
- id – print user identity
- who – see who is logged into the system
- Basic Unix/Linux commands
- Advanced Unix Commands
