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

- avec un `cat TRUC | grep TRUC` je veux voir que la bonne ligne

🌞 **Prouvez que votre utilisateur est bien dans le groupe `wheel`**

🌞 **Prouvez que la langue configurée pour l'OS est bien l'anglais**

- je veux une ligne de commande qui affiche la langue actuelle de l'OS
- que vos messages d'erreur soient en anglais ça me suffit pas ;D

🌞 **Prouvez que le firewall est déjà actif**

- le service de firewalling s'appelle `firewalld` sous Rocky (on le manipule avec la commande `firewall-cmd`)
