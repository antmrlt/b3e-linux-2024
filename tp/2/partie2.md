🌞 **Utiliser `strace` pour tracer l'exécution de la commande `ls`**

```
[antmrlt@vbox ~]$ strace ls 2>&1 | grep -A 1 write
write(1, "myfile.mp3\n", 11myfile.mp3
)            = 11
```

🌞 **Utiliser `strace` pour tracer l'exécution de la commande `cat`**

- mettez en évidence le *syscall* qui demande l'ouverture du fichier en lecture
```
[antmrlt@vbox ~]$ strace cat anotherone.txt 2>&1 | grep open | grep anotherone.txt
openat(AT_FDCWD, "anotherone.txt", O_RDONLY) = 3
```
- mettez en évidence le *syscall* qui écrit le contenu du fichier dans le terminal
```
[antmrlt@vbox ~]$ strace cat anotherone.txt 2>&1 | grep -A 1 write
write(1, "LALLAJSKEBHLIFAGBILRUAVJKMRIIUPA"..., 42LALLAJSKEBHLIFAGBILRUAVJKMRIIUPAVVIHRLHAV
) = 42
```

🌞 **Utiliser `strace` pour tracer l'exécution de `curl example.org`**

```
[antmrlt@vbox ~]$ strace -c curl exemple.org
...
time     seconds  usecs/call     calls    errors syscall
------ ----------- ----------- --------- --------- ----------------
 46.71    0.001699          56        30           poll
 11.08    0.000403           7        54           close
  9.68    0.000352         352         1         1 connect
  8.52    0.000310         155         2           socket
  4.56    0.000166          11        14           write
  3.85    0.000140         140         1           sendto
  3.52    0.000128           1        86           rt_sigaction
  2.94    0.000107           4        24           futex
  2.72    0.000099           1        60        14 openat
  1.98    0.000072          36         2           newfstatat
  1.54    0.000056          28         2           recvfrom
  0.74    0.000027           0        36           read
  0.63    0.000023           0        46           fstat
  0.58    0.000021           0       141           mmap
  0.22    0.000008           8         1           pipe
  0.19    0.000007           1         4           setsockopt
  0.16    0.000006           3         2           getdents64
  0.11    0.000004           4         1           getsockopt
  0.08    0.000003           3         1           sysinfo
  0.05    0.000002           2         1           getsockname
  0.05    0.000002           2         1           getpeername
  0.05    0.000002           0         6           fcntl
  0.00    0.000000           0        38           mprotect
  0.00    0.000000           0         1           munmap
  0.00    0.000000           0         4           brk
  0.00    0.000000           0         3           rt_sigprocmask
  0.00    0.000000           0         2           ioctl
  0.00    0.000000           0         4           pread64
  0.00    0.000000           0         2         1 access
  0.00    0.000000           0         2           socketpair
  0.00    0.000000           0         1           execve
  0.00    0.000000           0         2           statfs
  0.00    0.000000           0         2         1 arch_prctl
  0.00    0.000000           0         1           set_tid_address
  0.00    0.000000           0         1           set_robust_list
  0.00    0.000000           0         1           prlimit64
  0.00    0.000000           0         1           getrandom
  0.00    0.000000           0         1           rseq
  0.00    0.000000           0         1           clone3
------ ----------- ----------- --------- --------- ----------------
100.00    0.003637           6       583        17 total
```

🌞 **Utiliser `sysdig` pour tracer les *syscalls*  effectués par `ls`**

```
[antmrlt@vbox ~]$ sudo sysdig proc.name=ls | grep -E 'write|close|open'
```

🌞 **Utiliser `sysdig` pour tracer les *syscalls*  effectués par `cat`**

- mettez en évidence le *syscall* qui demande l'ouverture du fichier en lecture
```
[antmrlt@vbox ~]$ sudo sysdig proc.name=cat | grep open
1022 12:54:54.926484580 0 cat (6607) > openat dirfd=-100(AT_FDCWD) name=/etc/ld.so.cache flags=4097(O_RDONLY|O_CLOEXEC) mode=0
1023 12:54:54.926487222 0 cat (6607) < openat fd=3(<f>/etc/ld.so.cache) dirfd=-100(AT_FDCWD) name=/etc/ld.so.cache flags=4097(O_RDONLY|O_CLOEXEC) mode=0 dev=FD00 ino=33686605
```
- mettez en évidence le *syscall* qui écrit le contenu du fichier dans le terminal
```
[antmrlt@vbox ~]$ sudo sysdig proc.name=cat | grep write
1546 12:55:39.248724634 0 cat (6622) > write fd=1(<f>/dev/tty1) size=42
1547 12:55:39.248870150 0 cat (6622) < write res=42 data=LALLAJSKEBHLIFAGBILRUAVJKMRIIUPAVVIHRLHAV.
```

🌞 **Utiliser `sysdig` pour tracer les *syscalls*  effectués par votre utilisateur**

```
[antmrlt@vbox ~]$ sudo sysdig | grep antmrlt
```

🌞 **Livrez le fichier `curl.scap` dans le dépôt git de rendu**

[fichier curl.scap](curl.scap)

> `sysdig` est un outil moderne qui sert de base à toute la suite d'outils de la boîte du même nom. On pense par exemple à Falco qui permet de tracer, monitorer, lever des alertes sur des *syscalls* , au sein d'un cluster Kubernetes.

## 3. Bonus : Stratoshark

Un tout nouveau tool bien stylé : [Stratoshark](https://wiki.wireshark.org/Stratoshark). L'interface de Wireshark (et ses fonctionnalités de fou) mais pour visualiser des captures de *syscalls*  (et autres).

Vous prenez pas trop la tête avec ça, mais si vous voulez vous amuser avec une interface stylée, il est là !

Vous pouvez exporter une capture `sysdig` avec `sysdig -w meo.scap proc.name=echo` par exemple, et la lire dans Stratoshark. 
