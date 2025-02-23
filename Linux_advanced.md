# Part I : Rocky install

Dans cette partie, vous allez simplement dérouler l'installation de Rocky, en suivant les instructions que je vous donne pour la conf.

Une fois l'installation terminée, vous pourrez éteindre la VM, et elle vous servira de base pour tous nos TPs : dès qu'un TP nécessite une VM, vous pourrez cloner celle-ci.

## Index

- [Part I : Rocky install](#part-i--rocky-install)
  - [Index](#index)
  - [1. Install instructions](#1-install-instructions)
  - [2. Proofs](#2-proofs)

## 1. Install instructions

➜ **Créer la VM dans votre hyperviseur avec :**

- un disque de 30G
- une carte réseau permettant un accès internet
- 1024M ou 2048M de RAM car l'installation est graphique (on peut tranquillement passer sur 512M de RAM après)

![Install Linux](./img/install.png)

➜ **L'OS doit être en anglais**

- allez j'ai même plus besoin d'argumenter ça, si ?
- n'oubliez pas de configurer une disposition clavier azerty (fin en fonction de votre clavier quoi)

➜ **Le fuseau horaire configuré doit être cohérent**

- nous c'est Europe/Paris

➜ **Une carte réseau permettant un accès internet doit être allumée**

- la carte NAT de VBox ça fait le taff

➜ **Utilisateur `root`**

- vous définissez un password que vous oubliez pas svp

➜ **Utilisateur administratif**

- vous vous créez un utilisateur, avec un mot de passe
- y'a une case à cocher pour "faire de cet utilisateur un administrateur", cochez-la

➜ **Partitionnement**

- vous configurez le schéma de partitionnement suivant :


| Point de montage | Taille       | FS    |
| ---------------- | ------------ | ----- |
| /                | 10G          | ext4  |
| /home            | 5G           | ext4  |
| /var             | 5G           | ext4  |
| swap             | 1G           | swap  |
| espace libre     | ce qui reste | aucun |

➜ **Lancez l'install !**

➜ **Une fois l'installation terminée**

- connectez-vous à la VM
- et effectuez les commandes suivantes :

```bash
# désactivation temporaire de SELinux
sudo setenforce 0

# désactivation définitive de SELinux
sudo sed -i 's/enforcing/permissive/' /etc/selinux/config

# mise à jour du système si besoin
sudo dnf update -y

# installation de paquets qu'on sera ptet amenés à use
sudo dnf install -y traceroute vim bind-utils tcpdump nano nc epel-release
```

➜ **Une fois que c'est fait, éteignez la VM et vous la rallumez jamais**

- vous cloner cette VM pour la suite du TP
- vous la clonerez dès que nécessaire si on a besoin de VM

## 2. Proofs

Pour répondre à chaque soleil, une seule ligne de commande suffit. Abusez des syntaxes `cat TRUC | grep TRUC` par exemple pour ne montrer que ce qui est demandé.

➜ **Cloner donc la VM qu'on vient d'installer, créez la machine `node1.tp1.b3`**

- et tu te connectes direct en SSH avec ton utilisateur (avec `root` c'est désactivé par défaut sous Rocky)
- sans SSH tu pourras pas faire de copier/coller, ~~ni pour copier/coller bêtement ce que te dit chatGPT, ni~~ pour écrire mon compte-rendu
- s'il faut tu configures une IP vitefé, j'donne [les instructions pour le faire à la partie 2](./part2.md) si tu as besoin de le faire now

> Appelez-moi si vous galérez à vous co en SSH, ~~bande de noobs.~~, il **faut** que ça fonctionne, c'est pas une option. Et il **faut** que tu comprennes que c'est pas une option et que c'est important, que c'est la base. Et il **faut** que tu maîtrises ça, je te le ré-expliquerai autant de fois que nécessaire.

🌞 **Prouvez que le schéma de partitionnement a bien été appliqué**

- genre les partitions qu'on a défini à l'install

``` [rockynj@node1 ~]$ lsblk
NAME                         MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                            8:0    0   30G  0 disk 
├─sda1                         8:1    0  500M  0 part /boot
└─sda2                         8:2    0   21G  0 part 
  ├─rl_efrei--xmg4agau1-root 253:0    0   10G  0 lvm  /
  ├─rl_efrei--xmg4agau1-swap 253:1    0    1G  0 lvm  [SWAP]
  ├─rl_efrei--xmg4agau1-var  253:2    0    5G  0 lvm  /var
  └─rl_efrei--xmg4agau1-home 253:3    0    5G  0 lvm  /home
sr0                           11:0    1 1024M  0 rom

```

- une commande qui affiche toutes les partitions en cours d'utilisation

``` [rockynj@node1 ~]$ df -h
Filesystem                            Size  Used Avail Use% Mounted on
devtmpfs                              4.0M     0  4.0M   0% /dev
tmpfs                                 888M     0  888M   0% /dev/shm
tmpfs                                 355M  5.0M  350M   2% /run
/dev/mapper/rl_efrei--xmg4agau1-root  9.8G  1.3G  8.0G  14% /
/dev/sda1                             436M  304M  133M  70% /boot
/dev/mapper/rl_efrei--xmg4agau1-var   4.9G  145M  4.5G   4% /var
/dev/mapper/rl_efrei--xmg4agau1-home  4.9G   44K  4.6G   1% /home
tmpfs                                 178M     0  178M   0% /run/user/1000
```

- ainsi que l'espace disponible sur chacune des partitions
```
[rockynj@node1 ~]$ df -h | tr -s " " | cut -d " " -f1,4
Filesystem Avail
devtmpfs 4.0M
tmpfs 888M
tmpfs 350M
/dev/mapper/rl_efrei--xmg4agau1-root 8.0G
/dev/sda1 133M
/dev/mapper/rl_efrei--xmg4agau1-var 4.5G
/dev/mapper/rl_efrei--xmg4agau1-home 4.6G
tmpfs 178M
```

➜ Par défaut, sous Rocky Linux :

- il existe un groupe appelé `wheel` déjà créé à l'installation
- le groupe `wheel` est déjà dans la conf `sudo` pour autoriser ses membres à utiliser les droits de `root` avec la commande `sudo`

🌞 **Mettre en évidence la ligne de configuration `sudo` qui concerne le groupe `wheel`**

- avec un `cat TRUC | grep TRUC` je veux voir que la bonne ligne

```
[rockynj@node1 ~]$ sudo cat /etc/sudoers | tr -s " " | cut -d " " -f2,3 | grep wheel
%wheel	ALL=(ALL)	ALL
%wheel	ALL=(ALL)	NOPASSWD: ALL
```

🌞 **Prouvez que votre utilisateur est bien dans le groupe `wheel`**

```
[rockynj@node1 ~]$ cat /etc/group | grep wheel
wheel:x:10:rockynj
```

🌞 **Prouvez que la langue configurée pour l'OS est bien l'anglais**

- je veux une ligne de commande qui affiche la langue actuelle de l'OS*

```
[rockynj@node1 ~]$ localectl | grep US
System Locale: LANG=en_US.UTF-8
```

- que vos messages d'erreur soient en anglais ça me suffit pas ;D

🌞 **Prouvez que le firewall est déjà actif**

- le service de firewalling s'appelle `firewalld` sous Rocky (on le manipule avec la commande `firewall-cmd`)

```
[rockynj@node1 ~]$ sudo firewall-cmd --state
running
```

# Part II : Networking

Le réseau c'est la porte d'entrée pour toutes les autres machines. C'est le seul moyen d'être attaqué à distance.

Maîtriser au mieux le réseau d'une machine est donc primordial pour prétendre en renforcer la sécurité.

## Index

- [Part II : Networking](#part-ii--networking)
  - [Index](#index)
  - [1. Basic networking conf](#1-basic-networking-conf)
    - [A. Static IP](#a-static-ip)
    - [B. Hostname](#b-hostname)
  - [2. Listening ports](#2-listening-ports)
  - [3. Firewalling](#3-firewalling)

## 1. Basic networking conf

### A. Static IP

🌞 **Attribuer l'adresse IP `10.1.1.11/24`** à la VM

- ça veut dire que votre PC a pour adresse IP `10.1.1.X/24` (il est dans le même réseau)
- je vous file les instructions pour la définition de l'IP dans la VM, avec Rocky Linux on peut faire comme ça pour la définition d'une IP statique :

```bash
# on commence par repérer le nom de la carte réseau à configurer
$ ip a # on suppose dans la suite qu'on veut configurer enp0s8


[rockynj@node1 ~]$ ip a | grep 10.1.1.11
inet 10.1.1.11/24 scope global enp0s8


# on se déplace dans le dossier de conf réseau
$ cd /etc/sysconfig/network-scripts

[rockynj@node1 ~]$ cd /etc/sysconfig/network-scripts
[rockynj@node1 network-scripts]$

# création d'un fichier qui porte dans son nom le nom de l'interface réseau
$ sudo nano ifcfg-enp0s8

[rockynj@node1 network-scripts]$ ls | grep enp0s
ifcfg-enp0s8

# le contenu du fichier est le suivant :
$ sudo cat ifcfg-enp0s8
DEVICE=enp0s8 # le nom de la carte
NAME=lan      # un nom arbitraire pas trop chiant à taper

BOOTPROTO=static # static ou dhcp
ONBOOT=yes       # la carte s'allume automatiquement au démarrage

IPADDR=10.1.1.11
NETMASK=255.255.255.0

[rockynj@node1 network-scripts]$ sudo cat ifcfg-enp0s8
DEVICE=enp0s8
NAME=lan   

BOOTPROTO=static
ONBOOT=yes      

IPADDR=10.1.1.11
NETMASK=255.255.255

# on indique à NetworkManager, le programme qui gère le réseau, qu'on a modifié la conf
$ sudo nmcli con reload

[rockynj@node1 network-scripts]$ sudo nmcli con reload

# on allume la carte réseau et applique la nouvelle conf
$ sudo nmcli con up lan # on réutilise le nom qu'on a mis dans le fichier

[rockynj@node1 network-scripts]$ sudo nmcli con up lan

```

### B. Hostname

🌞 **Attribuer le nom `node1.tp1.b3` à la VM**

- ça se fait avec une commande `hostnamectl` en 2025 svp

```
[rockynj@node1 network-scripts]$ sudo hostnamectl set-hostname node1.tp1.b3
```

## 2. Listening ports

🌞 **Déterminer la liste des programmes qui écoutent sur un port TCP**

```
[rockynj@node1 ~]$ ss -tlp
State                 Recv-Q                Send-Q                               Local Address:Port                               Peer Address:Port               Process               
LISTEN                0                     128                                        0.0.0.0:ssh                                     0.0.0.0:*                                        
LISTEN                0                     128                                           [::]:ssh                                        [::]:*
```

🌞 **Déterminer la liste des programmes qui écoutent sur un port UDP**

```
[rockynj@node1 ~]$ ss -ulp
State                 Recv-Q                Send-Q                               Local Address:Port                               Peer Address:Port               Process               
UNCONN                0                     0                                        127.0.0.1:323                                     0.0.0.0:*                                        
UNCONN                0                     0                                            [::1]:323                                        [::]:*
```

## 3. Firewalling

![fw](./img/fw.png)

➜ **Vous pouvez afficher l'état actuel de `firewalld`, le firewall de Rocky Linux, avec :**

```bash
sudo firewall-cmd --list-all
```

🌞 **Pour chacun des ports précédemment repérés...**

- montrez qu'il existe une règle firewall qui autorise le trafic entrant sur ce port
- ou pas ?
```
[rockynj@node1 ~]$ sudo firewall-cmd --list-all | grep ports
  ports:
  ```

> **Attention !** Le firewall de Rocky Linux, `firewalld`, a deux concepts pour ouvrir un port TCP/UDP. Soit on ouvre... un port avec `--add-port` et on le voit apparaître devant `ports:`. Soit on ouvre un "service" avec `--add-service` et on le voit apparaître devant `services:`. Chaque "service" est donc un port ouvert (et à fermer potentiellement à la question suivante ;) ).

🌞 **Fermez tous les ports inutilement ouverts dans le firewall**

- principe du moindre privilège encore et encore !
- pas besoin qu'un port soit ouvert si aucun service n'écoute dessus

[rockynj@node1 ~]$ sudo firewall-cmd --remove-service dhcpv6-client --permanent
success
[rockynj@node1 ~]$ sudo firewall-cmd --remove-service cockpit --permanent
success
[rockynj@node1 ~]$ sudo firewall-cmd --list-all | grep services
  services: cockpit dhcpv6-client ssh
[rockynj@node1 ~]$ sudo firewall-cmd --reload
success
[rockynj@node1 ~]$ sudo firewall-cmd --list-all | grep services
  services: ssh
[rockynj@node1 ~]$ 


🌞 **Pour toutes les applications qui sont en écoute sur TOUTES les adresses IP**

- dans Linux, ce sont les applications qui écoutent sur la pseudo-adresse IP `0.0.0.0` : ça signifie que toutes les adresses IP de la machine sont concernées
- modifier la configuration de l'application pour n'écouter que une seule IP : celle qui est nécessaire

```
[rockynj@node1 ~]$ sudo nano /etc/ssh/sshd_config
[rockynj@node1 ~]$ sudo systemctl reload sshd
[rockynj@node1 ~]$ ss -tlp
State                 Recv-Q                Send-Q                               Local Address:Port                               Peer Address:Port               Process               
LISTEN                0                     128                                      10.1.1.11:ssh                                     0.0.0.0:*                                        

```


# Part III : Storage is still disks in 2025

> Titre de la partie en référence au fait que le cloud est partout mais bon, c'est toujours des disques durs derrière ça n'a pas bougé. Un skill donc toujours primordial.

Dans cette partie, on joue avec le stockage de la machine. Au menu : mumuse avec des disques et des partitions, formatage, scénario de remplissage de partition.

> **Munissez vous du [mémo LVM](../../cours/memo/lvm.md) pour réaliser cette partie.**

## Index

- [Part III : Storage is still disks in 2025](#part-iii--storage-is-still-disks-in-2025)
  - [Index](#index)
  - [1. LVM](#1-lvm)
  - [2. HELP my partition is full](#2-help-my-partition-is-full)
  - [3. Prepare another partition](#3-prepare-another-partition)

![Partitions](./img/partition.png)

## 1. LVM

*LVM* (pour *Logical Volume Manager*) est l'outil de référence aujourd'hui sous Linux pour créer et gérer les partitions des disques.

> Il a beaucoup beaucoup trop de features de fou, il se contente pas de couper des disques !

🌞 **Afficher l'état actuel de LVM**

- afficher la liste des *PV* (*Volume Volumes*)
  - ce sont les disque durs et partitions physiques que LVM gère

```
[rockynj@node1 ~]$ lsblk | grep lvm
  ├─rl_efrei--xmg4agau1-root 253:0    0   10G  0 lvm  /
  ├─rl_efrei--xmg4agau1-swap 253:1    0    1G  0 lvm  [SWAP]
  ├─rl_efrei--xmg4agau1-var  253:2    0    5G  0 lvm  /var
  └─rl_efrei--xmg4agau1-home 253:3    0    5G  0 lvm  /home
```

- afficher la liste des *VG* (*Volume Groups*)
  - on regroupe les *PV* en des groupes appelés *VG*

```
[rockynj@node1 ~]$ sudo vgdisplay
  --- Volume group ---
  VG Name               rl_efrei-xmg4agau1
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  5
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                4
  Open LV               4
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               21.00 GiB
  PE Size               4.00 MiB
  Total PE              5377
  Alloc PE / Size       5376 / 21.00 GiB
  Free  PE / Size       1 / 4.00 MiB
  VG UUID               LE9S74-oTfB-RSWd-DPX8-r8Is-BlNN-RbKHdf
```

- afficher la liste des *LV* (*Logical Volumes*)
  - les *VG* sont découpés en *LV*
  - un *LV* est une partition utilisable

  ```
  [rockynj@node1 ~]$ sudo lvdisplay
  --- Logical volume ---
  LV Path                /dev/rl_efrei-xmg4agau1/var
  LV Name                var
  VG Name                rl_efrei-xmg4agau1
  LV UUID                tLNyNy-UsTY-frau-8g47-n3YQ-Wt7o-wWQeRO
  LV Write Access        read/write
  LV Creation host, time efrei-xmg4agau1.etudiants.campus.villejuif, 2025-02-17 12:27:17 +0100
  LV Status              available
  # open                 1
  LV Size                5.00 GiB
  Current LE             1280
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:2
   
  --- Logical volume ---
  LV Path                /dev/rl_efrei-xmg4agau1/root
  LV Name                root
  VG Name                rl_efrei-xmg4agau1
  LV UUID                gO1A1Q-BH0q-5chR-0Vsl-pl3R-pDtb-jw5gB0
  LV Write Access        read/write
  LV Creation host, time efrei-xmg4agau1.etudiants.campus.villejuif, 2025-02-17 12:27:17 +0100
  LV Status              available
  # open                 1
  LV Size                10.00 GiB
  Current LE             2560
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0
   
  --- Logical volume ---
  LV Path                /dev/rl_efrei-xmg4agau1/home
  LV Name                home
  VG Name                rl_efrei-xmg4agau1
  LV UUID                Vi0KGg-2tGf-DZus-4MFJ-tDYY-7Kd4-dOkH0f
  LV Write Access        read/write
  LV Creation host, time efrei-xmg4agau1.etudiants.campus.villejuif, 2025-02-17 12:27:17 +0100
  LV Status              available
  # open                 1
  LV Size                5.00 GiB
  Current LE             1280
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:3
   
  --- Logical volume ---
  LV Path                /dev/rl_efrei-xmg4agau1/swap
  LV Name                swap
  VG Name                rl_efrei-xmg4agau1
  LV UUID                3KE40K-rUJB-Blse-K4Ob-EMMi-OePT-BE9Jeo
  LV Write Access        read/write
  LV Creation host, time efrei-xmg4agau1.etudiants.campus.villejuif, 2025-02-17 12:27:17 +0100
  LV Status              available
  # open                 2
  LV Size                1.00 GiB
  Current LE             256
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:1
  ```



🌞 **Déterminer le type de système de fichiers**

- de la partition montée sur `/`
- de la partition montée sur `/home`
- **attention** : 
  - j'attends une commande qui détecte le type de système de fichiers sur une partition donnée : `<COMMANDE> /dev/chemin/partition`
  - je ne VEUX PAS une commande qui affiche les partitions actuellement utilisées où on voit le système de fichiers utilisé (pas de `mount` par exemple)
  
```
[rockynj@node1 ~]$ fsck -N /dev/rl_efrei-xmg4agau1/root
fsck from util-linux 2.37.4
[/usr/sbin/fsck.ext4 (1) -- /] fsck.ext4 /dev/mapper/rl_efrei--xmg4agau1-root 

```



## 2. HELP my partition is full


🌞 **Remplissez votre partition `/home`**

- on va simuler avec un truc bourrin :

```
dd if=/dev/zero of=/home/<TON_USER>/bigfile bs=4M count=2500
```

```
[rockynj@node1 ~]$ dd if=/dev/zero of=/home/rockynj/bigfile bs=4M count=2500
dd: error writing '/home/rockynj/bigfile': No space left on device
1171+0 records in
1170+0 records out
4911112192 bytes (4.9 GB, 4.6 GiB) copied, 3.29899 s, 1.5 GB/s
```

> 2500x4M ça fait 20G. Ca fait trop.

🌞 **Constater que la partition est pleine**

- avec un `df -h`

```
[rockynj@node1 ~]$ df -h | grep home
/dev/mapper/rl_efrei--xmg4agau1-home  4.9G  4.6G     0 100% /home
```

🌞 **Agrandir la partition**

- avec des commandes LVM il faut agrandir le logical volume

```
[rockynj@node1 ~]$ df -h
Filesystem                            Size  Used Avail Use% Mounted on
devtmpfs                              4.0M     0  4.0M   0% /dev
tmpfs                                 229M     0  229M   0% /dev/shm
tmpfs                                  92M  2.5M   89M   3% /run
/dev/mapper/rl_efrei--xmg4agau1-root  9.8G  1.3G  8.0G  14% /
/dev/sda1                             436M  304M  133M  70% /boot
/dev/mapper/rl_efrei--xmg4agau1-home  4.9G  4.6G  4.0K 100% /home
/dev/mapper/rl_efrei--xmg4agau1-var   4.9G  213M  4.4G   5% /var
tmpfs                                  46M     0   46M   0% /run/user/1000

[rockynj@node1 ~]$ sudo vgextend rl_efrei-xmg4agau1 /dev/sda3
  Volume group "rl_efrei-xmg4agau1" successfully extended


[rockynj@node1 ~]$ sudo lvextend -L+5G /dev/mapper/rl_efrei--xmg4agau1-home
  Size of logical volume rl_efrei-xmg4agau1/home changed from 5.00 GiB (1280 extents) to 10.00 GiB (2560 extents).
  Logical volume rl_efrei-xmg4agau1/home successfully resized.


```

- ensuite il faudra indiquer au système de fichier ext4 que la partition a été agrandie

```
[rockynj@node1 ~]$ sudo resize2fs /dev/mapper/rl_efrei--xmg4agau1-home
resize2fs 1.46.5 (30-Dec-2021)
Filesystem at /dev/mapper/rl_efrei--xmg4agau1-home is mounted on /home; on-line resizing required
old_desc_blocks = 1, new_desc_blocks = 2
The filesystem on /dev/mapper/rl_efrei--xmg4agau1-home is now 2621440 (4k) blocks long.
```

- prouvez avec un `df -h` que vous avez récupéré de l'espace en plus

```
[rockynj@node1 ~]$ df -h
Filesystem                            Size  Used Avail Use% Mounted on
devtmpfs                              4.0M     0  4.0M   0% /dev
tmpfs                                 229M     0  229M   0% /dev/shm
tmpfs                                  92M  2.5M   89M   3% /run
/dev/mapper/rl_efrei--xmg4agau1-root  9.8G  1.3G  8.0G  14% /
/dev/sda1                             436M  304M  133M  70% /boot
/dev/mapper/rl_efrei--xmg4agau1-home  9.8G  4.6G  4.8G  50% /home
/dev/mapper/rl_efrei--xmg4agau1-var   4.9G  213M  4.4G   5% /var
tmpfs                                  46M     0   46M   0% /run/user/1000
```

🌞 **Remplissez votre partition `/home`**

- on va simuler encore avec un truc bourrin :

```
dd if=/dev/zero of=/home/<TON_USER>/bigfile bs=4M count=2500
```

> 2500x4M ça fait toujours 20G. Et ça fait toujours trop.

➜ **Eteignez la VM et ajoutez lui un disque de 40G**

🌞 **Utiliser ce nouveau disque pour étendre la partition `/home` de 20G**

- dans l'ordre il faut :
- indiquer à LVM qu'il y a un nouveau PV dispo
```
[rockynj@node1 ~]$ sudo pvcreate /dev/sdb
  Physical volume "/dev/sdb" successfully created.

[rockynj@node1 ~]$ sudo pvdisplay | grep sdb
  "/dev/sdb" is a new physical volume of "40.00 GiB"
  PV Name               /dev/sdb
```
- ajouter ce nouveau PV au VG existant
```
[rockynj@node1 ~]$ sudo vgextend rl_efrei-xmg4agau1 /dev/sdb
  Volume group "rl_efrei-xmg4agau1" successfully extended
```
- étendre le LV existant pour récupérer le nouvel espace dispo au sein du VG

```
[rockynj@node1 ~]$ sudo vgextend rl_efrei-xmg4agau1 /dev/sdb1
  Physical volume "/dev/sdb1" successfully created.
  Volume group "rl_efrei-xmg4agau1" successfully extended

  [rockynj@node1 ~]$ sudo vgdisplay
  --- Volume group ---
  VG Name               rl_efrei-xmg4agau1
  System ID             
  Format                lvm2
  Metadata Areas        3
  Metadata Sequence No  10
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                4
  Open LV               4
  Max PV                0
  Cur PV                3
  Act PV                3
  VG Size               49.50 GiB
  PE Size               4.00 MiB
  Total PE              12672
  Alloc PE / Size       7552 / 29.50 GiB
  Free  PE / Size       5120 / 20.00 GiB
  VG UUID               LE9S74-oTfB-RSWd-DPX8-r8Is-BlNN-RbKHdf

[rockynj@node1 ~]$ sudo lvextend -l +100%FREE /dev/rl_efrei-xmg4agau1/home
  Size of logical volume rl_efrei-xmg4agau1/home changed from 10.00 GiB (2560 extents) to 30.00 GiB (7680 extents).
  Logical volume rl_efrei-xmg4agau1/home successfully resized.

```

- indiquer au système de fichier ext4 que la partition a été agrandie

```
[rockynj@node1 ~]$ sudo resize2fs /dev/rl_efrei-xmg4agau1/home 
resize2fs 1.46.5 (30-Dec-2021)
Filesystem at /dev/rl_efrei-xmg4agau1/home is mounted on /home; on-line resizing required
old_desc_blocks = 2, new_desc_blocks = 4
The filesystem on /dev/rl_efrei-xmg4agau1/home is now 7864320 (4k) blocks long.
```

- prouvez avec un `df -h` que vous avez récupéré de l'espace en plus

```
[rockynj@node1 ~]$ df -h
Filesystem                            Size  Used Avail Use% Mounted on
devtmpfs                              4.0M     0  4.0M   0% /dev
tmpfs                                 229M     0  229M   0% /dev/shm
tmpfs                                  92M  2.5M   89M   3% /run
/dev/mapper/rl_efrei--xmg4agau1-root  9.8G  1.3G  8.0G  14% /
/dev/sda1                             436M  304M  133M  70% /boot
/dev/mapper/rl_efrei--xmg4agau1-var   4.9G  213M  4.4G   5% /var
/dev/mapper/rl_efrei--xmg4agau1-home   30G  9.3G   19G  33% /home
tmpfs                                  46M     0   46M   0% /run/user/1000

```

## 3. Prepare another partition

Pour la suite du TP, on va préparer une dernière partition. Il devrait vous rester 20G de libre avec le disque de 40 que vous venez d'ajouter.

**Cette partition contiendra des fichiers HTML pour des sites web (fictifs).**

🌞 **Créez une nouvelle partition**

- le LV doit s'appeler `web`
```
[rockynj@node1 ~]$ sudo lvcreate -L+19.9G -n web rl_efrei-xmg4agau1
  Rounding up size to full physical extent 19.90 GiB
  Logical volume "web" created.
```
- elle doit faire 20G et être formatée en ext4

```
[rockynj@node1 ~]$ sudo mkfs -t ext4 /dev/rl_efrei-xmg4agau1/web
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 5217280 4k blocks and 1305600 inodes
Filesystem UUID: 8ab3633b-5658-4c63-853b-dc84a2c45342
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
	4096000

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done   
```

- il faut la monter sur `/var/www`

```
[rockynj@node1 ~]$ sudo mount /dev/rl_efrei-xmg4agau1/web /var/www
[sudo] password for rockynj: 
[rockynj@node1 ~]$ mount
proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)
sysfs on /sys type sysfs (rw,nosuid,nodev,noexec,relatime,seclabel)
devtmpfs on /dev type devtmpfs (rw,nosuid,seclabel,size=4096k,nr_inodes=53174,mode=755,inode64)
securityfs on /sys/kernel/security type securityfs (rw,nosuid,nodev,noexec,relatime)
tmpfs on /dev/shm type tmpfs (rw,nosuid,nodev,seclabel,inode64)
devpts on /dev/pts type devpts (rw,nosuid,noexec,relatime,seclabel,gid=5,mode=620,ptmxmode=000)
tmpfs on /run type tmpfs (rw,nosuid,nodev,seclabel,size=93584k,nr_inodes=819200,mode=755,inode64)
cgroup2 on /sys/fs/cgroup type cgroup2 (rw,nosuid,nodev,noexec,relatime,seclabel,nsdelegate,memory_recursiveprot)
pstore on /sys/fs/pstore type pstore (rw,nosuid,nodev,noexec,relatime,seclabel)
bpf on /sys/fs/bpf type bpf (rw,nosuid,nodev,noexec,relatime,mode=700)
/dev/mapper/rl_efrei--xmg4agau1-root on / type ext4 (rw,relatime,seclabel)
selinuxfs on /sys/fs/selinux type selinuxfs (rw,nosuid,noexec,relatime)
systemd-1 on /proc/sys/fs/binfmt_misc type autofs (rw,relatime,fd=29,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=15454)
debugfs on /sys/kernel/debug type debugfs (rw,nosuid,nodev,noexec,relatime,seclabel)
tracefs on /sys/kernel/tracing type tracefs (rw,nosuid,nodev,noexec,relatime,seclabel)
mqueue on /dev/mqueue type mqueue (rw,nosuid,nodev,noexec,relatime,seclabel)
hugetlbfs on /dev/hugepages type hugetlbfs (rw,relatime,seclabel,pagesize=2M)
none on /run/credentials/systemd-sysctl.service type ramfs (ro,nosuid,nodev,noexec,relatime,seclabel,mode=700)
fusectl on /sys/fs/fuse/connections type fusectl (rw,nosuid,nodev,noexec,relatime)
configfs on /sys/kernel/config type configfs (rw,nosuid,nodev,noexec,relatime)
none on /run/credentials/systemd-tmpfiles-setup-dev.service type ramfs (ro,nosuid,nodev,noexec,relatime,seclabel,mode=700)
/dev/sda1 on /boot type xfs (rw,relatime,seclabel,attr2,inode64,logbufs=8,logbsize=32k,noquota)
/dev/mapper/rl_efrei--xmg4agau1-var on /var type ext4 (rw,relatime,seclabel)
/dev/mapper/rl_efrei--xmg4agau1-home on /home type ext4 (rw,relatime,seclabel)
none on /run/credentials/systemd-tmpfiles-setup.service type ramfs (ro,nosuid,nodev,noexec,relatime,seclabel,mode=700)
tmpfs on /run/user/1000 type tmpfs (rw,nosuid,nodev,relatime,seclabel,size=46788k,nr_inodes=11697,mode=700,uid=1000,gid=1000,inode64)
/dev/mapper/rl_efrei--xmg4agau1-web on /var/www type ext4 (rw,relatime,seclabel)
```

🌞 **Proposez au moins une option de montage**

- au moment où on monte la partition (avec fstab ou la commande `mount`), on peut choisir des options de montage
- proposez au moins une option de montage qui augmente le niveau de sécurité lors de l'utilisation de la partition
```
[rockynj@node1 ~]$ sudo nano /etc/fstab
/dev/mapper/rl_efrei--xmg4agau1-web                             ext4    defaults        0 0
```
- je rappelle que la partition ne contiendra que des fichiers HTML

# Part IV : User management

**Hum, cette partie est censée être envoyée vite fait bien fait ! Prouvez-le moi :D**

Gestion d'utilisateurs, de mot de passe, et de `sudo` ! Puis dans un deuxième temps, on continue sur la gestion de permissions.

## Index

- [Part IV : User management](#part-iv--user-management)
  - [Index](#index)
  - [1. Users](#1-users)
    - [A. Master what already exists](#a-master-what-already-exists)
    - [B. User creation and configuration](#b-user-creation-and-configuration)
    - [C. Hackers gonna hack](#c-hackers-gonna-hack)
  - [2. Files and permissions](#2-files-and-permissions)
    - [A. Listing POSIX permissions](#a-listing-posix-permissions)
    - [B. Protect a file using permissions](#b-protect-a-file-using-permissions)
    - [C. Extended attributes](#c-extended-attributes)

## 1. Users

### A. Master what already exists

🌞 **Déterminer l'existant :**

- lister tous les utilisateurs créés sur la machine

```
[rockynj@node1 ~]$ cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
operator:x:11:0:operator:/root:/sbin/nologin
games:x:12:100:games:/usr/games:/sbin/nologin
ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin
nobody:x:65534:65534:Kernel Overflow User:/:/sbin/nologin
systemd-coredump:x:999:997:systemd Core Dumper:/:/sbin/nologin
dbus:x:81:81:System message bus:/:/sbin/nologin
tss:x:59:59:Account used for TPM access:/:/usr/sbin/nologin
sssd:x:998:996:User for sssd:/:/sbin/nologin
sshd:x:74:74:Privilege-separated SSH:/usr/share/empty.sshd:/usr/sbin/nologin
chrony:x:997:995:chrony system user:/var/lib/chrony:/sbin/nologin
rockynj:x:1000:1000:rockyNJ:/home/rockynj:/bin/bash
tcpdump:x:72:72::/:/sbin/nologin
```

- lister tous les groupes d'utilisateur
```
[rockynj@node1 ~]$ cat /etc/group | cut -d ":" -f1
root
bin
daemon
sys
adm
tty
disk
lp
mem
kmem
wheel
cdrom
mail
man
dialout
floppy
games
tape
video
ftp
lock
audio
users
nobody
utmp
utempter
input
kvm
render
systemd-journal
systemd-coredump
dbus
ssh_keys
tss
sssd
sshd
chrony
sgx
rockynj
tcpdump
```

- déterminer la liste des groupes dans lesquels se trouvent votre utilisateur

```
[rockynj@node1 ~]$ cat /etc/group | grep rockynj
wheel:x:10:rockynj
rockynj:x:1000:
```

🌞 **Lister tous les processus qui sont actuellement en cours d'exécution, lancés par `root`**
```
[rockynj@node1 ~]$ ps aux | grep root
```


🌞 **Lister tous les processus qui sont actuellement en cours d'exécution, lancés par votre utilisateur**

```
[rockynj@node1 ~]$ ps aux | grep "^rockynj "
rockynj      871  0.0  2.9  23868 13944 ?        Ss   13:45   0:00 /usr/lib/systemd/systemd --user
rockynj      873  0.0  1.5 109212  7188 ?        S    13:45   0:00 (sd-pam)
rockynj      880  0.0  0.8   7444  4096 tty1     Ss+  13:45   0:00 -bash
rockynj      941  0.0  1.4  20356  6948 ?        S    14:19   0:00 sshd: rockynj@pts/0
rockynj      942  0.0  0.9   7572  4224 pts/0    Ss   14:19   0:00 -bash
rockynj      986  0.0  0.7  10148  3328 pts/0    R+   14:38   0:00 ps aux
rockynj      987  0.0  0.4   6412  2304 pts/0    S+   14:38   0:00 grep --color=auto ^rockynj
```

🌞 **Déterminer le hash du mot de passe de `root`**
```
[rockynj@node1 ~]$ sudo cat /etc/shadow | grep root | cut -d ":" -f2
$6$ZOHyKh1ifKXslQqM$3NBeZSuhKjzu4ectjVmMjScNsN/rIPjp0iUZEYsHIId9bkRfHRTNjJliWMA.pjtxuHmZC6/VX0xH1XAEu1I0..
```

🌞 **Déterminer le hash du mot de passe de votre utilisateur**

```
[rockynj@node1 ~]$ sudo cat /etc/shadow | grep rockynj | cut -d ":" -f2
$6$IfkYgtMQFmcqYIMv$sw.OOGtEWVuHxhkZz40YNhGxEPbc2/6Ky9/owKrVVbTKIv7IzZnLmb2OknvKI27CpB1tQCB5gr7Ofus5vnyIR1

```

🌞 **Déterminer la fonction de hachage qui a été utilisée**

Le $6$ représente la méthode de hashage SHA-512

```
[rockynj@node1 ~]$ sudo cat /etc/shadow | grep root | cut -d ":" -f2 | cut -d "Z" -f1
$6$
```

🌞 **Déterminer, pour l'utilisateur `root`** :

- son shell par défaut
```
[rockynj@node1 ~]$ cat /etc/passwd | grep root | cut -d ":" -f7
/bin/bash
/sbin/nologin
```

- le chemin vers son répertoire personnel

```
[rockynj@node1 ~]$ cat /etc/passwd | grep root | cut -d ":" -f6
/root
/root
```

🌞 **Déterminer, pour votre utilisateur** :

- son shell par défaut

```
[rockynj@node1 ~]$ cat /etc/passwd | grep rockynj | cut -d ":" -f7
/bin/bash
```

- le chemin vers son répertoire personnel
```
[rockynj@node1 ~]$ cat /etc/passwd | grep rockynj | cut -d ":" -f6
/home/rockynj
```

🌞 **Afficher la ligne de configuration du fichier `sudoers` qui permet à votre utilisateur d'utiliser `sudo`**

![sudo](./img/sudo.png)

```
%wheel  ALL=(ALL)       ALL
```

### B. User creation and configuration

🌞 **Créer un utilisateur :**

Il faut créer le groupe au préalable :
```
[rockynj@node1 ~]$ sudo groupadd admins
```

- doit s'appeler `meow`

```
[rockynj@node1 ~]$ sudo useradd -m -s /usr/sbin/nologin -G admins -d /nonexistent meow
```

- ne doit appartenir QUE à un groupe nommé `admins`

```
[rockynj@node1 ~]$ cat /etc/passwd | grep meow
meow:x:1001:1002::/nonexistent:/usr/sbin/nologin
[rockynj@node1 ~]$ cat /etc/group
admins:x:1001:meow
```
- ne doit pas avoir de répertoire personnel utilisable
```
meow:x:1002:
```

- ne doit pas avoir un shell utilisable
```
/usr/sbin/nologin
```

> Il s'agit donc ici d'un utilisateur avec lequel on pourra pas se connecter à la machine (ni en console, ni en SSH).

🌞 **Configuration `sudoers`**

- ajouter une configuration `sudoers` pour que l'utilisateur `meow` puisse exécuter seulement et uniquement les commandes `ls`, `cat`, `less` et `more` en tant que votre utilisateur

```
meow ALL=(rockynj)	NOPASSWD: /bin/ls, /bin/cat, /bin/less,	/bin/more
```

- ajouter une configuration `sudoers` pour que les membres du groupe `admins` puisse exécuter seulement et uniquement la commande `apt` en tant que `root`
```
%admins ALL=(ALL:ALL) NOPASSWD: /usr/bin/apt
```

- ajouter une configuration `sudoers` pour que votre utilisateur puisse exécuter n'importe quel commande en tant `root`, sans avoir besoin de saisir un mot de passe
```
rockynj ALL=(ALL) NOPASSWD: ALL
```

- prouvez que ces 3 configurations ont pris effet (vous devez vous authentifier avec le bon utilisateur, et faire une commande `sudo` qui doit fonctioner correctement)

```

[rockynj@node1 ~]$ su - meow
Password: 
su: Authentication failure

[rockynj@node1 ~]$ sudo -u rockynj ls
bigfile

[rockynj@node1 ~]$ sudo -u rockynj cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
operator:x:11:0:operator:/root:/sbin/nologin
games:x:12:100:games:/usr/games:/sbin/nologin
ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin
nobody:x:65534:65534:Kernel Overflow User:/:/sbin/nologin
systemd-coredump:x:999:997:systemd Core Dumper:/:/sbin/nologin
dbus:x:81:81:System message bus:/:/sbin/nologin
tss:x:59:59:Account used for TPM access:/:/usr/sbin/nologin
sssd:x:998:996:User for sssd:/:/sbin/nologin
sshd:x:74:74:Privilege-separated SSH:/usr/share/empty.sshd:/usr/sbin/nologin
chrony:x:997:995:chrony system user:/var/lib/chrony:/sbin/nologin
rockynj:x:1000:1000:rockyNJ:/home/rockynj:/bin/bash
tcpdump:x:72:72::/:/sbin/nologin
meow:x:1001:1001::/nonexistent:/usr/sbin/nologin

[rockynj@node1 ~]$ sudo -u rockynj less /etc/group

[2]+  Stopped
```

> Pour chaque point précédent, c'est une seule ligne de configuration à ajouter dans le fichier `sudoers` de la machine.

### C. Hackers gonna hack

🌞 **Déjà une configuration faible ?**

- l'utilisateur `meow` est en réalité complètement `root` sur la machine hein là. Prouvez-le.

Dans cette ligne on administre tous les droits de notre utilisateur à meow sachant que notre utilisateur à le droits de faire ce qu'il veux il est basiquement root sur la machine. Dans ce sens meow aussi.
```
meow ALL=(rockynj)	NOPASSWD: /bin/ls, /bin/cat, /bin/less,	/bin/more
```

- proposez une configuration similaire, sans présenter cette faiblesse de configuration
  - vous pouvez ajouter de la configuration
  - ou supprimer de la configuration
  - du moment qu'on garde des fonctionnalités à peu près équivalentes !

  on ajoute un refus sur toutes les autres commandes.

  ```
  meow ALL=(rockynj) NOPASSWD: ALL, !/bin/*
  ```

## 2. Files and permissions

**Dans un OS, en particulier Linux, on dit souvent que "tout est fichier".**

En effet, que ce soit les programmes (que ce soit `ls`, ou Firefox, ou Steam, ou le kernel), les fichiers personnels, les fichiers de configuration, et bien d'autres, **l'ensemble des composants d'un OS, et tout ce qu'on peut y ajouter se résume à un gros tas de fichiers.**

Gérer correctement les permissions des fichiers est une étape essentielle dans le renforcement d'une machine.

**C'est la première barrière de sécurité, (beaucoup) trop souvent négligée, alors qu'elle est extrêmement efficace et robuste.**

### A. Listing POSIX permissions

🌞 **Déterminer les permissions des fichiers/dossiers...**

- le fichier qui contient la liste des utilisateurs

```
[rockynj@node1 ~]$ ls -l /etc/passwd
-rw-r--r--. 1 root root 1026 Feb 23 19:33 /etc/passwd
```

- le fichier qui contient la liste des hashes des mots de passe des utilisateurs

```
[rockynj@node1 ~]$ ls -l /etc/shadow
----------. 1 root root 876 Feb 23 21:42 /etc/shadow
```

- le fichier de configuration du serveur OpenSSH
```
[rockynj@node1 ~]$ ls -l /etc/ssh/sshd_config
-rw-------. 1 root root 3669 Feb 17 17:35 /etc/ssh/sshd_config
```

- le répertoire personnel de l'utilisateur `root`

```
[rockynj@node1 ~]$ ls -ld /root
dr-xr-x---. 3 root root 4096 Feb 23 19:58 /root
```

- le répertoire personnel de votre utilisateur

```
[rockynj@node1 ~]$ ls -ld /home/rockynj
drwx------. 3 rockynj rockynj 4096 Feb 23 22:38 /home/rockynj
```

- le programme `ls`

```
[rockynj@node1 ~]$ ls -l /bin/ls
-rwxr-xr-x. 1 root root 140952 Nov  6 17:29 /bin/ls
```

- le programme `systemctl`

```
[rockynj@node1 ~]$ ls -l /bin/systemctl
-rwxr-xr-x. 1 root root 305744 Nov 16 02:22 /bin/systemctl
```

> POSIX c'est le nom d'un standard qui regroupe plein de concepts avec lesquels vous êtes finalement déjà familiers. Les permissions rwx qu'on retrouve sous les OS Linux (et MacOS, et BSD, et d'autres) font partie de ce standard et sont donc appelées "permissions POSIX".

![Windows POSIX](./img/posix_compliant.png)

### B. Protect a file using permissions

🌞 **Restreindre l'accès à un fichier personnel**

- créer un fichier nommé `dont_readme.txt` (avec le contenu de votre choix)

```
[rockynj@node1 ~]$ echo "Pouet pouet" > /home/rockynj/dont_readme.txt
```

- il doit se trouver dans un dossier lisible et écrivable par tout le monde

```
[rockynj@node1 ~]$ mkdir /tmp/dossier_pouet_partage
[rockynj@node1 ~]$ mv /home/rockynj/dont_readme.txt /tmp/dossier_pouet_partage/
```

- faites en sorte que seul votre utilisateur (pas votre groupe) puisse lire ou modifier ce fichier

```
[rockynj@node1 ~]$ chmod 777 /tmp/dossier_pouet_partage/
```

- personne ne doit pouvoir l'exécuter

```
[rockynj@node1 ~]$ chmod 600 /tmp/dossier_pouet_partage/dont_readme.txt
```

- prouvez que :

```
[rockynj@node1 ~]$ nano /tmp/dossier_pouet_partage/dont_readme.txt 
  - votre utilisateur peut le lire
  GNU nano 5.6.1                                                          /tmp/dossier_pouet_partage/dont_readme.txt                                                                    
Pouet pouet
```

  - votre utilisateur peut le modifier

```
  [rockynj@node1 ~]$ strings /tmp/dossier_pouet_partage/dont_readme.txt 
Pouet pouet pouet
```

  - l'utilisateur `meow` ne peut pas y toucher

```
[meow@node1 rockynj]$ nano /tmp/dossier_pouet_partage/dont_readme.txt
[ Error reading /tmp/dossier_pouet_partage/dont_readme.txt: Permission denied ]
```

  - l'utilisateur `root` peut quand même y toucher

```
  [root@node1 ~]# nano /tmp/dossier_pouet_partage/dont_readme.txt
   GNU nano 5.6.1                                                          /tmp/dossier_pouet_partage/dont_readme.txt                                                                    
Pouet pouet pouet
```

> C'est l'un des "superpouvoirs" de `root` : contourner les permissions POSIX (les permissions `rwx`). On verra bien assez tôt que `root` n'a pas de "superpouvoirs" mais que ces contournements sont liés à une mécanique qu'on appelle les *capabilites*. C'est pour plus tard ! :)

### C. Extended attributes

🌞 **Lister tous les programmes qui ont le bit SUID activé**

```
[rockynj@node1 ~]$ sudo find / -type f -perm -4000 -exec ls -l {} \;
[sudo] password for rockynj: 
-rwsr-xr-x. 1 root root 15664 Nov 26 00:28 /usr/sbin/pam_timestamp_check
-rwsr-xr-x. 1 root root 15592 Feb  4 19:44 /usr/sbin/grub2-set-bootflag
-rwsr-xr-x. 1 root root 24016 Nov 26 00:28 /usr/sbin/unix_chkpwd
-rwsr-xr-x. 1 root root 32656 May 15  2022 /usr/bin/passwd
-rwsr-xr-x. 1 root root 48680 Nov  7 01:24 /usr/bin/mount
---s--x--x. 1 root root 185304 Feb 14  2024 /usr/bin/sudo
-rwsr-xr-x. 1 root root 78184 Dec 17 22:48 /usr/bin/gpasswd
-rwsr-xr-x. 1 root root 36312 Nov  7 01:24 /usr/bin/umount
-rwsr-xr-x. 1 root root 73872 Dec 17 22:48 /usr/bin/chage
-rwsr-xr-x. 1 root root 57304 Dec 17 22:55 /usr/bin/crontab
-rwsr-xr-x. 1 root root 41920 Dec 17 22:48 /usr/bin/newgrp
-rwsr-xr-x. 1 root root 57136 Nov  7 01:24 /usr/bin/su
find: ‘/proc/1125/task/1125/fdinfo/6’: No such file or directory
find: ‘/proc/1125/fdinfo/5’: No such file or directory
```

🌞 **Rendre le fichier `dont_readme.txt` immuable**

- ça se fait avec les attributs étendus
- "immuable" ça veut dire qu'il ne peut plus être modifié DU TOUT : il est donc en read-only
- prouvez que le fichier ne peut plus être modifié par **personne**

```
[rockynj@node1 ~]$ sudo chattr +i /tmp/dossier_pouet_partage/dont_readme.txt
```

# Part V : OpenSSH Server

**Le serveur OpenSSH est strictement nécessaire à l'administration, et occupe aussi une place cruciale dans le niveau de sécurité d'une machine.**

En effet, on parle d'un programme qui tourne en `root` (obligé...), qui écoute sur un port réseau (il est donc attaquable, c'est une porte potentiellement ouverte), et qui en plus, bah sert à prendre le contrôle d'une machine à distance.

Besoin d'un dessin pour expliquer à quel point c'est sensible ?

Néanmoins nécessaire partout.

## Index

- [Part V : OpenSSH Server](#part-v--openssh-server)
  - [Index](#index)
  - [1. Basics](#1-basics)
  - [2. Authentication modes](#2-authentication-modes)
    - [A. Key-based authentication](#a-key-based-authentication)
  - [3. Bonus : Cert-based authentication](#3-bonus--cert-based-authentication)
  - [4. Further hardening](#4-further-hardening)
  - [5. fail2ban](#5-fail2ban)
  - [6. Automatisation](#6-automatisation)

## 1. Basics

🌞 **Afficher l'identifiant du processus serveur OpenSSH en cours d'exécution**

- listez tous les programmes en cours d'exécution (avec une commande `ps`)

```
[rockynj@node1 ~]$ ps aux
```

- mettez en évidence uniquement la ligne qui concerne le serveur SSH (y'en a qu'une)

```
[rockynj@node1 ~]$ ps aux | grep sshd
```

> On peut aussi obtenir l'info avec un `systemctl status` bien senti ;D

🌞 **Changer le port d'écoute du serveur OpenSSH**
```
sudo nano /etc/ssh/sshd_config
Port 8350
```
- prouvez que votre changement a pris effet
- prouvez que vous pouvez toujours vous connecter à la machine en SSH, sur ce nouveau port
- expliquez pourquoi on considère parfois utile de changer le port d'écoute par défaut du serveur SSH

```
njboulot@njboulot-LOQ-15IRH8:~$ ssh rockynj@10.1.1.11 -p 8350
rockynj@10.1.1.11's password: 
Last login: Mon Feb 24 00:10:25 2025
```

éviter les attaques automatiser qui se font très fréquente sur lke port 22.

## 2. Authentication modes

### A. Key-based authentication

> Un classique ! Vous **devez** être à l'aise avec ça. Jamais trop tard pour s'y mettre.

🌞 **Configurer une authentification par clé**

```
njboulot@njboulot-LOQ-15IRH8:~$ sudo ssh-keygen -t rsa -b 4096
```

- vous devez pouvoir vous connecter sur votre utilisateur
- sans saisir de password
- en utilisant une paire de clés

```


🌞 **Désactiver la connexion par password**

🌞 **Désactiver la connexion en tant que `root`**

![ssh as root](./img/ssh_as_root.png)

## 3. Bonus : Cert-based authentication

> Moins classique, mais supporté depuis très longtemps par OpenSSH, et très fort en terme de sécurité !

⭐ **BONUS** : **Configurer une authentification par certificat**

- j'ai dit par certificat, pas par simple clé
- pareil, faites-le avec votre utilisateur pour les tests

> L'authentification par certificat est toujours plus forte que l'authentification par simple clé : les deux parties (typiquement, le client et le serveur) doivent prouver l'identité à l'autre. De plus, le certificat ne peut pas être falsifié, du moins si on utilise une autorité de certification digne de confiance. L'idée du certificat : on va signer la clé du client avec la clé d'une autorité de certification. Ainsi, la clé n'est plus falsifiable, l'autorité de certification peut attester que c'est la bonne clé pour le bon client.

## 4. Further hardening

🌞 **Proposer au moins 5 configurations supplémentaires qui permettent de renforcer la sécurité du serveur OpenSSH**

> Je vous recommande fooooortement de vous inspirer de ressources d'Internet pour ça. Regardez par exemple le guide de l'ANSSI à ce sujet (obsolète, mais la plupart des principes sont toujours valides), ou encore le guide CIS sur le sujet, ou l'excellent guide Mozilla sur le sujet, . Il existe d'autres ressources de confiance, à votre meilleur moteur de recherches !

## 5. fail2ban

> Un outil extrêmement récurrent dans le monde Linux : un premier rempart contre les attaques de bruteforce.

🌞 **Installer fail2ban sur la machine**

🌞 **Configurer fail2ban**

- en cas de multiples tentatives de connexion échouées sur le serveur SSH, l'utilisateur sera banni
- précisément : après 7 tentatives de connexion échouées en moins de 5 minutes
- c'est l'adresse IP de la personne qui fait des connexions échouées de façon répétée qui est blacklistée

🌞 **Prouvez que fail2ban est effectif**

- faites-vous ban
- montrez l'état de la jail fail2ban pour voir quelles IP sont ban
- levez le ban avec une commande adaptée

## 6. Automatisation

Dernière section : un peu de dév en bash pour automatiser toute la configuration que vous venez de faire.

L'idée est simple : écrire un script shell qui applique la configuration de cette Partie V (openSSH et fail2ban) sur une machine Rocky Linux fraîchement installée.

🌞 **Ecrire le script `harden.sh`**

- il doit vérifier que le serveur openSSH est démarré 
- il doit vérifier que le port d'écoute de openSSH n'est pas 22
- il doit effectuer les configurations openSSH relatives à la sécurité
  - je fais référence aux points 2. et 3.
- il doit vérifier que fail2ban est installé et démarré
- il doit vérifier que fail2ban surveille bien les logs de openSSH
 
