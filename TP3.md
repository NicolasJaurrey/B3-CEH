# I. Service SSH

> Pour rappel : un *service* c'est juste un programme lancé par l'OS à notre place. Donc quand on dit qu'un "service tourne", ça veut concrètement dire qu'il y a un programme en cours d'exécution.

Le service SSH est déjà installé sur la machine, et il est aussi déjà démarré. C'est par défaut sur Rocky.

> *En effet Rocky c'est un OS qui est spécialisé pour faire tourner des serveurs. Pas choquant d'avoir un serveur SSH préinstallé et dispo dès l'installation pour pouvoir s'y connecter à distance ! Ce serait pas le cas sur un OS Ubuntu (par exemple) que vous installeriez sur votre PC. C'est pas le cas non plus sur un Windows 11 ou un MacOS ou un Android ouuuu... bref t'as capté. Mais sur n'importe lequel de ces OS, on peut **ajouter** un service SSH si on le souhaite.*

- [I. Service SSH](#i-service-ssh)
  - [1. Analyse du service](#1-analyse-du-service)
  - [2. Modification du service](#2-modification-du-service)

## 1. Analyse du service

On va, dans cette première partie, analyser le service SSH qui est en cours d'exécution.

🌞 **S'assurer que le service `sshd` est démarré**



```bash
[njaure@localhost ~]$ systemctl status sshd
● sshd.service - OpenSSH server daemon
     Loaded: loaded (/usr/lib/systemd/system/sshd.service; enabled; preset: enabled)
     Active: active (running) since Tue 2024-09-10 14:55:43 CEST; 32min ago
       Docs: man:sshd(8)
             man:sshd_config(5)
   Main PID: 710 (sshd)
      Tasks: 1 (limit: 11112)
     Memory: 4.8M
        CPU: 46ms
     CGroup: /system.slice/sshd.service
             └─710 "sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups"

Sep 10 14:55:43 localhost.localdomain systemd[1]: Starting OpenSSH server daemon...
Sep 10 14:55:43 localhost.localdomain sshd[710]: Server listening on 0.0.0.0 port 22.
Sep 10 14:55:43 localhost.localdomain sshd[710]: Server listening on :: port 22.
Sep 10 14:55:43 localhost.localdomain systemd[1]: Started OpenSSH server daemon.
Sep 10 14:56:29 localhost.localdomain sshd[1289]: Accepted password for njaure from 10.3.1.12 port 51956 ssh2
Sep 10 14:56:29 localhost.localdomain sshd[1289]: pam_unix(sshd:session): session opened for user njaure(uid=1000) by njaure(uid=0)
```

🌞 **Analyser les processus liés au service SSH**


```bash
# Exemple de manipulation de | grep

# admettons un fichier texte appelé "fichier_demo"
# on peut afficher son contenu avec la commande cat :
$ cat fichier_demo
bob a un chapeau rouge
emma surfe avec un dinosaure
eve a pas toute sa tête

# il est possible de filtrer la sortie de la commande cat pour afficher uniquement certaines lignes
$ cat fichier_demo | grep emma
emma surfe avec un dinosaure

$ cat fichier_demo | grep bob
bob a un chapeau rouge
```


```bash
[njaure@localhost ~]$ sudo systemctl list-units -t service -a | grep sshd
[sudo] password for njaure: 
  sshd-keygen@ecdsa.service              loaded    inactive dead    OpenSSH ecdsa Server Key Generation
  sshd-keygen@ed25519.service            loaded    inactive dead    OpenSSH ed25519 Server Key Generation
  sshd-keygen@rsa.service                loaded    inactive dead    OpenSSH rsa Server Key Generation
  sshd.service                           loaded    active   running OpenSSH server daemon
```

🌞 **Déterminer le port sur lequel écoute le service SSH**


```bash
[njaure@localhost ~]$ sudo ss -lpt | grep ssh
tcp   LISTEN 0      128                                       0.0.0.0:22                      0.0.0.0:*          
tcp   LISTEN 0      128                                          [::]:22                         [::]:* 
```
🌞 **Consulter les logs du service SSH**



```bash
  [njaure@localhost ~]$ journalctl -u sshd
Sep 10 14:55:43 localhost.localdomain systemd[1]: Starting OpenSSH server daemon...
Sep 10 14:55:43 localhost.localdomain sshd[710]: Server listening on 0.0.0.0 port 22.
Sep 10 14:55:43 localhost.localdomain sshd[710]: Server listening on :: port 22.
Sep 10 14:55:43 localhost.localdomain systemd[1]: Started OpenSSH server daemon.
Sep 10 14:56:29 localhost.localdomain sshd[1289]: Accepted password for njaure from 10.3.1.12 port 51956 ssh2
Sep 10 14:56:29 localhost.localdomain sshd[1289]: pam_unix(sshd:session): session opened for user njaure(uid=1000) by njaure(uid=0)
```



```bash
  [njaure@localhost ~]$ tail /var/log/lastlog
�A�fpts/010.3.1.12
```

![When she tells you](./img/when_she_tells_you.png)

## 2. Modification du service

Dans cette section, on va aller visiter et modifier le fichier de configuration du serveur SSH.

Comme tout fichier de configuration, celui de SSH se trouve dans le dossier `/etc/`.

Plus précisément, il existe un sous-dossier `/etc/ssh/` qui contient toute la configuration relative à SSH

🌞 **Identifier le fichier de configuration du serveur SSH**

 ```bash
 [njaure@localhost ~]$ ls /etc/ssh/sshd_config
/etc/ssh/sshd_config
```

🌞 **Modifier le fichier de conf**



```bash
[njaure@localhost ~]$ echo $RANDOM /etc/ssh/sshd_config
31243 /etc/ssh/sshd_config
```



  ```bash
  [njaure@localhost ~]$ cat /etc/ssh/sshd_config | grep Port
  Port 4096
  ```

  ```bash
  [njaure@localhost ~]$ sudo firewall-cmd --remove-port=22/tcp
  Warning: NOT_ENABLED: '22:tcp'
  success
  ```

  ```bash
  [njaure@localhost ~]$ sudo firewall-cmd --add-port=4096/tcp
  success
  ```
 

  ```bash
  [njaure@localhost ~]$ sudo firewall-cmd --list-all | grep 4096
  ports: 4096/tcp
  ```

🌞 **Redémarrer le service**



```bash
[njaure@localhost ~]$ sudo firewall-cmd --reload
success
```

🌞 **Effectuer une connexion SSH sur le nouveau port**



```bash
njboulot@njboulot-LOQ-15IRH8:~$ ssh -p 4096 njaure@10.3.1.11
njaure@10.3.1.11's password: 
Last login: Wed Sep 11 13:31:18 2024
[njaure@localhost ~]$ 
```

> Je vous conseille de remettre le port par défaut une fois que cette partie est terminée.

✨ **Bonus : affiner la conf du serveur SSH**

- faites vos plus belles recherches internet pour améliorer la conf de SSH
- par "améliorer" on entend essentiellement ici : augmenter son niveau de sécurité
- le but c'est pas de me rendre 10000 lignes de conf que vous pompez sur internet pour le bonus, mais de vous éveiller à divers aspects de SSH, la sécu ou d'autres choses liées

![Such a hacker](./img/such_a_hacker.png)
# II. Service HTTP

**Dans cette partie, on ne va pas se limiter à un service déjà présent sur la machine : on va ajouter un service à la machine.**

➜ **On va faire dans le *clasico* et installer un serveur HTTP très réputé : NGINX**

Un serveur HTTP permet d'héberger des sites web.

➜ **Un serveur HTTP (ou "serveur Web") c'est :**

- un programme qui écoute sur un port (ouais ça change pas ça)
- il permet d'héberger des sites web
  - un site web c'est un tas de pages html, js, css
  - un site web c'est aussi parfois du code php, go, python, ou autres, qui indiquent comment le site doit se comporter
- il permet à des clients de visiter les sites web hébergés
  - pour ça, il faut un client HTTP (par exemple, un navigateur web, ou la commande `curl`)
  - le client peut alors se connecter au port du serveur (connu à l'avance)
  - une fois le tunnel de communication établi, le client effectuera des *requêtes HTTP*
  - le serveur répondra par des *réponses HTTP*

> Une requête HTTP c'est "donne moi tel fichier HTML". Une réponse c'est "voici tel fichier HTML" + le fichier HTML en question.

**Ok bon on y va ?**

- [II. Service HTTP](#ii-service-http)
  - [1. Mise en place](#1-mise-en-place)
  - [2. Analyser la conf de NGINX](#2-analyser-la-conf-de-nginx)
  - [3. Déployer un nouveau site web](#3-déployer-un-nouveau-site-web)

## 1. Mise en place

![nngijgingingingijijnx ?](./img/njgjgijigngignx.jpg)

> Si jamais, pour la prononciation, NGINX ça vient de "engine-X" et vu que c'était naze comme nom... ils ont choisi un truc imprononçable ouais je sais cherchez pas la logique.

🌞 **Installer le serveur NGINX**


```bash
[njaure@localhost ~]$ sudo dnf install nginx
```


```bash
[njaure@localhost ~]$ dnf search nginx
```

🌞 **Démarrer le service NGINX**

```bash
[njaure@localhost ~]$ sudo service nginx start
Redirecting to /bin/systemctl start nginx.service
```

🌞 **Déterminer sur quel port tourne NGINX**

```bash
[njaure@localhost ~]$ cat /etc/nginx/nginx.conf | grep listen
        listen       80;
        listen       [::]:80;
#        listen       443 ssl http2;
#        listen       [::]:443 ssl http2;
```

🌞 **Déterminer les processus liés au service NGINX**



```bash
[njaure@localhost ~]$ ps aux | grep nginx |grep -v grep
root        1934  0.0  0.0  10116  1440 ?        Ss   16:00   0:00 nginx: master process /usr/sbin/nginx
nginx       1935  0.0  0.2  14352  4896 ?        S    16:00   0:00 nginx: worker process
```

🌞 **Déterminer le nom de l'utilisateur qui lance NGINX**


```bash 
 [njaure@localhost ~]$ ps aux | grep nginx
njaure      1575  0.0  0.1   6408  2048 pts/0    S+   18:24   0:00 grep --color=auto nginx
```

🌞 **Test !**


```bash
[njaure@localhost ~]$ curl http://127.0.0.1 | head
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0<!doctype html>
<html>
  <head>
    <meta charset='utf-8'>
    <meta name='viewport' content='width=device-width, initial-scale=1'>
    <title>HTTP Server Test Page powered by: Rocky Linux</title>
    <style type="text/css">
      /*<![CDATA[*/
      
      html {
100  7620  100  7620    0     0   744k      0 --:--:-- --:--:-- --:--:--  744k
curl: (23) Failed writing body

```

## 2. Analyser la conf de NGINX

🌞 **Déterminer le path du fichier de configuration de NGINX**


```bash
[njaure@localhost ~]$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

🌞 **Trouver dans le fichier de conf**

```bash
  [njaure@localhost ~]$ cat /etc/nginx/nginx.conf | grep include
include /usr/share/nginx/modules/*.conf;
    include             /etc/nginx/mime.types;
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    include /etc/nginx/conf.d/*.conf;
        include /etc/nginx/default.d/*.conf;
#        include /etc/nginx/default.d/*.conf;
```

## 3. Déployer un nouveau site web

🌞 **Créer un site web**



```bash
[njaure@localhost ~]$ sudo mkdir -p /var/www/tp3_linux
```

- dans ce dossier `/var/www/tp3_linux`, créez un fichier `index.html`
  - il doit contenir `<h1>MEOW mon premier serveur web</h1>`

```bash
[njaure@localhost ~]$ curl http://10.3.1.11
<!DOCTYPE html>
<html>
<head>
    <h1>MEOW mon premier serveur web<h1>
</head>
</html>
```

🌞 **Gérer les permissions**

```bash
[njaure@localhost ~]$ ls -al /var/www/tp3_linux/
total 4
drwxr-xr-x. 2 njaure njaure 24 Sep 15 22:47 .
drwxr-xr-x. 3 root   root   23 Sep 15 22:42 ..
-rwxr-xr-x. 1 njaure njaure 87 Sep 15 22:53 index.html
```

🌞 **Adapter la conf NGINX**

```nginx
server {
  # le port choisi devra être obtenu avec un 'echo $RANDOM' là encore
  listen 1567;

  root /var/www/tp3_linux;
}
```

🌞 **Visitez votre super site web**



```bash
[njaure@localhost ~]$ curl http://10.3.1.11:1567
<!DOCTYPE html>
<html>
<head>
    <h1>MEOW mon premier serveur web<h1>
</head>
</html>
```


# III. Your own services

Dans cette partie, on va créer notre propre service :)

HE ! Vous vous souvenez de `netcat` ou `nc` ? Le ptit machin de notre premier cours de réseau ? C'EST L'HEURE DE LE RESORTIR DES PLACARDS.

- [III. Your own services](#iii-your-own-services)
  - [1. Au cas où vous l'auriez oublié](#1-au-cas-où-vous-lauriez-oublié)
  - [2. Analyse des services existants](#2-analyse-des-services-existants)
  - [3. Création de service](#3-création-de-service)

## 1. Au cas où vous l'auriez oublié

> Rien à écrire dans le rendu pour cette partie, c'juste pour vous remettre `nc` en main.

➜ Dans la VM

- `nc -l 8888`
  - lance netcat en mode listen
  - il écoute sur le port 8888
  - sans rien préciser de plus, c'est le port 8888 TCP qui est utilisé

➜ Connectez-vous au netcat qui est en écoute sur la VM

- depuis votre PC ou depuis une autre VM (il faut avoir la commande `nc`)
- `nc <IP_PREMIERE_VM> 8888`
- vérifiez que vous pouvez envoyer des messages dans les deux sens

> N'oubliez pas d'ouvrir le port 8888/tcp de la première VM bien sûr :)

## 2. Analyse des services existants



🌞 **Afficher le fichier de service SSH**



  ```bash
  [njaure@localhost ~]$ cat /usr/lib/systemd/system/sshd.service | grep ExecStart=
  ExecStart=/usr/sbin/sshd -D $OPTIONS
  ```
  


🌞 **Afficher le fichier de service NGINX**



```bash
# Comme celle juste avant
```

## 3. Création de service

![Create service](./img/create_service.png)

Bon ! On va créer un petit service qui lance un `nc`. Et vous allez tout de suite voir pourquoi c'est pratique d'en faire un service et pas juste le lancer à la main : on va faire en sorte que `nc` se relance tout seul quand un client se déconnecte.

> *Le comportement de base de `nc` c'est de quitter, de se fermer, si un utilisateur se connecte puis s'en va.*

Ca reste un truc pour s'exercer, c'pas non plus le truc le plus utile de l'année que de mettre un `nc` dans un service n_n

🌞 **Créez le fichier `/etc/systemd/system/tp3_nc.service`**



```service
[Unit]
Description=Super netcat tout fou

[Service]
ExecStart=/usr/bin/nc -l 5225 -k
```

🌞 **Indiquer au système qu'on a modifié les fichiers de service**


```bash
[njaure@localhost ~]$ sudo systemctl daemon-reload
```

🌞 **Démarrer notre service de ouf**

```bash
[njaure@localhost ~]$ sudo systemctl start tp3_nc.service
```

🌞 **Vérifier que ça fonctionne**



```bash
[njaure@localhost ~]$ sudo systemctl status tp3_nc.service
● tp3_nc.service - Super netcat tout fou
     Loaded: loaded (/etc/systemd/system/tp3_nc.service; static)
     Active: active (running) since Sun 2024-09-15 23:20:14 CEST; 32s ago
   Main PID: 1950 (nc)
      Tasks: 1 (limit: 11112)
     Memory: 2.3M
        CPU: 3ms
     CGroup: /system.slice/tp3_nc.service
             └─1950 /usr/bin/nc -l 5225 -k

Sep 15 23:20:14 localhost.localdomain systemd[1]: Started Super netcat to>
...skipping...
```

```bash
[sudo] password for njaure: 
tcp   LISTEN 0      10           0.0.0.0:5225      0.0.0.0:*          
tcp   LISTEN 0      10              [::]:5225         [::]:*
```



🌞 **Les logs de votre service**

- mais euh, ça s'affiche où les messages envoyés par le client ? Dans les logs !
- `sudo journalctl -xe -u tp3_nc` pour visualiser les logs de votre service
- `sudo journalctl -xe -u tp3_nc -f` pour visualiser **en temps réel** les logs de votre service
  - `-f` comme follow (on "suit" l'arrivée des logs en temps réel)
- dans le compte-rendu je veux
  - une commande `journalctl` filtrée avec `grep` qui affiche la ligne qui indique le démarrage du service
  - une commande `journalctl` filtrée avec `grep` qui affiche un message reçu qui a été envoyé par le client
  - une commande `journalctl` filtrée avec `grep` qui affiche la ligne qui indique l'arrêt du service

🌞 **S'amuser à `kill` le processus**



```bash
[njaure@localhost ~]$ sudo systemctl status tp3_nc.service
○ tp3_nc.service - Super netcat tout fou
     Loaded: loaded (/etc/systemd/system/tp3_nc.service; static)
     Active: inactive (dead)

Sep 15 23:20:14 localhost.localdomain systemd[1]: Started Super netcat to>
Sep 15 23:39:28 localhost.localdomain systemd[1]: tp3_nc.service: Deactiv>
```


🌞 **Affiner la définition du service**



```bash
[njaure@localhost ~]$ sudo systemctl status tp3_nc.service
● tp3_nc.service - Super netcat tout fou
     Loaded: loaded (/etc/systemd/system/tp3_nc.service; enabled; preset:>
     Active: active (running) since Sun 2024-09-15 23:44:32 CEST; 12s ago
   Main PID: 2109 (nc)
      Tasks: 1 (limit: 11112)
     Memory: 796.0K
        CPU: 3ms
     CGroup: /system.slice/tp3_nc.service
             └─2109 /usr/bin/nc -l 5225 -k

Sep 15 23:44:32 localhost.localdomain systemd[1]: Started Super netcat to>
```

