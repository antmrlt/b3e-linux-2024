🌞 **Afficher l'identifiant du processus serveur OpenSSH en cours d'exécution**

```
[antna@node1 ~]$ systemctl status sshd | grep PID
   Main PID: 1835 (sshd)
```

🌞 **Changer le port d'écoute du serveur OpenSSH**

Dans le fichier `/etc/ssh/sshd_config
```
Port 222
```

```
[antna@node1 ~]$ ss -tulnep
Netid         State           Recv-Q          Send-Q                   Local Address:Port                   Peer Address:Port         Process
udp           UNCONN          0               0                            127.0.0.1:323                         0.0.0.0:*             ino:21278 sk:44 cgroup:/system.slice/chronyd.service <->
udp           UNCONN          0               0                                [::1]:323                            [::]:*             ino:21279 sk:45 cgroup:/system.slice/chronyd.service v6only:1 <->
tcp           LISTEN          0               128                            0.0.0.0:2222                        0.0.0.0:*             ino:56777 sk:46 cgroup:/system.slice/sshd.service <->
tcp           LISTEN          0               128                               [::]:2222                           [::]:*             ino:56792 sk:47 cgroup:/system.slice/sshd.service v6only:1 <->
```
- expliquez pourquoi on considère parfois utile de changer le port d'écoute par défaut du serveur SSH
Pour contrer les scan auto des robots

🌞 **Configurer une authentification par clé**

Je prend ma clee publique dans `C:Utilisateurs\antoine\.ssh\id_rsa.pub` et la met dans `.ssh/autorized_keys`.

🌞 **Désactiver la connexion par password**

Dans `/etc/ssh/sshd_config`
```
PasswordAuthentication no
```
Et on restart

🌞 **Désactiver la connexion en tant que `root`**

```
PermitRootLogin no
```

⭐ **BONUS** : **Configurer une authentification par certificat**

- j'ai dit par certificat, pas par simple clé
- pareil, faites-le avec votre utilisateur pour les tests

> L'authentification par certificat est toujours plus forte que l'authentification par simple clé : les deux parties (typiquement, le client et le serveur) doivent prouver l'identité à l'autre. De plus, le certificat ne peut pas être falsifié, du moins si on utilise une autorité de certification digne de confiance. L'idée du certificat : on va signer la clé du client avec la clé d'une autorité de certification. Ainsi, la clé n'est plus falsifiable, l'autorité de certification peut attester que c'est la bonne clé pour le bon client.

🌞 **Proposer au moins 5 configurations supplémentaires qui permettent de renforcer la sécurité du serveur OpenSSH**

```
PermitEmptyPasswords no
LoginGraceTime 20*
MaxAuthTries 3
MaxSessions 1
AllowAgentForwarding no
AllowTcpForwarding no
X11Forwarding no
PermitTunnel no
```

https://ittavern.com/ssh-server-hardening/

🌞 **Installer fail2ban sur la machine**

```
[antna@node1 ~]$ sudo dnf install fail2ban
```

🌞 **Configurer fail2ban**

```
[sshd]
enabled = true
maxretry = 7
findtime = 300
bantime = 3600
```

🌞 **Prouvez que fail2ban est effectif**

```
[antna@node1 ~]$ sudo fail2ban-client status sshd
Status for the jail: sshd
|- Filter
|  |- Currently failed: 1
|  |- Total failed:     12
|  `- Journal matches:  _SYSTEMD_UNIT=sshd.service + _COMM=sshd + _COMM=sshd-session
`- Actions
   |- Currently banned: 1
   |- Total banned:     1
   `- Banned IP list:   10.1.1.1

[antna@node1 ~]$ sudo fail2ban-client set sshd unbanip 10.1.1.1
1
```

## 6. Automatisation

Dernière section : un peu de dév en bash pour automatiser toute la configuration que vous venez de faire.

L'idée est simple : écrire un script shell qui applique la configuration de cette Partie V (openSSH et fail2ban) sur une machine Rocky Linux fraîchement installée.

🌞 **Ecrire le script `harden.sh`**

```bash
#!/bin/bash

#openssh
echo '[+] Checking if openssh is running...'

if systemctl is-active --quiet sshd;
then
    echo " - openssh is running"
else
    echo " - opnssh is not running"
    exit 2
fi

# ssh port
SSH_PORT=$(sudo cat /etc/ssh/sshd_config | grep -w Port | cut -d' ' -f2)
echo '[+] Checking if ssh is using port 22...'

if [ "$SSH_PORT" == "22" ]; then
    echo " - /!\ it is"
else
    echo " - it is not, using $SSH_PORT"
fi

# fail2ban
echo '[+] Checking if fail2ban is running...'

if systemctl is-active --quiet sshd;
then
    echo " - fail2ban is running"
    echo '[+] Checking if fail2ban is monitoring openssh logs...'
        if sudo fail2ban-client status sshd &>/dev/null; then
            echo " - it is"
        else
            echo " - it is not"
    fi
else
    echo " - fail2ban is not running"
    exit 2
fi
```
 