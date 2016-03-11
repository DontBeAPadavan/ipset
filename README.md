# ipset
This [ipset](http://ipset.netfilter.org/) usage example will show how to:
* block incoming connections from TOR nodes and\or China (sorry, guys. Every PPTP brutforce try comes from your IPs).
* block transit traffic from\to Microsoft servers, but leave Skype working.

## Prerequisites
* Padavan firmware with ipset (every build type except nano),
* [Entware installed](https://bitbucket.org/padavan/rt-n56u/wiki/EN/HowToConfigureEntware).

## Installation
* Install script for fill ip sets and make it executable:
```
wget --no-check-certificate -O /opt/etc/init.d/S99_fill_ipset https://raw.githubusercontent.com/DontBeAPadavan/ipset/master/opt/etc/init.d/S99_fill_ipset
chmod +x /opt/etc/init.d/S99_fill_ipset
```
* Edit `/opt/etc/init.d/S10iptables`:
```
#!/bin/sh

case "$1" in
start|update)
        # add iptables custom rules
        [ -d '/opt/etc' ] || exit 0
        iptables -I INPUT -m set --match-set TorNodes src -j DROP
        iptables -I INPUT -m set --match-set China src -j DROP
        iptables -I FORWARD -m set --match-set Microsoft src,dst -j DROP
        iptables -I FORWARD -m set --match-set Skype src,dst -j ACCEPT
        ;;
stop)
        # delete iptables custom rules
        echo "firewall stopped"
        ;;
*)
        echo "Usage: $0 {start|stop|update}"
        exit 1
        ;;
esac
```
* Go to `Customization > Scripts` web interface page and put following content to `Run After Router Started` field:
```
#!/bin/sh

modprobe ip_set_hash_ip
modprobe ip_set_hash_net
modprobe xt_set

ipset -N TorNodes nethash hashsize 4096
ipset -N China nethash hashsize 2048
ipset -N Microsoft nethash
ipset -N Skype iphash
```
* Now switch to `LAN > DHCP Server` page and add this line to `Custom Configuration File "dnsmasq.conf"` field:
```
ipset=/skype.net/Skype
```
* Reboot router to take effect.
