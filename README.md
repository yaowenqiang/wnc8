What's New in Centos 8

# Using the Cockpi web interface

## Cockpit Web Console

+ Since CentOS 7.5 the web console, cockpit has been available
+ View Performance: Real-time performance data
+ Manage users: Create users and groups
+ Read logs: Access and filter log files

## Socket Units

When a socket unit starts the port used by the service is held open by systemd.When users connect to the service it is started. When service is no longer needed the service shuts down and systemd maintains the socket.

> sudo systemctl enable --now cockpit.socket
> sudo ss -ntlp
> sudo systemctl status cockpit.socket
> sudo systemctl status cockpit.service
> http://ip:9090


> yum list cockpit*
> yum info cockpit-dashboard


# Using the enhanced firewall: nftables

## Nftables

Released with the Linux kernel 3.13 in January 2014 
nftables can be used as a replacement for netfilter modules used previously.the Command nft is used to natively manage the rules and combines management if IPv4, IPv6, ARP and Bridge filters

Nftables is without Default Tables

Unlike netfilter modules and the associated iptables command, nftables does not have any predefined tables. These all need to be defined.By default in CentOS 8 this is handled by firewalld.

> systemctl disable --now firewalld
> reboot
> nft list tables
> nft list tables ip
> systemctl enable --now firewalld
> nft list tables
> nft list tables ip
> nft list table ip filter

## Creating a Table and Chain

Tables and Chains are the basis of firewall rules. Disabling firewalld and rebooting the system will allow us to build everything from scratch. Using inet as the family we can work with both IPv4 and IPv6

> systemctl disable --now firewalld; reboot
> nft list tables
> nft add table inet filter
> nft add chain inet filter INPUT { type filter hook input priority 0\; policy accept\;}
> nft list table inet filter
> nft list ruleset


### Basick chain Types

+ filter
> route
> nat

### Basic Hooks Types

+ prerouting
> input
> forward
> output
> postrouting
> ingress


### Building a Basic Nftables Firewall

If we need inbound SSH connection to the system a basic firewall is not that different to one we would build with iptables but working with both IPv4 and IPv6

> nft add rule inet filter INPUT iif lo accept
> nft add rule inet filter INPUT ct state established, related accept
> nft add rule net filter INPUT tcp dport 22 accept
> nft add rule net filter INPUT counter drop

### Persisting nftables Rules

We can list the complete ruleset and redirect to a file. We can then flush the table, clearing all associated chains and delete the table. Reestablishing rules by reading the file back with the option -f


> nft list ruleset > /root/myrules
> nft flush table inet filter
> nft delete table inet filter
> nft -f /root/myrules
> nft list ruleset / etc/sysconfig/nftables.conf
> nft flush table inet filter
> nft delete table inet filter
> systemctl enable --now nftables





# Managing NFSv4 with nfsconf

## Installing and configuring NFS


NFS versions

The NFS server and client are installed using the nfs-utils package as it was in CentOS 7. A default install will disable NFSv2 and enable NFSv3 and above. Disabling NFSv3 makes the system more firewall friendly. only needing TCP port 2049 to be opened


configuring NFS in CentOS 8

We can either edit the new /etc/nfs.conf file or make use of the equally new nfsconf command line utility

> yum install nfs-utils
> systemctl  enable --now nfs-server.service
> ss -tl
> ss -tnl
> ss  -4 -tnl
> systemctl  mask --now rpc-statd rpcbind.service  rpcbind.socket



> nfsconf --set nfsd vers4 y
> nfsconf --set  nfsd tcp y
> nfsconf --set  nfsd udp n
> nfsconf --set  nfsd vers3 n



Allowing Access through Firewalld

On the NFS server we can allow inbound TCP port 2049 by making use of the nfs service xml file.


> firewall-cmd --add=service=nfs
> firewall-cmd --runtime-to-permanent


> mkdir  /share
> find /usr/shar/doc -name "*.txt" -exec cp  {} /share \; 
> vim /etc/exports
> /share *(rw)

> exportfs -rav
> exportfs -v
> firewall-cmd --list-all
> firewall-cmd --add-service=nfs --permanent


> yum list nfs-utils

> mount bog:/share /mnt
> lst /mnt


### SELinux Support for NFSv4

SELinux support for NFS continues and there are many customizations that can be made to SELinux polcies to better enable NFS in the way you need. The first step is to make sure you have the MAN pages installed

### Installing NFS Man pages

MAN pages for SELinux types can be installed using the command sepolicy

> yum provides "*/sepolicy"
> yum install policycoreutils-devel
> sepolicy manpage -d nfsd_t -p /usr/share/man/man8
> mandb
> apropos _selinux

# Layered storage management using stratis

## Strais Vaolume Management

Managing volumes with Stratis allows you to create thinly provisioned volumes file systems with a single command whilst utilizing existing dev-mapper and XFS technology

> yum install stratisd stratis-cli
> systemctl enable  --now stratisd

## Managing Stratis Pools

Stratis pools aggregate storage space and represent volume groups and thin pools in DM management. The sub-command add-data is used to extend the size of an existing pool

> stratis pool create pool1 /dev/sdb
> stratis pool add-data pool1 /dev/sdc
> stratis pool list
> blockdev

## Creating pools

> lsblk
> wipefs --all /dev/sdb
> wipefs --all /dev/sdc
> ps -fp $(pgrep stratis)
> stratis daemon version
> stratis pool create  pool1 /dev/sdb
> stratis pool
> stratis  pool add-data pool1 /dev/sdc
> stratis  blockdev
> stratis filesystem


## Managing Stratis File Systems

Stratis volumes are thinly provisioned and as such we do not set the size. They are assigned more space than is available. Ensure that the mount option in the /etc/fstab is used to ensure the service is running before the mount is attempted.

> stratis filesystem create pool1 fs1
> mount /stratis/pool1/fs1 /data
> x-systemd.requires=stratisd.service


> vim /etc/fstab

> /stratis/pool1/fs1 /data xfs defaults,x-systemd.requires=stratisd.service 0 0

> mount -a
> mount -t XFS
> find /usr/share/doc/  -name '*.html' -exec cp {} /data \;
> df -h /data

## Snapshots


Snapshots are point in time copies of a file system and can be used as a method of rolling back changes on a simple online backup agent.


> stratis filesystem snapshot pool1 fs1 snap1

> mount /stratis/pool1/snap1 /backup
> rm -f /data*
> umount /data
> umount /backup
> vim /etc/fstab
> stratis filesystem destroy pool1 fs1

# Data de=duplication and compressiion using VDO

## VDO

The Virtoal Data Optimizer became available with CentOS 7.5 and allows for compression and deduplication of data by Linux Kernel modules

## Installing VDO


You will probably find that these items are installed on most CentOS 8 deployments. You will need a disk larger than 4 GB to use as a VDO device
    
> yum install vdo kmod-kvdo
> systemd enable --now vdo.service


## Creating VDO Devices

We create a logical size for the volume as it is this size that is used by the file system without knowkedge of the gains from optimization

> modprobe kvdo
> vdo create --name=vdo1 --device=/dev/sdb --vdoLogicalSize=20G
> vdo status --name=vdo1 | grep -E '(Dedup|Compression)'
> mkfs.xfs -K /dev/mapper/vdo1
> mkfs.xfs -K /dev/mapper/vdo2

## Mounting VDO Devices

When mounting VDO devices from the /etc/fstab file ,don't overlook the mount option. We can view device usage to mointor actual space left.

> x-systemd.requires=vdo.service
> vdostats --human-readable
> vim /etc/fstab
> /dev/mapper/vdo1 /data xfs defaults,x-systemd.requires=vdo.service 0 0
> mount -a
> ls -lh /boot

> for i in {1..100};  do  echo $i; done
> for i in {1..100};  do  cp /boot/vmlinuz-4.18.0-193.6.3.el8_2.x86_64 /data/$i; done
> du -sh /data
> vdostats --human-readable



## Increasing the Logical Size

The vdoLogicalSize is the size of the thinly provisioned device presented to the system. Red Hat recommends a 10:1 ratio for filesystems hosting virtual machines and a 3:1 ratio for object stores such as ceph. We can increase but not reduce the logical size.
 
> vdo growLogical --name=vdo1 --vdoLogicalSize=40G
> xfs_growfs   /data
> df -h /data
> vdo status --name=vdo1 | grep 'Logical size'


## Controlling Features

Compression adn deduplication can be independently enabled and disabled. Both are enabled by default

> vdo disableDeduplication  --name=vdo1
> vdo enableDeduplication  --name=vdo1



> Linux Kernel 4.18
> ping 127.1
> ping 1.1
