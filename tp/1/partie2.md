🌞 **Attribuer l'adresse IP `10.1.1.11/24`** à la VM

```
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:61:ca:77 brd ff:ff:ff:ff:ff:ff
    inet 10.1.1.11/24 brd 10.1.1.255 scope global noprefixroute enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe61:ca77/64 scope link
       valid_lft forever preferred_lft forever
```

🌞 **Attribuer le nom `node1.tp1.b3` à la VM**

```
[antna@node1 ~]$ history | grep 'sudo hostname'
    7  sudo hostnamectl set-hostname node1.tp1.b3
```

🌞 **Déterminer la liste des programmes qui écoutent sur un port TCP**

```
[antna@node1 ~]$ sudo ss -tulpn | grep tcp
tcp   LISTEN 0      128          0.0.0.0:22        0.0.0.0:*    users:(("sshd",pid=723,fd=3))
tcp   LISTEN 0      128             [::]:22           [::]:*    users:(("sshd",pid=723,fd=4))
```

🌞 **Déterminer la liste des programmes qui écoutent sur un port UDP**

```
[antna@node1 ~]$ sudo ss -tulpn | grep udp
udp   UNCONN 0      0          127.0.0.1:323       0.0.0.0:*    users:(("chronyd",pid=688,fd=5))
udp   UNCONN 0      0              [::1]:323          [::]:*    users:(("chronyd",pid=688,fd=6))
```

🌞 **Pour chacun des ports précédemment repérés...**

```
[antna@node1 ~]$ sudo firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8
  sources:
  services: cockpit dhcpv6-client ssh
  ports:
  protocols:
  forward: yes
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

🌞 **Fermez tous les ports inutilement ouverts dans le firewall**

``` 
[antna@node1 ~]$ sudo firewall-cmd --remove-service cockpit
success
[antna@node1 ~]$ sudo firewall-cmd --remove-service dhcpv6-client
success
[antna@node1 ~]$ sudo firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8
  sources:
  services: ssh
  ports:
  protocols:
  forward: yes
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

🌞 **Pour toutes les applications qui sont en écoute sur TOUTES les adresses IP**

Dans le fichier `/etc/ssh/sshd_config`
```
ListenAddress 10.1.1.11
```

```
[antna@node1 ~]$ sudo ss -tulnep
Netid      State       Recv-Q      Send-Q           Local Address:Port            Peer Address:Port      Process
udp        UNCONN      0           0                    127.0.0.1:323                  0.0.0.0:*          users:(("chronyd",pid=688,fd=5)) ino:20975 sk:1 cgroup:/system.slice/chronyd.service <->
udp        UNCONN      0           0                        [::1]:323                     [::]:*          users:(("chronyd",pid=688,fd=6)) ino:20976 sk:2 cgroup:/system.slice/chronyd.service v6only:1 <->
tcp        LISTEN      0           128                  10.1.1.11:22                   0.0.0.0:*          users:(("sshd",pid=1724,fd=3)) ino:28753 sk:7 cgroup:/system.slice/sshd.service <->
```
