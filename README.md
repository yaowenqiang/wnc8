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


# Managing NFSv4 with nfsconf
# Layered storage management using stratis
# Data de=duplication and compressiion using VDO


> Linux Kernel 4.18
> ping 127.1
> ping 1.1
