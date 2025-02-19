🌞 **Utiliser `file` pour déterminer le type de :**

- la commande `ls`
```
[antmrlt@vbox ~]$ file /usr/bin/ls
/usr/bin/ls: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=fe37adecca22a782c4fb274ae601f220cc1fbb4d, for GNU/Linux 3.2.0, stripped
```
- la commande `ip`
```
[antmrlt@vbox ~]$ file /usr/sbin/ip
/usr/sbin/ip: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=77a2f5899f0529f27d87bb29c6b84c535739e1c7, for GNU/Linux 3.2.0, stripped
```
- un fichier `.mp3` que vous aurez téléchargé sur le disque de la VM
```
[antmrlt@vbox ~]$ file myfile.mp3
myfile.mp3: Audio file with ID3 version 2.3.0, contains:MPEG ADTS, layer III, v1, 320 kbps, 44.1 kHz, JntStereo
```

🌞 **Utiliser `readelf` sur le programme `ls`**

- afficher le *header* du programme
```
[antmrlt@vbox ~]$ readelf -h /usr/bin/ls
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              DYN (Shared object file)
  Machine:                           Advanced Micro Devices X86-64
  Version:                           0x1
  Entry point address:               0x6b10
  Start of program headers:          64 (bytes into file)
  Start of section headers:          138952 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         13
  Size of section headers:           64 (bytes)
  Number of section headers:         30
  Section header string table index: 29
```
- afficher la liste des sections du programme
```
[antmrlt@vbox ~]$ readelf -S /usr/bin/ls
There are 30 section headers, starting at offset 0x21ec8:

Section Headers:
  [Nr] Name              Type             Address           Offset
       Size              EntSize          Flags  Link  Info  Align
  [ 0]                   NULL             0000000000000000  00000000
       0000000000000000  0000000000000000           0     0     0
...
       0000000000000b08  0000000000000000           0     0     1
  [29] .shstrtab         STRTAB           0000000000000000  00021da0
       0000000000000128  0000000000000000           0     0     1
Key to Flags:
  W (write), A (alloc), X (execute), M (merge), S (strings), I (info),
  L (link order), O (extra OS processing required), G (group), T (TLS),
  C (compressed), x (unknown), o (OS specific), E (exclude),
  l (large), p (processor specific)
```
- déterminer à quel adresse commence le code du programme
```
[antmrlt@vbox ~]$ readelf -S /usr/bin/ls | grep -A 1 text
  [15] .text             PROGBITS         0000000000004d50  00004d50
       0000000000012532  0000000000000000  AX       0     0     16
```

🌞 **Utiliser `ldd` sur le programme `ls`**

```
[antmrlt@vbox ~]$ ldd /usr/bin/ls
        linux-vdso.so.1 (0x00007ffc33b74000)
        libselinux.so.1 => /lib64/libselinux.so.1 (0x00007ff579db8000)
        libcap.so.2 => /lib64/libcap.so.2 (0x00007ff579dae000)
        libc.so.6 => /lib64/libc.so.6 (0x00007ff579a00000)
        libpcre2-8.so.0 => /lib64/libpcre2-8.so.0 (0x00007ff579d12000)
        /lib64/ld-linux-x86-64.so.2 (0x00007ff579e0f000)
[antmrlt@vbox ~]$ ldd /usr/bin/ls | grep libc.so
        libc.so.6 => /lib64/libc.so.6 (0x00007fc8aaa00000)
```

🌞 **Donner le nom ET l'identifiant unique d'un syscall qui permet à un processus de...**

- lire un fichier stocké sur disque : `0 read`
- écrire dans un fichier stocké sur disque : `1 write`
- lancer un nouveau processus : `57 fork`

🌞 **Utiliser `objdump`** sur la commande `ls`

- afficher le contenu de la section `.text`
```
[antmrlt@vbox ~]$ objdump -s -j .text /usr/bin/ls

/usr/bin/ls:     file format elf64-x86-64

Contents of section .text:
 04d50 50e8daf9 ffffe8d5 f9ffffe8 d0f9ffff  P...............
 04d60 e8cbf9ff ffe8c6f9 ffffe8c1 f9ffffe8  ................
 04d70 bcf9ffff e8b7f9ff ffe8b2f9 ffff6690  ..............f.
 04d80 f30f1efa 41574156 41554154 55534883  ....AWAVAUATUSH.
 04d90 ec78488b 2e64488b 04252800 00004889  .xH..dH..%(...H.
```
- mettez en évidence quelques lignes qui contiennent l'instruction `call`
```
[antmrlt@vbox ~]$ objdump  -d /usr/bin/ls | grep callq
    4014:       ff d0                   callq  *%rax
    4d51:       e8 da f9 ff ff          callq  4730 <abort@plt>
    4d56:       e8 d5 f9 ff ff          callq  4730 <abort@plt>
    4d5b:       e8 d0 f9 ff ff          callq  4730 <abort@plt>
    4d60:       e8 cb f9 ff ff          callq  4730 <abort@plt>
    4d65:       e8 c6 f9 ff ff          callq  4730 <abort@plt>
    ...
```
- mettez en évidence quelques lignes qui contiennent l'instruction `syscall`
```
[antmrlt@vbox ~]$ objdump  -d /usr/bin/ls | grep syscall
[antmrlt@vbox ~]$
```

🌞 **Utiliser `objdump`** sur la librairie Glibc

- mettez en évidence quelques lignes qui contiennent l'instruction `syscall`
```
[antmrlt@vbox ~]$ objdump -d /lib64/libc.so.6 | grep -w 'syscall '
   29769:       0f 05                   syscall
   29799:       0f 05                   syscall
   29843:       0f 05                   syscall
   298af:       0f 05                   syscall
   298ee:       0f 05                   syscall
   29abf:       0f 05                   syscall
   29b32:       0f 05                   syscall
   ...
```
- trouvez l'instrution `syscall` qui exécute le syscall `close()`
```
[antmrlt@vbox ~]$ objdump -d /lib64/libc.so.6 | grep -B 1 syscall | grep -A 1 '$0x3'
```
