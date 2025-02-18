🌞 **Prouvez que le schéma de partitionnement a bien été appliqué**

```
[antna@node1 ~]$ lsblk
NAME             MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                8:0    0   30G  0 disk
├─sda1             8:1    0   26G  0 part
│ ├─rl_vbox-root 253:0    0   10G  0 lvm  /
│ ├─rl_vbox-swap 253:1    0    1G  0 lvm  [SWAP]
│ └─rl_vbox-home 253:2    0   15G  0 lvm  /home
└─sda2             8:2    0    1G  0 part /boot
sr0               11:0    1 1024M  0 rom
```

🌞 **Mettre en évidence la ligne de configuration `sudo` qui concerne le groupe `wheel`**

```
[antna@node1 ~]$ sudo cat /etc/sudoers | grep %wheel
%wheel  ALL=(ALL)       ALL
```

🌞 **Prouvez que votre utilisateur est bien dans le groupe `wheel`**

```
[antna@node1 ~]$ groups
antna wheel
```

🌞 **Prouvez que la langue configurée pour l'OS est bien l'anglais**

```
[antna@node1 ~]$ locale | grep LANG
LANG=en_US.UTF-8
```

🌞 **Prouvez que le firewall est déjà actif**

```
[antna@node1 ~]$ systemctl status firewalld
● firewalld.service - firewalld - dynamic firewall daemon
     Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; preset: enabled)
     Active: active (running) since Mon 2025-02-17 14:11:15 CET; 14min ago
       Docs: man:firewalld(1)
   Main PID: 685 (firewalld)
      Tasks: 2 (limit: 23160)
     Memory: 44.9M
        CPU: 743ms
     CGroup: /system.slice/firewalld.service
             └─685 /usr/bin/python3 -s /usr/sbin/firewalld --nofork --nopid

Feb 17 14:11:14 node1.tp1.b3 systemd[1]: Starting firewalld - dynamic firewall daemon...
Feb 17 14:11:15 node1.tp1.b3 systemd[1]: Started firewalld - dynamic firewall daemon.
```
