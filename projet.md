# Mise en place d'un serveur Rust sur Linux

## 1. Mise à jour et préparation du système

### -> Mettre à jour le système

```
root@vmi2316883:~# apt update && sudo apt upgrade -y
```

### -> Installer unattended-upgrades pour gérer les mises à jour automatiques.

```
root@vmi2316883:~# apt install unattended-upgrades -y
```

### -> Configurer les mises à jour automatiques

```
root@vmi2316883:~# dpkg-reconfigure --priority=low unattended-upgrades
```

### -> Configurer les sauvegardes automatiques

```
rustserver@vmi2316883:~$ sudo apt install rsync

rustserver@vmi2316883:~$ rsync -avz -e "ssh -p 2222" --delete /home/rustserver sauvegarde@157.173.122.151:/chemin/de/sauvegarde

rustserver@vmi2316883:~$ sudo nano /usr/local/bin/backup_rustserver.sh

#!/bin/bash

# Répertoire à sauvegarder
SOURCE_DIR="/home/rustserver"

# Destination de sauvegarde
DEST_USER="sauvegarde"
DEST_SERVER="157.173.122.151" 
DEST_PATH="/chemin/de/sauvegarde"

# Commande rsync
rsync -avz --delete $SOURCE_DIR ${DEST_USER}@${DEST_SERVER}:${DEST_PATH}

# Enregistrement des logs
if [ $? -eq 0 ]; then
    echo "$(date '+%Y-%m-%d %H:%M:%S') - Sauvegarde réussie" >> /var/log/backup_rustserver.log
else
    echo "$(date '+%Y-%m-%d %H:%M:%S') - Erreur lors de la sauvegarde" >> /var/log/backup_rustserver.log
fi

rustserver@vmi2316883:~$ chmod +x /usr/local/bin/backup_rustserver.sh

rustserver@vmi2316883:~$ crontab -e

0 3 * * * /usr/local/bin/backup_rustserver.sh

```

### (Consultez le journal /var/log/backup_rustserver.log pour vérifier l’état des sauvegardes.)

## 2. Création de l'utilisateur dédié au serveur Rust
   
### -> Créer un nouvel utilisateur pour le serveur Rust

```
root@vmi2316883:~# adduser rustserver
```

### -> Ajouter cet utilisateur au groupe sudo pour lui permettre d'exécuter des commandes avec des privilèges élevés

```
root@vmi2316883:~# usermod -aG sudo rustserver
```

### -> Se connecter avec l'utilisateur rustserver

```
root@vmi2316883:~# su - rustserver
```

### -> Vérifier qu'on a bien les privilèges sudo

```
rustserver@vmi2316883:~$ sudo whoami
```

## 3. Sécurisation du serveur

### -> Configurer le systeme ssh

```
rustserver@vmi2316883:~$ sudo nano /etc/ssh/sshd_config

Port 2222
PasswordAuthentication no
PermitRootLogin no

rustserver@vmi2316883:~$ sudo systemctl restart ssh

rustserver@vmi2316883:~$ sudo ufw allow 2222

```

### -> Creation des clés ssh

```
PS C:\Users\teamc> ssh-keygen -t rsa -b 4096

Your identification has been saved in C:\Users\teamc/.ssh/id_rsa
Your public key has been saved in C:\Users\teamc/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:PAegpDyfAe3PYJwHtGF6StWmtNFgGUzBiGreNVNxAro teamc@PCNAMM
The key's randomart image is:
+---[RSA 4096]----+
| ..B%Ooo..       |
|...OX++oo        |
|. *==B. .        |
|.o =X*.. .       |
|o oE+=o S .      |
| . .  o  o       |
|                 |
|                 |
|                 |
+----[SHA256]-----+

rustserver@vmi2316883:~$ echo "ssh-rsa AAAAB3NzaC1yc2Enx(je vais pas donner la clé en entier c'est long) teamc@PCNAMM" | tee -a /home/rustserver/.ssh/authorized_keys
```

### -> Installer fail2ban, un outil pour sécuriser ton serveur contre les attaques par force brute.

```
rustserver@vmi2316883:~$ sudo apt install fail2ban -y
```

### -> Configurer fail2ban pour protéger le service SSH, en changeant le port SSH et en limitant le nombre de tentatives de connexion.

```
rustserver@vmi2316883:~$ sudo nano /etc/fail2ban/jail.local

[sshd]
enabled = true
port = 2222
logpath = /var/log/auth.log
bantime = 600
maxretry = 5
```

### -> Redémarrer fail2ban pour appliquer les changements.

```
rustserver@vmi2316883:~$ sudo systemctl restart fail2ban
``` 
## 4. Configuration du pare-feu avec UFW et d'un anti cheat

### -> Installer et configurer ufw (Uncomplicated Firewall) pour protéger le serveur.

```
rustserver@vmi2316883:~$ sudo apt install ufw -y
rustserver@vmi2316883:~$ sudo ufw default deny incoming
rustserver@vmi2316883:~$ sudo ufw default allow outgoing
```

### -> Autoriser les ports nécessaires pour Rust (TCP et UDP sur le port 28015).

```
rustserver@vmi2316883:~$ sudo ufw allow 28015/tcp
rustserver@vmi2316883:~$ sudo ufw allow 28015/udp
```

### -> Installation de 

### *1. Installer uMod/Oxide

```

Télécharger la dernière version d'uMod pour Rust :
wget https://umod.org/download/latest

Extrayez et remplacez les fichiers dans le dossier du serveur :

unzip latest.zip -d ~/rust_server

Redémarrez votre serveur Rust pour activer uMod :

cd ~/rust_server

./RustDedicated -batchmode -nographics +server.port 28015 ...
```

### *2. Ajouter un plugin anticheat

```
uMod propose plusieurs plugins d'anticheat pour Rust. Voici comment en installer un :

Télécharger le plugin "RustAdminAntiCheat" :

Placez-le dans le dossier oxide/plugins/ sur votre serveur.

mv oxide.cs ~/rust_server/oxide/plugins/

Chargez le plugin : Connectez-vous à la console RCON ou utilisez un terminal dans le jeu :

oxide.reload RustAdminAntiCheat
```

### 3*. Configurer et personnaliser l’anticheat

```
nano ~/rust_server/oxide/config/RustAdminAntiCheat.json

```

## 5. Installation de SteamCMD et du serveur Rust

### -> Installer les dépendances nécessaires pour SteamCMD.

```
rustserver@vmi2316883:~$ sudo apt install wget tar curl lib32gcc-s1 -y
```

### -> Créer un dossier pour SteamCMD et le télécharger.

```
rustserver@vmi2316883:~$ mkdir ~/steamcmd && cd ~/steamcmd
rustserver@vmi2316883:~/steamcmd$ wget https://steamcdn-a.akamaihd.net/client/installer/steamcmd_linux.tar.gz
rustserver@vmi2316883:~/steamcmd$ tar -xvzf steamcmd_linux.tar.gz
```

### -> Lancer steamcmd pour installer le serveur Rust.

```
rustserver@vmi2316883:~/steamcmd$ ./steamcmd.sh
rustserver@vmi2316883:~ ./steamcmd.sh +login anonymous +force_install_dir ~/rust_server +app_update 258550 validate +quit
```

## 6. Configuration du serveur Rust

### -> Une fois le serveur installé, accèder à son répertoire.

```
rustserver@vmi2316883:~/steamcmd$ cd ~/rust_server
```

### -> Créer un dossier pour la configuration du serveur.

```
rustserver@vmi2316883:~/rust_server$ mkdir -p server/notre_serveur/cfg
```

### -> Créer et modifier le fichier de configuration principal server.cfg :

```
rustserver@vmi2316883:~/rust_server$ nano server/notre_serveur/cfg/server.cfg

server.hostname "Notre Serveur Rust"
server.description "Bienvenue sur le serveur de Louis et Titou"
server.url "http://yourwebsite.com"
server.maxplayers 50
server.pvp true
server.saveinterval 300
```

## 7. Création du script de démarrage du serveur

### -> Créer un fichier start.sh pour démarrer le serveur automatiquement avec les paramètres souhaités.

```
rustserver@vmi2316883:~$ nano start.sh

#!/bin/bash
clear
echo "Starting Rust Server..."
./RustDedicated -batchmode -nographics \
+server.port 28015 \
+server.ip 0.0.0.0 \
+server.maxplayers 50 \
+server.hostname "MonServeurRust" \
+server.identity "rust_server" \
+server.seed 12345 \
+server.worldsize 4000 \
+rcon.port 28016 \
+rcon.password "votre_mot_de_passe_rcon" \
+rcon.web 1
```

### -> Rendre ce script exécutable.

```
rustserver@vmi2316883:~$ chmod +x start.sh
```

## 8. Lancer le serveur

### -> Lancer le serveur avec le script start.sh.

```
rustserver@vmi2316883:~$ ./start.sh
```

## 9. Relance automatique du serveur après reboot

### -> Créer un service systemd pour relancer le serveur après un redémarrage du VPS.

```
rustserver@vmi2316883:~$ sudo nano /etc/systemd/system/rust_server.service

[Unit]
Description=Rust Dedicated Server
After=network.target

[Service]
User=your_username
WorkingDirectory=/home/your_username/rust_server
ExecStart=/home/your_username/rust_server/RustDedicated -batchmode -nographics +server.port 28015 +server.identity "my_server" +server.seed 12345 +server.worldsize 3500 +server.maxplayers 50 +server.hostname "My Rust Server" +server.description "Welcome to my Rust server!"
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
Recharge les configurations systemd et active le service.
```

```
rustserver@vmi2316883:~$ sudo systemctl daemon-reload
rustserver@vmi2316883:~$ sudo systemctl enable rust_server
rustserver@vmi2316883:~$ sudo systemctl start rust_server
```

## 10.  Surveillance et consultation des logs pour la maintenance

### -> Surveiller les processus

```
rustserver@vmi2316883:~$ sudo apt install htop
```

### -> Surveiller les connexions réseaux

```
rustserver@vmi2316883:~$ sudo apt install net-tools
rustserver@vmi2316883:~$ netstat -tuln
```

### -> Consulter les logs du serveur pour la maintenance

```
rustserver@vmi2316883:~$ cat server/my_server/identity/logs/output_log.txt
```