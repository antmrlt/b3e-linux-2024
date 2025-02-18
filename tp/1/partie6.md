🌞 **Installer NGINX**

```
[antna@node1 ~]$ sudo systemctl start nginx
[antna@node1 ~]$ sudo firewall-cmd --add-port=80/tcp
success
```

🌞 **Prouver que le serveur Web est fonctionnel**

```
[antna@node1 ~]$ curl localhost
<!doctype html>
<html>
  <head>
    <meta charset='utf-8'>
    <meta name='viewport' content='width=device-width, initial-scale=1'>
    <title>HTTP Server Test Page powered by: Rocky Linux</title>
    <style type="text/css">
      /*<![CDATA[*/

      html {
        height: 100%;
        width: 100%;
        ...
```

🌞 **Créer un nouveau site web**

On crée un user et un group en amont
```
[antna@node1 ~]$ sudo chown webuser:webgroup /var/www/super_site
[antna@node1 ~]$ sudo chmod 060 /var/www/super_site
[antna@node1 ~]$ sudo chmod 060 /var/www/super_site/index.html
```

```
server {
    listen 80;
    server_name super-site.local;

    root /var/www/super_site;
    index index.html;

    location / {
        root /var/www/super_site;
    }
}
```

🌞 **Prouvez que le nouveau site web peut être visité**

```
[bingo@node1 ~]$ curl http://super-site.local
<h1>Coucou</h1>
```

## 2. Reverse proxy

🌞 **Installer NGINX**

```
[antna@node1 ~]$ sudo systemctl start nginx
[antna@node1 ~]$ sudo firewall-cmd --add-port=80/tcp
success
```

🌞 **Proposer une configuration minimale**

- NGINX doit agir comme reverse proxy pour accéder au site web de la première VM
- 

🌞 **Générer une clé et un certificat auto-signé**

- stockez-les à un endroit standard
- dans Rocky Linux, il existe un dossier standard pour les clés, et un pour les certificats

🌞 **Ajuster la configuration du reverse proxy**

- il doit utiliser votre clé et votre certificat
- écoute sur le port 443
- propose une connexion avec TLS d'activé

## 4. Bonus : dear TLS

⭐ **Bonus : faites en sorte d'avoir une connexion vérifiée sur le client**

- on parle d'avoir un cadenas vert si on visite en navigateur
- vous pouvez utiliser la méthode de votre choix
- je vous recommande de créer une CA

⭐ **Bonus : harden**

- proposer de la configuration additionnelle pour renforcer la sécurité de la connexion à votre serveur Web
- ça peut toucher à TLS ou autre chose

⭐ **Bonus : mTLS**

- TLS ça peut aussi mTLS pour mutual TLS
- une extension du protocole qui permet au client de s'authentifier auprès du serveur (et pas seulement l'inverse)
- extrêmement fort pour limiter l'accès à un site web donné
- mettre en palce du mTLS sur le reverse proxy : le client doit posséder un certificat pour accéder à la page
