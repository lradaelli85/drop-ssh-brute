# drop-ssh-brute
iptables rules to mitigate SSH DoS/Brute force attack

# Requirements
- ipset

# Introduction
The aim of this script is to mitigate SSH Dos/Brute force attack using iptables.
With this script no logs are parsed to retrieve attacker IP,everything is done using iptables recent match and ipset

### Drop hosts that open more than 5 SSH connections in 1 minute for 1 day

```
ipset create ssh_drop hash:ip timeout 86400
iptables -A INPUT -m set --match-set ssh_drop src -j DROP
iptables -A INPUT -i ens3 -p tcp --dport 22 -m conntrack --ctstate NEW -m recent --update --reap --seconds 60 --hitcount 5 --name SSH-CHECK --rsource -j SET --add-set ssh_drop src
iptables -A INPUT -i ens3 -p tcp -m tcp --dport 22 -m conntrack --ctstate NEW -m recent --name SSH-CHECK --set -j ACCEPT
```

### Explanation

`ipset create ssh_drop hash:ip timeout 86400`

with this command we will create a set with a entries lifetime of 24 hours (86400 seconds).After 24 hours expired entries are automatically removed

`iptables -A INPUT -m set --match-set ssh_drop src -j DROP`

with this rule we will drop connections coming from IPs stored in ssh_drop set

`iptables -A INPUT -i eth0 -p tcp --dport 22 -m conntrack --ctstate NEW -m recent --update --reap --seconds 60 --hitcount 5 --name SSH-CHECK --rsource -j SET --add-set ssh_drop src`

with this rule we will inspect each new SSH connection coming from eth0 interface and if the same source IP opens 5 connections in 1 minute,add it to ssh_drop set

`iptables -A INPUT -i eth0 -p tcp -m tcp --dport 22 -m conntrack --ctstate NEW -m recent --name SSH-CHECK --set -j ACCEPT`

with this rule we will accept new connections from IPs that are below the threshold (less than 5 connections in 1 minute)


