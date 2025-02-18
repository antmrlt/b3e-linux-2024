🌞 **Déterminer l'existant :**

- lister tous les utilisateurs créés sur la machine
```
[antna@node1 ~]$ cat /etc/passwd
```
- lister tous les groupes d'utilisateur
```
[antna@node1 ~]$ cat /etc/group
```

- déterminer la liste des groupes dans lesquels se trouvent votre utilisateur*
```
[antna@node1 ~]$ groups
```

🌞 **Lister tous les processus qui sont actuellement en cours d'exécution, lancés par `root`**

```
[antna@node1 ~]$ ps -fu root
```

🌞 **Lister tous les processus qui sont actuellement en cours d'exécution, lancés par votre utilisateur**

```
[antna@node1 ~]$ ps -fu root
```

🌞 **Déterminer le hash du mot de passe de `root`**

```
[antna@node1 ~]$ sudo cat /etc/shadow | grep root | cut -d':' -f1,2
root:$6$Ee/tfRUDPuxcJNMd$VFgZ5jKb5HjInlr6W5CPHwi0Jy5TNkBF4DTNK1.XkiLOzBnfaP5BqAdghw2rPQdjygVI/fo.jwnvSgFWz8vLs0
```

🌞 **Déterminer le hash du mot de passe de votre utilisateur**

```
[antna@node1 ~]$ sudo cat /etc/shadow | grep antna | cut -d':' -f1,2
antna:$6$NkRjinwFu/o/FksO$FR.s2uxlOuHrDlLumbMcMNVe41C/N0ErcV3Nst.fkL3Hcovqvcfh1soUKhDmQq.V7BrEiMPAWzuH66mqPkI5u/
```

🌞 **Déterminer la fonction de hachage qui a été utilisée**

`antna:$6$NkRjinwFu/o/FksO$FR.s2uxlOuHrDlLumbMcMNVe41C/N0ErcV3Nst.fkL3Hcovqvcfh1soUKhDmQq.V7BrEiMPAWzuH66mqPkI5u/`
        ^ id : 6, donc c'est la fonction de hashage sha-512 qui a été utilisé

```
ID  | Method
─────────────────────────────────────────────────────────
1   | MD5
2a  | Blowfish (not in mainline glibc; added in some
    | Linux distributions)
5   | SHA-256 (since glibc 2.7)
6   | SHA-512 (since glibc 2.7)
```

🌞 **Déterminer, pour l'utilisateur `root`** :

- son shell par défaut
```
[antna@node1 ~]$ cat /etc/passwd | grep root:x: | cut -d':' -f1,7
root:/bin/bash
```
- le chemin vers son répertoire personnel
```
[antna@node1 ~]$ cat /etc/passwd | grep root:x: | cut -d':' -f1,6
root:/root
```

🌞 **Déterminer, pour votre utilisateur** :

- son shell par défaut
```
[antna@node1 ~]$ cat /etc/passwd | grep antna:x: | cut -d':' -f1,7
antna:/bin/bash
```
- le chemin vers son répertoire personnel
```
[antna@node1 ~]$ cat /etc/passwd | grep antna:x: | cut -d':' -f1,6
antna:/home/antna
```

🌞 **Afficher la ligne de configuration du fichier `sudoers` qui permet à votre utilisateur d'utiliser `sudo`**

```
[antna@node1 ~]$ sudo cat /etc/sudoers | grep %wheel
%wheel  ALL=(ALL)       ALL
```

🌞 **Créer un utilisateur :**

```
[antna@node1 ~]$ sudo useradd -M -N -g admins -s /sbin/nologin meow
```

🌞 **Configuration `sudoers`**

- ajouter une configuration `sudoers` pour que l'utilisateur `meow` puisse exécuter seulement et uniquement les commandes `ls`, `cat`, `less` et `more` en tant que votre utilisateur
```
moew ALL=(antna) /bin/ls, /bin/cat, /bin/more, /bin/less
```
- ajouter une configuration `sudoers` pour que les membres du groupe `admins` puisse exécuter seulement et uniquement la commande `apt` en tant que `root`
```
%admins   ALL=(root) /bin/apt
```
- ajouter une configuration `sudoers` pour que votre utilisateur puisse exécuter n'importe quel commande en tant `root`, sans avoir besoin de saisir un mot de passe
```
antna ALL=(root)  NOPASSWD:ALL
```
- prouvez que ces 3 configurations ont pris effet (vous devez vous authentifier avec le bon utilisateur, et faire une commande `sudo` qui doit fonctioner correctement)
```
[meow@node1 ~]$ sudo -l
Matching Defaults entries for meow on node1:
    !visiblepw, always_set_home, match_group_by_gid, always_query_group_plugin, env_reset, env_keep="COLORS DISPLAY HOSTNAME HISTSIZE KDEDIR LS_COLORS",
    env_keep+="MAIL PS1 PS2 QTDIR USERNAME LANG LC_ADDRESS LC_CTYPE", env_keep+="LC_COLLATE LC_IDENTIFICATION LC_MEASUREMENT LC_MESSAGES",
    env_keep+="LC_MONETARY LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE", env_keep+="LC_TIME LC_ALL LANGUAGE LINGUAS _XKB_CHARSET XAUTHORITY",
    secure_path=/sbin\:/bin\:/usr/sbin\:/usr/bin

User meow may run the following commands on node1:
    (antna) /bin/ls, /bin/cat, /usr/bin/less, /usr/bin/more
    (root) /bin/apt

[meow@node1 ~]$ sudo -u antna /bin/ls /tmp
systemd-private-74f7b2e320ff4ed89e4301081b44a2af-chronyd.service-pGAwDp      systemd-private-74f7b2e320ff4ed89e4301081b44a2af-kdump.service-kKMrPh
systemd-private-74f7b2e320ff4ed89e4301081b44a2af-dbus-broker.service-KLWuVm  systemd-private-74f7b2e320ff4ed89e4301081b44a2af-systemd-logind.service-VFaWpA
```

🌞 **Déjà une configuration faible ?**

```
[meow@node1 ~]$ sudo -u antna /bin/less /etc/profile

# puis taper `v` pour ouvrir l'editeur et puis `:shell` pour pop un shell

[antna@node1 meow]$ whoami
antna
[antna@node1 meow]$
```

`meow ALL=(meow) /bin/ls, /bin/cat, /usr/bin/less, /usr/bin/more`

🌞 **Déterminer les permissions des fichiers/dossiers...**

- le fichier qui contient la liste des utilisateurs
```
[antna@node1 ~]$ ls -l /etc/passwd
-rw-r--r--. 1 root root 1014 Feb 18 10:51 /etc/passwd
```
- le fichier qui contient la liste des hashes des mots de passe des utilisateurs
```
[antna@node1 ~]$ ls -l /etc/shadow
----------. 1 root root 758 Feb 18 10:51 /etc/shadow*
```
- le fichier de configuration du serveur OpenSSH
```
[antna@node1 ~]$ ls -l /etc/ssh/sshd_config
-rw-------. 1 root root 3670 Feb 17 17:17 /etc/ssh/sshd_config
```
- le répertoire personnel de l'utilisateur `root`
```
[antna@node1 ~]$ sudo ls -ld /root
dr-xr-x---. 3 root root 4096 Feb 18 10:34 /root
```
- le répertoire personnel de votre utilisateur
```
[antna@node1 ~]$ sudo ls -ld /home/antna/
drwx------. 3 antna antna 148 Feb 17 17:12 /home/antna/
```
- le programme `ls`
```
[antna@node1 ~]$ ls -l /usr/bin/ls
-rwxr-xr-x. 1 root root 140952 Nov  6 17:29 /usr/bin/ls
```
- le programme `systemctl`
```
[antna@node1 ~]$ ls -l /usr/bin/systemctl
-rwxr-xr-x. 1 root root 305744 Nov 16 02:22 /usr/bin/systemctl
```

🌞 **Restreindre l'accès à un fichier personnel**

```
[antna@node1 ~]$ nano dont_read.txt
[antna@node1 ~]$ chmod 600 dont_read.txt
[antna@node1 ~]$ cat dont_read.txt
I SAID DONT READ ME
[antna@node1 ~]$ echo "!!" >> dont_read.txt
echo "cat dont_read.txt " >> dont_read.txt
[antna@node1 ~]$ cat dont_read.txt
I SAID DONT READ ME
cat dont_read.txt
```

```
[antna@node1 ~]$ sudo -u root cat /home/antna/dont_read.txt
I SAID DONT READ ME
cat dont_read.txt
```

```
[antna@node1 ~]$ sudo -u meow cat /home/antna/dont_read.txt
[sudo] password for antna:
cat: /home/antna/dont_read.txt: Permission denied
```

🌞 **Lister tous les programmes qui ont le bit SUID activé**

```
[antna@node1 ~]$ sudo find / -perm /4000
```

🌞 **Rendre le fichier `dont_readme.txt` immuable**

```
[antna@node1 ~]$ sudo chattr +i dont_read.txt

[antna@node1 ~]$ sudo rm dont_read.txt
rm: cannot remove 'dont_read.txt': Operation not permitted

[antna@node1 ~]$ sudo -u root rm /home/antna/dont_read.txt
rm: cannot remove '/home/antna/dont_read.txt': Operation not permitted
```
