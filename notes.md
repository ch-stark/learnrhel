# RHCSA Course

### How to create a Sudouser
echo "%adminuser ALL=(ALL) ALL" >> /etc/sudoers.d/adminuser


How to use Tar Command in Linux with example
https://www.interserver.net/tips/kb/use-tar-command-linux-examples/

e.g
# tar -cvzf Videos.tar ./Latest/


### Increase Swap-Size

https://linuxconfig.org/redhat-8-increase-swap-size

### Recover Root Password

```
https://linuxconfig.org/redhat-8-recover-root-password

type e and enter:
rd.break

mount | grep sysroot

mount -o remount,rw /sysroot/
mount | grep sysroot

chroot /sysroot

touch /.autorelabel
```

### Local and Remote Logins

#### Commands Introduced

```
### Generate an SSH key
ssh-keygen
### Copy SSH key to another host
ssh-copy-id user@host
### Access rh support
redhat-support-tool
### Generate SOS Report -> /var/tmp/sosreport-*.tar.xz
sosreport
```

### File System Navigation

```
### Copy, move, remove(directory), make directory, touch a file
cp, mv, rm(dir), mkdir, touch
### List files(directories)
ls (-d)
### Link Files|Directories
ln [-s] TARGET LINK_NAME
```

### Users and Groups

```
### Info about current logged in user
id
### Local user information
/etc/passwd
username:password:UID:GID:GECOS:/home/dir:shell
### Group info
/etc/group
groupname:password:GID:list,of,users
### Switch user (temporarily elevate)
su(do)
### User add|modify|remove
user(add|mod|del)
### Set passwords
passwd
### Group add|modify|remove
group(add|mod|del)
### User password data (account aging)
/etc/shadow
name:password:lastchange:minage:maxage:warning:inactive:expire:blank
### Password aging
chage
### Shadow password config
/etc/login.defs
### Disable login
/sbin/nologin
### LDAP Config
### server
yum -y install authconfig-gtk sssd krb5-workstation
### client
yum -y ipa-client && ipa-client-install
/etc/openldap/ldap.conf
### Kerberos
/etc/krb5.conf
### System security services daemon (sssd)
/etc/sssd/sssd.conf
### Check ldap
getent passwd ldapuserX
```

#### Combined Users Lab

```
### newly created users should change password every 30days
sed -i 's/^PASS_MAX_DAYS.*$/PASS_MAX_DAYS 30/' /etc/login.defs
### Create consultants with GID 900
groupadd -g 900 consultants
### Add sspade bboop dtracy to consultants (password = default)
### Expire accounts in 90 days
for u in sspade bboop dtracy; do
  useradd -G consultants $u
  echo 'default' | passwd --stdin $u
  chage -E $(date +%Y-%m-%d -d +90days) $u
done
### bboop should change password every 15 days
chage -M 15 bboop
### Change password on first login
for u in sspade bboop dtracy; do
  chage -d 0 $u
done
### Install ipa-client
yum -y install ipa-client
### Configure ipa-client
ipa-client-install --domain=server7.example.com --no-ntp --mkhomedir
```

### File Permissions

```
### Change file permissions
chmod WhoWhatWhich file|directory
### Change file ownership
chown user:group file
### Default file permissions
umask
### Get|Set Access control lists
(get|set)acl
### Switch group
newgrp [group]
```

### SELinux Permissions

```
### Display|Set SELinux contexts
ps,ls,cp,mkdir -Z
### Get|Set SELinux Mode
(get|set)enforce
### Manage context
semanage fcontext -a -t httpd_sys_content_t '/custom(/.*)?'
+++
restorecon -Rv file|dir
### Get|Set SELinux Booleans
(get|set)sebool
### List
semanage -l
### Get info about selinux failures
tail /var/log/audit/audit.log /var/log/messages
sealert -l ##SELinux alert##
```

### Process Management

```
### Send signal to process(es)
kill(all), pkill
### Find processes
pgrep, top, w
### How long has system been running
uptime
### Change priority of process
(re)nice
```

### Updating Software Packages

```
yum list|search|info|provides|install|update|remove|history
```

### Creating and Mounting File Systems

```
### Overview of existing partitions with a file system
blkid
### (un)Mount a file system
(u)mount /dev/vdb1 /mnt/placeholder
### Manage partitions
fdisk /dev/vdb + partprobe /dev/vdb
### Create filesystem
mkfs.(xfs,ext4) /dev/vdb1
### Make swap space
mkswap
### Persistent mount fs
/etc/fstab
### Swap (activate|show)
swapon [-a|-s] ++ swapoff
```

### Service Management


```
### Manage system services
systemctl (status|enable|disable|start|stop|mask) service
### Targets
systemctl get-default
systemctl set-default graphical|multi-user.target
### Boot Time systemd
^linux16.*systemd.unit=rescue|emergency.target$
### Grub
/etc/default/grub
grub2-mkconfig > /boot/grub2/grub.cfg
```

#### Recover root password

1. reboot
1. Interrupt boot loader
1. Move cursor to entry that needs to be booted
1. Press `e` to edit entry
1. Move cursor to `linux16` line
1. Append `rd.break` (remember only one console)
1. Remount /sysroot as read-write `mount -o remount,rw /sysroot`
1. Chroot jail `chroot /sysroot`
1. Change password `passwd root`
1. Force SELinux relabel `touch /.autorelabel`
1. Exit

### Network Configuration

```
ip (addr|link|route) show
ping
traceroute access.redhat.com
ss (replaces netstat)
### All files are in /etc/sysconfig/network-scripts
nmcli con (add|mod|show)
nmcli reload
nmcli con up <con_name>
### hostname configuration
/etc/hosts
hostname(ctl)
### Add dns
/etc/resolv.conf
nmcli con mod ID ipv4.dns IP
### Test DNS Server
host HOSTNAME
```

#### Lab: Managing RHEL Networking

1. Create a new connection with a static network connection (name=lab, ip=172.25.7.10/24, gateway=172.25.7.254, dns=172.25.254.254)

```
nmcli con add \
  type ethernet \
  ifname eth0 \
  con-name lab \
  ip4 172.25.7.10/24 \
  gw4 172.25.7.254
nmcli con mod "lab" ipv4.dns 172.25.254.254
```

1. Autostart connection, other connections should not start

```
nmcli con mod "lab" connection.autoconnect yes
nmcli con mod "System eth0" connection.autoconnect no
```

1. Add an address 10.0.7.1/24

```
nmcli con mod "lab" +ipv4.addresses 10.0.7.1/24
```

1. Configure host 10.0.7.1/24 can be referenced as "private"

```
echo "10.0.7.1 private" >> /etc/hosts
```

### System Logging and NTP

```
### Configure rsyslog to loga all debug messages to /var/log/messages-debug
echo "*.debug /var/log/messages-debug" >/etc/rsyslog.d/debug.conf
systemctl restart rsyslog
logger -p local7.debug "Debug log entry created on server7"
tail /var/log/messages-debug
### Show the full system journal
journalctl
### Persistent journal
mkdir /var/log/journal
chown root:systemd-journal /var/log/journal
chmod 2755 /var/log/journal
killall -USR1 systemd-journald
### Time
tzselect
timedatectl set-timezone <TIMEZONE>
systemctl restart chronyd
```

### LVM

```
### Creating LVs
fdisk
partprobe
{pv,vg,lv}{create,remove,display}
mkfs
### Extendings LVs
pvcreate
{vg,lv}extend -r (or xfs_growfs|resize2fs)
```

### Cron Jobs

```
/etc/crontab
/etc/cron.d/*
/etc/cron.{hourly,daily,weekly,monthly}
systemd-tmpfiles(-clean)
systemd-tmpfiles(-clean.service|timer)
```

### Mounting Network File Systems

```
### Kerberos
/etc/krb5.keytab
yum -y install nfs-utils
systemctl enable nfs-secure
showmount -e serverX
mount -t nfs -o sync serverX:/share /mountpoint
### Automounting network files
yum -y install autofs
$ cat /etc/auto.master.d/shared.autofs
# indirectly mounted
/shared/directory /etc/auto.shared
# directly mounted
/- /etc/auto.shared
$ cat /etc/auto.shared
# directl maps
work -rw,sync server7:/shares/work
# indirect maps
* -rw,sync(,sec=krb5p) server7:/shares/&
systemctl (enable + start) autofs
```

#### Automounting NFS

```
wget -O /etc/krb5.keytab http://classroom.example.com/...
yum -y install nfs-utils autofs
systemctl enable nfs-secure
systemctl start nfs-secure
echo "/- /etc/auto.direct" > /etc/auto.master.d/direct.autofs
echo "/mnt/public -rw,sync,sec=krb5p serverX:/shares/public" > /etc/auto.direct
mkdir -p /mnt/public
echo "/shares /etc/auto.shares" > /etc/auto.master.d/shares.autofs
echo "* -rw,sync,sec=krb5p serverX:/shares/&" > /etc/auto.shares
systemctl enable autofs
systemctl start autofs
ssh ldapuserX@localhost
```

#### SMB Example

```
yum -y install cifs-utils autofs
echo "/shares /etc/auto.shares" > /etc/auto.master.d/shares.autofs
echo "work -fstype=cifs,credentials=/etc/me.cred ://serverX/student" >> /etc/auto.shares
echo "docs -fstype=cifs,guest ://serverX/public" >> /etc/auto.shares
echo "cases -fstype=cifs,credentials=/etc/me.cred ://serverX/bakerst" >> /etc/auto.shares
cat <<EOF > /etc/me.cred
  username=student
  password=student
  domain=MYGROUP
EOF
chmod 600 /etc/me.cred
systemctl enable autofs
systemctl start autofs
cat /shares/work/samba.txt (rw)
cat /shares/docs/samba.txt (ro)
cat /shares/cases/samba.txt (rw)
```

### Firewall Configuration

```
firewall-cmd --(get|set)-default-zone
firewall-cmd --zone=public --add-(source|port|service) --permanent
firewall-cmd --reload
```

### Virtualization and Kickstart

```
/root/anaconda-ks.cfg
ksvalidator /path/to/ks
gPXE> autoboot
.../dvd quiet ks=http://desktopX/path/to/ks
