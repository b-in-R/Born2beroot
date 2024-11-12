

# Guide d'Installation pour Born2beRoot

Ce document fournit des instructions détaillées étape par étape pour l'installation et la configuration de votre serveur dans le cadre du projet Born2beRoot.

## Table des Matières

1. [Prérequis](#prérequis)
2. [Installation du Système d'Exploitation](#installation-du-système-dexploitation)
3. [Configuration de Base](#configuration-de-base)
4. [Configuration SSH](#configuration-ssh)
5. [Configuration du Pare-feu](#configuration-du-pare-feu)
6. [Politique de Mot de Passe](#politique-de-mot-de-passe)
7. [Configuration de Sudo](#configuration-de-sudo)
8. [Partie Bonus (Optionnelle)](#partie-bonus-optionnelle)
9. [Vérification de la Configuration](#vérification-de-la-configuration)

## Prérequis

- Avoir installé VirtualBox ou UTM. (sur les machines dans les clusters, VirtualBox est deja installe)
`sudo apt install VirtualBox`
- Télécharger l'image ISO de Debian.
https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/



## Installation du Système d'Exploitation

1. Créer une nouvelle machine virtuelle dans VirtualBox/UTM.
2. Sélectionnez le type de système d'exploitation Linux.
3. Attribuez au moins 2 Go de RAM à la machine virtuelle.
4. Créez un disque virtuel de taille suffisante (~12 Gb).
5. Démarrez la machine virtuelle et sélectionnez l'image ISO téléchargée pour l'installation.
6. Suivez les instructions d'installation en choisissant les options appropriées.

## Installation OS Debian

1. Sélectionnez l'option d'installation sans interface graphique. `install`
2. Choisissez la langue voulue, (de preference `English`)
3. La localisation.
4. Keyboard Layout, le layout de votre machine.
5. Pour les informations d'identification voir ci-dessous : [Informations identifications](#Information-identifications)
6. Partitions Disks:
   - Sélectionnez `guided - Use entire ... encrypted LVM`.
   - Selectionner le disque `SCI3` (Disque entier)
   - Lors de la partition, utilisez `Separate /home, /var...`
   - Valider les changements sur le dique.
   - Creer un passphrase pour l'encriptage du disque.
	   - [Informations identifications](#Information-identifications)
	- Choisissez la taille max du disque en ecrivant `max`
	- Confirmer `finish patritionning and write change to disk`
7. Configure package manager :
	- Refusez de `Scan another CD or DVD`
	- Pour le mirroir Debian, selectionnez le pazs ou vous vous trouver.
	- choisissez `deb.debian.org`
	- Laissez blanc le proxy.
8. Refusez le survey
9. Enlevez toutes les coches. `SSH Server` et `Standard system utilities`
10. Installez le GRUB `to the master boot record`
11. Selectionnez le disque `/dev/sda`

L'OS va finaliser l'installation. 
Bravo Loris tu as reussi. 
 
 ## Ouverture VM
 - Sur VirtualBox, cliquez sur `Play`
 - Entrez votre passphrase
 - utilisez votre login pour vous connecter
 - Vous pouvez voir vos partitions en tapant `lsblk`
 - Lorsque vous fait des installations, pensez a taper `apt-get update -y` et `apt-get upgrade -y`

		update  : met a jour la liste des paquets disponible 
		upgrade : met a jour les paquets installes
		-y      : permet d'ignorer la confirmation d'installation
  

## Information identifications
Voici les informations a mettre lors de la creation des comptes :

	- Hostname = Login+42. Exemple "jroulet42"a
	- Username full name = "jroulet_full"
	- Login = login 42. Exemple "jroulet"
	- Pensez a mettre un mot de passe identique pour le root, le login et le passphrase. 
	


## Installation et configuration de Sudo

1. Pour changer d'utilisateur, tapez la commande `su` (switch utilisateur)
	- Sans parametre, le system va passer sur le compte root (superutilisateur)  
2. Installez sudo en etant connecte avec le compte Super Utilisateur : `apt install sudo`.
3. Éditez le fichier de configuration sudo : `sudo visudo`.
	### ⚠️ NOTE: pour la modification du fichier de configuration d'acces (/etc/sudoers) ne doit se faire UNIQUEMENT avec visudo
4. Ajoutez les configurations requises pour une utilisation stricte de sudo 

![PDF](https://raw.githubusercontent.com/jimourouly/Born2BeRoot/main/Images/sudo.png)



6. Pour ajouter votre utilisateur dans le groupe sudo, tapez `usermod -aG sudo "votre_login"` 
7. Pour permettre a votre compte d'utiliser toutes les fonction sudo, trouvez la ligne `#user privilege sepcifcation` et ajoutez `your_login ALL=(ALL:ALL) ALL`
	- Apres avoir fait cette modification vous n'aurez plus besoin d'utilisez la commande `su` pour faire des manipulations en etant Super Utilisateur

## Installation Vim
1. Tapez `apt-get install vim -y`


## Configuration SSH

1. Installez le serveur SSH si ce n'est pas déjà fait : `sudo apt install openssh-server`.
2. Éditez le fichier de configuration SSH : `sudo vim /etc/ssh/sshd_config`.
(Si Vim n'est pas installe utilisez nano)
3. Changez le port par défaut en 4242 : `Port 4242`.
4. Désactivez la connexion en tant que root : `PermitRootLogin no`. 
5. Redémarrez le service SSH : `sudo systemctl restart ssh`.

## Tester la configuration SSH

1. Eteignez la VM.
	- `sudo systemctl poweroff`
2. Allez dans les parametres de la VM sur VirtualBox.
3. Ouvrez les settings, Network, Adaptater 1.
4. Cliquez sur Advanced, puis Port Forwarding.
5. Cliquez sur le "+" vert. Laissez Rule 1, changez Host port et Guest port en "4242".
6. Quittez les settings. 
7. Redemarrez la vm.

8. Avec le teminal connectez vous en ssh en tapant `ssh lgaume@127.0.0.1 -p 4242`.
Changer lgaume par votre nom d'utilisateur.

9. Sur la vm, ecrivez `sudo service sshd status`.
Vous devriez voir la connexion qui a ete faite. 

## Configuration du Pare-feu

1. Installez UFW (ou firewalld pour Rocky) : `sudo apt install ufw`
2. Activez UFW : `sudo ufw enable`.
3. Autorisez le port 4242 : `sudo ufw allow 4242/tcp`.
4. Vérifiez la configuration : `sudo ufw status`.

## Politique de Mot de Passe


1. Éditez le fichier de configuration PAM : `sudo vim /etc/pam.d/common-password.`
2. Ajoutez vos règles pour la complexité du mot de passe (Voir https://linux.die.net/man/8/pam_pwquality).
3. indice : retry, minlen, ucredit, dcredit ... 
<details>
	<summary>Regle a modifier Spoiler</summary>

4. Trouvez la ligne qui commence par `password requisite pam-pwquality.so`.
5. Ajoutez a la suite `retry=3 minlen=10 ucredit=-1 maxrepeat=3 reject_username difok=7 enforce_for_root`.
6. Modifiez le fichier `/etc/login.defs` pour modifier les lignes ci-dessous : 
	- MAX_PASS_DAYS 30
	- MIN_PASS_DAYS 2
	- PASS_WARN_AGE 7


</details>

## Gestion utilisateur

### Création Utilisateur 
1. Sudo adduser `new_user`
	- un mot de passe sera demandé
2. Pour avoir la liste des utilisateurs `cut -d: -f1 /etc/passwd`
	
### Création Groupe 
1. sudo groupadd `groupname`
	 - Pour ajouter un utilisateur au groupe : `sudo usermod -aG groupname username`
2. Pour savoir si un groupe a été créer, tapez `getent group`

### Création des Logs 
1. Créer le dossier `/var/log/sudo`
2. Créer un fichier `sudo.log` dans le dossier
3. Modifier le visudo `sudo visudo` et ajouter :
 ```
 Defaults	env_reset
Defaults	mail_badpass
Defaults	secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"
Defaults	badpass_message="Mauvais mot de passe, reessaye!"
Defaults	passwd_tries=3
Defaults	logfile="/var/log/sudo/asudo.log"
Defaults	log_input, log_output
Defaults	requiretty
```

# Monitoring configuration
1. Installation de Crontab (planificateur de tâches pour l'éxecution du script de monitoring)
- `apt-get install -y net-tools` 
2. Rendez vous sur le dossier `/usr/local/bin`
3. Créer le fichier monitoring.sh
4. Modifier les accès du dossier `chmod 777 monitoring.sh`
5. Ajoutez-y le script :
<details>
	<summary>Script</summary>

```
#!/bin/bash
arc=$(uname -a)
pcpu=$(grep "physical id" /proc/cpuinfo | sort | uniq | wc -l) 
vcpu=$(grep "^processor" /proc/cpuinfo | wc -l)
fram=$(free -m | awk '$1 == "Mem:" {print $2}')
uram=$(free -m | awk '$1 == "Mem:" {print $3}')
pram=$(free | awk '$1 == "Mem:" {printf("%.2f"), $3/$2*100}')
fdisk=$(df -BG | grep '^/dev/' | grep -v '/boot$' | awk '{ft += $2} END {print ft}')
udisk=$(df -BM | grep '^/dev/' | grep -v '/boot$' | awk '{ut += $3} END {print ut}')
pdisk=$(df -BM | grep '^/dev/' | grep -v '/boot$' | awk '{ut += $3} {ft+= $2} END {printf("%d"), ut/ft*100}')
cpul=$(top -bn1 | grep '^%Cpu' | cut -c 9- | xargs | awk '{printf("%.1f%%"), $1 + $3}')
lb=$(who -b | awk '$1 == "system" {print $3 " " $4}')
lvmu=$(if [ $(lsblk | grep "lvm" | wc -l) -eq 0 ]; then echo no; else echo yes; fi)
ctcp=$(ss -neopt state established | wc -l)
ulog=$(users | wc -w)
ip=$(hostname -I)
mac=$(ip link show | grep "ether" | awk '{print $2}')
cmds=$(journalctl _COMM=sudo | grep COMMAND | wc -l)
wall "	#Architecture: $arc
	#CPU physical: $pcpu
	#vCPU: $vcpu
	#Memory Usage: $uram/${fram}MB ($pram%)
	#Disk Usage: $udisk/${fdisk}Gb ($pdisk%)
	#CPU load: $cpul
	#Last boot: $lb
	#LVM use: $lvmu
	#Connections TCP: $ctcp ESTABLISHED
	#User log: $ulog
	#Network: IP $ip ($mac)
	#Sudo: $cmds cmd"
```


</details>

5. Pour permettre l'execution du script, ajouter la ligne ci-dessous.  `sudo visudo` et ajouter la ligne `your_username ALL=(ALL) NOPASSWD: /usr/local/bin/monitoring.sh` en dessous de la ligne `sudo ALL=ALL...`
6. Redémarrez votre VM `sudo reboot`
7. Lancer votre script avec `sudo /usr/local/bin/monitoring.sh`
8. Ouvrir Crontab `sudo crontab -u root -e`
9. A la fin du fichier ajouter la règle d'éxecution de chaque 10 minutes : 
	- Voir https://crontab.guru/ pour savoir quoi mettre
	
<details>
	<summary>Règle Crontab</summary>
	https://crontab.guru/every-10-minutes
</details>

# Contrôle pour l'evaluation 
1.  pour checker la date d'éxpiration du mot de passe `sudo chage -l username`
2. Pour contrôler l'assignation a un groupe `getent group groupname`
3. Controle le nom de domaine `hostnamectl`
4. Modifier le nom d'hôte `hostnamectl set-hostname new_hostname`
5. Voir les partitions `lsblk`
6. checker si le sudo est bien installé `dpkg -s | grep sudo -`
7. Checker si le Par-feu est bien installé `dpkg -s | grep ufw -`
8. Checker si le ssh est installé `dpkg -s | grep ssh -`
9. Verifier si le par-feu a les bonnes règles `sudo ufw status numbered`
10. Verifier le fichiers de logs Sudo `cd /var/log/sudo/`

# Signature 
1. Se rendre dans le dossier ou se trouve la VM. 
2. Tapez dans le terminal `shasum your_vm.vdi`
3. Copier la suite de numéro dans un fichier `signature.txt`. Pusher le fichier sur votre Git. 
4. Attention, il est possible que la signature change d'une evaluation a l'autre. Faites un snapshot de votre VM lorsque celle-ci est éteinte. Au cas ou la signature diffère, revenez sur le snapshot d'avant la première évaluation.
## Partie Bonus (Optionnelle)

<details>
	<summary>Partie Bonus</summary>

g pa fé
</details>

## Vérification de la Configuration


1. Testez la connexion SSH sur le port 4242.
`ssh -p 4242 votre_utilisateur@1270.0.1 -p 4242`

3. Vérifiez que le pare-feu est configuré correctement.
`sudo ufw status`

4. Assurez-vous que la politique de mot de passe est appliquée.
`cat /etc/pam.d/common-password`

5. Vérifiez la configuration de sudo.
`cat /etc/sudoers`

---

Une fois que tout est configuré, vous pouvez revenir au [README.md](README.md) pour les étapes finales et les instructions pour le rendu.


=
