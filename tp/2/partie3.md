🌞 **Tracer l'exécution du programme NGINX**

```
[antmrlt@vbox ~]$ sudo strace -c nginx
% time     seconds  usecs/call     calls    errors syscall
------ ----------- ----------- --------- --------- ----------------
  0.00    0.000000           0        22           read
  0.00    0.000000           0        31           close
  0.00    0.000000           0        24         1 fstat
  0.00    0.000000           0         3           lseek
  0.00    0.000000           0        38           mmap
  0.00    0.000000           0        11           mprotect
  0.00    0.000000           0         2           munmap
  0.00    0.000000           0         6           brk
  0.00    0.000000           0        12           rt_sigaction
  0.00    0.000000           0         2           ioctl
  0.00    0.000000           0         7           pread64
  0.00    0.000000           0         1           pwrite64
  0.00    0.000000           0         1         1 access
  0.00    0.000000           0         3           getpid
  0.00    0.000000           0         8           socket
  0.00    0.000000           0         6         6 connect
  0.00    0.000000           0         2           bind
  0.00    0.000000           0         4           listen
  0.00    0.000000           0         3           setsockopt
  0.00    0.000000           0         1           clone
  0.00    0.000000           0         1           execve
  0.00    0.000000           0         2           uname
  0.00    0.000000           0        10           fcntl
  0.00    0.000000           0         5         5 mkdir
  0.00    0.000000           0         1           sysinfo
  0.00    0.000000           0         1           geteuid
  0.00    0.000000           0         1           getppid
  0.00    0.000000           0         2         1 arch_prctl
  0.00    0.000000           0        22           futex
  0.00    0.000000           0         1           epoll_create
  0.00    0.000000           0         8           getdents64
  0.00    0.000000           0         1           set_tid_address
  0.00    0.000000           0        31         4 openat
  0.00    0.000000           0        13           newfstatat
  0.00    0.000000           0         1           set_robust_list
  0.00    0.000000           0         3           prlimit64
  0.00    0.000000           0         1           getrandom
  0.00    0.000000           0         1           rseq
------ ----------- ----------- --------- --------- ----------------
100.00    0.000000           0       292        18 total
```

🌞 **HARDEN**

```bash
[Unit]
Description=The nginx HTTP and reverse proxy server
After=network-online.target remote-fs.target nss-lookup.target
Wants=network-online.target

[Service]
Type=forking
PIDFile=/run/nginx.pid
# Nginx will fail to start if /run/nginx.pid already exists but has the wrong
# SELinux context. This might happen when running `nginx -t` from the cmdline.
# https://bugzilla.redhat.com/show_bug.cgi?id=1268621
ExecStartPre=/usr/bin/rm -f /run/nginx.pid
ExecStartPre=/usr/sbin/nginx -t
ExecStart=/usr/sbin/nginx
ExecReload=/usr/sbin/nginx -s reload
KillSignal=SIGQUIT
TimeoutStopSec=5
KillMode=mixed
PrivateTmp=true
SystemCallFilter=accept4 access arch_prctl bind brk clone close connect dup2 epoll_create epoll_create1 epoll_ctl epoll_wait eventfd2 execve exit_group fcntl fstat futex getdents64 geteuid getpid getppid getrandom gettid ioctl io_setup listen lseek mkdir mmap mprotect munmap newfstatat openat prctl pread64 pwrite64 read recvfrom recvmsg rseq rt_sigaction rt_sigprocmask rt_sigreturn rt_sigsuspend sendfile sendmsg sendto setgid setgroups setitimer set_robust_list setsid setsockopt set_tid_address setuid socket socketpair statfs sysinfo timerfd_create timerfd_settime umask uname unlink unlinkat wait4 write writev

[Install]
WantedBy=multi-user.target
```