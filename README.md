# Born2beroot

- infos: 
    faire machine virtuelle et y creer mon propre systeme d'exploitation (serveur) en implementant des
    regles strictes

- fichier rendu:
    signature.txt
        contient la ??signature du disque virtuel

- soft:
    VirtualBox (ou UTM seulement si virtualbox ne fonctionne pas)

- details:
    - sans interface grafique (X.org ou autre serveur graphique interdit)

    - systeme d'exploitation: Debian
        - se renseigner sur le systeme (par ex. diff entre aptitude et apt)

    - ??AppArmor devra rester actif
        --> module de securite pour noyau linux qui fournit un controle d'acces obligatoire, permet de restraindre les capacites des applis en fonction des profils

    - ??SELinux devra rester actif (a verifier p.5)
    
    - ??KDump a mettre en place 

    - creer au minimum 2 ppartitions chiffrees en utilisant LVM
#        exemple:
                wil@wil:~$ lsblk
                NAME                         MAJ:MIN RM  SIZE    RO  TYPE    MOUNTPOINT
                sda                            8:0   0   8G      0   disk    
                |--sda1                        8:1   0   478M    0   part    /boot   
                |--sda2                        8:2   0   1k      0   part
                |--sda5                        8:5   0   7.5G    0   part
                    |--sda5_crypt             254:0  0   7.5G    0   crypt
                        |--wil--vg-root       254:1  0   2.8G    0   lvm     /
                        |--wil--vg-swap_1     254:2  0   976M    0   lvm     [SWAP]
                        |--wil--vg-home       254:3  0   3.8G    0   lvm     /home  
                sr0                            11:0  1   1024M   0   rom
                wil@wil:~$ _

/    - ??service SSH obligatoirement actif sur port 4242 de la VM
        - interdiction de se connecter avec par SSH avec user root
        - ??ssh a tester en creeant un nouveau compte, comprendre comment fonctionne ce service

/    - Securite:
        - Pare-feu: UFW
            - laisser ouvert que le port 4242 de la VM
            - devra etre actif au lancement de la VM

/        - MDP: cyfze9pzP
            - devra expirer tous les 30 jours
            - 2 jours min avant de pouvoir le modifier
            - l'utilisateur doit recevoir un avertissement 7 jours avant que le mdp expire
                - que se passe-t-il s'il expire? nouveau auto ou perdu??
            - min 10 caract, dont 1maj 1min 1chiffre, pas plus de 3 caract identiques consecutifs
            - ne devra pas comporter le nom de l'utilisateur associe
            - devra comporter au moins 7 caract qui ne sont pas dans l'ancien mdp (sauf pour user root)

/            - apres avoir mis en place les fichiers de config, les mdp de tous les comptes
              doivent etre changes (dont root)

/    - Utilisateurs
        - hostname: rabiner42
            - il faut pouvoir le modifier
        - ??root (meme que hostname?)
        - utilisateur1: rabiner
            - appartiendra aux groupes user42 et sudo
        - il faut pouvoir creer un nouvel utilisateur et lui assigner un groupe

/    - ??sudo
        - 3 essais max en cas de mdp faux pour s'authentifier en utilisant sudo
        - un message doit s'afficher (au choix) suite a un mauvais mdp (en utilisant sudo)
        - chaque action utilisant sudo sera archivee, input comme outputs,
          le journal se trouvera dans le dossier /var/log/sudo/.
        - ?? le mode TTY sera active pour des questions de securite
        - ?? les path utilisables par sudo seront restrints pour securite
            - ??ex: /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin

/    - Script a mettre en place: monitoring.sh (a dev en bash)
        - des le lancement du serveur, le script ecrira les informations suivantes
          toutes les 10min sur tous les terminaux (??voir wall), ??la banniere est facultative
          a aucun moment la moindre erreur ne doit etre visible

/        - infos du script:
            - architecture du sys d'exploit et sa version de kernel
            - nbr de processeurs physiques
            - nbr de processeurs virtuels
            - memoire vive disponible actuelle sur le serveur et taux d'utilisation (%)
            - memoire disponible actuelle sur le serveur et taux d'utilisation (%)
            - taux d'utilisation actuel des processeurs (%)
            - date et heure du dernier redemarrage
            - si LVM est actif ou pas
            - nbre de connexions actives
            - nbre d'utilisateurs utilisant le serveur
            - adresse IPv4 et MAC du serveur
            - nbre de commande executees avec le prog sudo

/            - il faut pouvoir expliquer le fonctionnement du script et interrompre l'execution 
              sans le modifier (??voir cron)

/            - ex de rendu:
                Broadcast message from root@wil (tty1) (Sun Apr 25 25:45:00 2021):

                   #Architecture: Linux wil 4.23.0-16-amd64 #1 SMP Debian 4.19.181-1 (2021-03-19) x86_64 GNU/Linux
                   #CPU physical : 1
                   #vCPU : 1
                   #Memory Usage: 74/087MB (7.50%)
                   #Disk Usage: 1009/2GB (49%)
                   #CPU load: 6.7%
                   #Last boot: 2021-04-25 14:34
                   #LVM use: yes
                   #Connexions TCP: 1 ESTABLISHED
                   #User log: 1
                   #Network: IP 10.0.2.15 (08:00:27:51:9b:a5)
                   #Sudo: 42 cmd


/    - exemples de commandes pour tests: (wil est le nom d'utilisateur d'exemple)

        root@wil:~# head -n 2 /etc/os-release
        PRETTY_NAME="Debian GNU/Linux 10 (buster)"
        NAME="Debian GNU/Linux"
        root@wil:/home/wil# /usr/sbin/aa-status
        apparmor module is loaded
        root@wil:/home/wil# ss-tunlp
        Netid   state   Recv-Q Send-Q Local Address:Port   Peer Address:Port

        tcp     LISTEN  0       128         0.0.0.0:4242        0.0.0.0:*       users:(("sshd"pid=523,fd=3))
        tcp     LISTEN  0       128            [::]:4242           [::]:*       users:(("sshd"pid=523,fd=4))
        root@wil:/home/wil#  /usr/sbin/ufw status
        Status: active

        To                  Action      From
        --                  ------      ----
        4242                ALLOW       Anywhere
        4242(v6)            ALLOW       Anywhere (v6)
