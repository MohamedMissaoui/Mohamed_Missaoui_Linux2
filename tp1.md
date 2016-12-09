# Compte-rendu TP Linux 2
*************************

##TP1
=====
####HYPERVISEUR VIRTUAL BOX & MACHINE VIRTUELLE
Installation de la machine virtuel Debian dans VirtualBox.
Installation des packets (sous root) aptitude  install lynx sudo tcpdump vim.

####SUDO
Prise d'information sur commande sudo.
Droit d'accès sudo pour notre utilisateur:
	- visudo (et non vim /etc/sudoers) qui permet d'éditer le fichier /etc/sudoers
	- rajouter la ligne: nomutilisateur ALL=(ALL:ALL) ALL
	- enregistrer et relancer le service sudo (service sudo restart)
	- tester si l'utilisateur a bien les droit de sudo
Nom des groupes de l'utilisateur (avec la commande groups):
	- cdrom		- floppy	- audio		- dip
	- video		- plugdev	- netdev	- bluetooth

####CLONAGE
Pour faire un clone de la VM "debian_base", il suffi de faire un clic droit sur celle-ci et de cliquer sur "Cloner" (ou raccourci Ctrl+O).

####SNAPSHOT
rm -rf / (cette ligne de commande supprime tout les fichiers)
En essayant de relancer le snapshot, on tombe sur grub rescue puisque tout à été supprimer au préalable.
Il suffi de restaurer le snapshot "Avant rm" afin de pouvoir relancer la VM.

####CONFIGURATION RÉSEAUX
Après avoir éxecuter les commandes (ip addr, lynx), on tombre sur une page qui nous demande de s'identifier avec nos username et mot de passe. Un e fois fait, il suffit de faire shift+G (Aller à) pour rechercher la page https://www.duckduckgo.com. De la même manière, on peut se connecter à la page https://www.startpage.com on observe alors que cette page est en faite notre page d'acceuil sur un navigateur web affiché seulement en ligne de commande (sans graphique).
On constate que l'on peut naviguer sur internet avec des lignes de commandes lorsqu'on possède unsystème d'exploitation sans interface graphique.

##TP2
=====
####FEUILLE DE ROUTE

0) Configuration de la carte réseau de la VM "Gateway" en mode d'accès réseau par pont.

1) Il suffit de faire la commande apt-get install ssh (qui va installer toutes les autres librairies dont on aura besoin) ou avec la commande aptitude ssh.
Il faut néanmoins veiller avant celà à bien se connecter au réseau avec lynx.

2) Pour se connecter en ssh, il suffit de faire la commande ip a et de selectionner l'adresse IP de la VM Gateaway. Une fois cela réalisé, il faut lancer la commande ssh nomutilisateur@adresseIPdelaVM sur la terminal de la machine.
En essayant avec l'utilisateur root, un message apparait lorsqu'on entre le mot de passe qui nous informe que nous n'avons pas la permission.

3) Pour la modification du port de lecture de ssh, il suffit d'ouvrir le fichier sshd_config via la commande nano /etc/ssh/sshd_config et de modifier le port (ici 22 en 26).
Veiller ici à exécuter cette commande en super utilisateur sinon la permission ne sera pas accordée.
En réessayant la connection ssh quentin@192.168.99.132 sur le terminal, on observe que la connection est impossible puisqu'il faut préciser le port sur lequel on veut se connecter ce qui donne ssh quentin@192.168.99.132 -p 26.

4) Pour commencer, il faut lancer la commande ssh-keygen sur le terminal de l'ordinateur afin de créer une clé ssh. Suite à ça, il faut envoyer la clé publique vers le port de la machine souhaité (ici la VM) avec la commande ssh-copy-id -i ~/.ssh/id_rsa.pub quentin@192.168.99.132 -p 26. Après cela, le terminal nous demande le mot de passe de la session de la VM afin de se connecter dirrectement à la VM (sans demande de password) les prochaines fois.

5) Une fois l'échange de clé réalisé, il suffit de lancer la commande ssh quentin@192.168.99.132 -p 26

6) L'installation des paquets pour le serveur dhcp se fait grâce à la commande apt-get install isc-dhcp-server en veillant encore une fois à bien être connecter au réseau.
Note : On ne peut être connecter à internet avec la VM et l'ordinateur en même temps ?

7)
**Sur le terminal de l'odrinateur (en ssh)
mv dhcpd.conf dhcpd.conf.old (pour expliquer option)
fichier de base garder.

GATEAWAY
nano /etc/dhcp/dhcp.conf :

<>
	##########################################
	#####----- Global Configuration -----#####
	##########################################
	ddns-updates off;
	deny client-updates;
	one-lease-per-client false;
	autoriser le PXE
	allow booting;
	allow bootp;
	INTERFACES="eth0 eth1";
	ddns-update-style none;

	option domain-name "";
	option domain-name-servers    xxx.xxx.xxx.xxx, xxx.xxx.xxx.xxx;

	#default-lease-time 6000;
	max-lease-time 7200;

	authoritative;
	##############################################
	#####----- End Global Configuration -----#####
	##############################################

	####################################################
	########----- START CONF ETH1 NETWORK -----########
	####################################################
	subnet 192.168.3.0 netmask 255.255.255.0 {
		range 192.168.3.50 192.168.3.150;
		interface eth1;
		default-lease-time 6000;
		max-lease-time 7200;
		option subnet-mask 255.255.255.0;
		option routers 192.168.3.254;
		option broadcast-address 192.168.3.255;
		#l'addresse du serveur TFTP pour le boot PXE
		next-server 192.168.1.44;
	}
	##################################################
	########----- End CONF ETH1 NETWORK -----#########
	##################################################
	
	####################################################
	########----- START CONF ETH2 NETWORK -----#########
	####################################################
	subnet 192.168.2.0 netmask 255.255.255.0 {
		range 192.168.2.50 192.168.2.150;
		interface eth2;
		default-lease-time 6000;
		max-lease-time 7200;
		option subnet-mask 255.255.255.0;
		option routers 192.168.2.254;
		option broadcast-address 192.168.2.255;
	}
	##################################################
	########----- End CONF ETH1 NETWORK -----#########
	##################################################
	
	###################################################
	########----- START CONF ETH2 NETWORK -----########
	###################################################
	subnet 192.168.2.0 netmask 255.255.255.0 {
		range 192.168.2.50 192.168.2.150;
		interface eth2;
		default-lease-time 6000;
		max-lease-time 7200;
		option subnet-mask 255.255.255.0;
		option routers 192.168.2.254;
		option broadcast-address 192.168.2.255;
	}
	##################################################
	########----- End CONF ETH1 NETWORK -----#########
	##################################################
	
	#################################################
	#########----- START Adresses Fixe -----#########
	#################################################
	#Client Linux
	host server-web {
		hardware ethernet 08:00:27:ce:68:0b;
		fixed-address 192.168.2.10;
	  }
	
	
	host client {
		hardware ethernet 08:00:27:04:55:11;
		fixed-address 192.168.3.10;
	}

	### CLIENT PXE ETH1
	host clientPXE {
		hardware ethernet 08:00:27:C7:A8:CA;
		fixed-address 192.168.3.20;
		filename "lpxelinux.0";
	}
	#################################################
	#########----- END Adresses Fixe -----###########
	#################################################

<>
sudo service isc-dhcp-server restart`

client et serveur_web (que les trois premieres lignes)`
sudo nano /etc/network/interfaces :`

<>
	`# The primary network interface`
	`allow-hotplug eth0`
	`iface eth0 inet dhcp`

	`allow-hotplug eth1`
	`iface eth1 inet static`
	`        address 192.168.1.254`
	`        netmask 255.255.255.0`

	`allow-hotplug eth2`
	`iface eth2 inet static`
	`        address 192.168.2.254`
	`        netmask 255.255.255.0`


	**Sur la VM Serveur_web :
	`service networking restart`
	`ifconfig pour vérifier s'il a bien pris l'adresse écrit.`

	**Sur la VM Serveur_web :
	`service networking restart`
	`ifconfigeth1 pour vérifier s'il a bien pris l'adresse écrit. (grace à la mac adress (option hagrdware))`
	`ifconfigeth2 pour vérifier s'il a bien pris l'adresse écrit.`
<>

Configuration du réseau de Serveur web :
réseau en NAT, nom natNetwork2

Configuration du réseau de Gateaway : (3 cartes réseau)
la premiere:
réseau acces par pont, nom enp0s25
la deuxieme:
réseau NAT, nom natnetwork1
la troisieme:
réseau NAT, nom natnetwork2
deux réseau nat network:
natnetwork1, natnetwork2

*Sources:*
<https://www.molivier.com/isc-dhcp-server-debian.html>

##TP3
=====


##Routage
========
POUR ACTIVER LE ROUTAGE FAUT MODIFIER LE FICHIER DE CONF 
verifier si le routage est activer :
`sysctl net.ipv4.ip_forward`
si la valeur retourner est egal à 0 il faut l'activer :
2 façon :

`sysctl -w net.ipv4.ip_forward=1` 
( je crois que au redemarrage la valeur change, vaut mieux changer le fichier de conf)

ou

`/etc/sysctl/syctl.conf`

modifier la ligne et mettre =1

`net.ipv4.ip_forward=0 `


##Iptables
==========
<>
	#!/bin/sh
	### BEGIN INIT INFO
	# Provides: firewall.sh
	# Required-Start: $networking
	# Required-Stop: $local_fs
	# X-Start-Before: $ifupdown
	# X-Start-After: $ifupdown
	# Default-Start: S
	# Default-Stop:       0 6
	# Short-description: Firewall
	# Description: Mise en places des règles iptables
	### END INIT INFO
	# Vider les tables actuelles
	iptables -t filter -F

	# Vider les règles personnelles
	iptables -t filter -X

	# Interdire toute connexion entrante et sortante
	iptables -t filter -P INPUT DROP
	iptables -t filter -P FORWARD DROP
	iptables -t filter -P OUTPUT DROP

	# ---

	# Ne pas casser les connexions etablies
	iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
	iptables -A OUTPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
	iptables -A FORWARD -m state --state RELATED,ESTABLISHED -j ACCEPT

	# Autoriser loopback
	iptables -t filter -A INPUT -i lo -j ACCEPT
	iptables -t filter -A OUTPUT -o lo -j ACCEPT

	# ICMP (Ping)
	#iptables -t filter -A INPUT -p icmp -j ACCEPT
	#iptables -t filter -A OUTPUT -p icmp -j ACCEPT

	# ---

	# SSH In PORT 26
	iptables -t filter -A INPUT -p tcp --dport 26 -j ACCEPT

	# SSH Out 22
	iptables -t filter -A OUTPUT -p tcp --dport 22 -j ACCEPT

	# DNS In/Out
	iptables -t filter -A OUTPUT -p tcp --dport 53 -j ACCEPT
	iptables -t filter -A OUTPUT -p udp --dport 53 -j ACCEPT
	iptables -t filter -A INPUT -p tcp --dport 53 -j ACCEPT
	iptables -t filter -A INPUT -p udp --dport 53 -j ACCEPT

	# HTTP + HTTPS Out
	iptables -t filter -A OUTPUT -p tcp --dport 80 -j ACCEPT
	iptables -t filter -A OUTPUT -p tcp --dport 443 -j ACCEPT
	iptables -t filter -A OUTPUT -p tcp --dport 8080 -j ACCEPT

	# HTTP + HTTPS In
	iptables -t filter -A INPUT -p tcp --dport 80 -j ACCEPT
	iptables -t filter -A INPUT -p tcp --dport 443 -j ACCEPT
	iptables -t filter -A INPUT -p tcp --dport 8080 -j ACCEPT

	# REGLE POUR LE PXE: TFTP

	iptables -t filter -A INPUT -p udp --dport 69 -j ACCEPT
	iptables -t filter -A OUTPUT -p udp --dport 69 -j ACCEPT
	#iptables -t filter -A FORWARD -p udp --dport 69 -j ACCEPT

	# REGLE FORWARD ETH1 VERS ETH2
	iptables -t filter -A FORWARD -i eth1 -o eth2 -p tcp --dport 80 -j ACCEPT
	iptables -t filter -A FORWARD -i eth1 -o eth2 -p tcp --dport 443 -j ACCEPT
	iptables -t filter -A FORWARD -i eth1 -o eth2 -p tcp --dport 115 -j ACCEPT
	iptables -t filter -A FORWARD -i eth1 -o eth2 -p tcp --dport 22 -j ACCEPT

	#PING
	#iptables -t filter -A FORWARD -i eth1 -o eth2 -p icmp -j ACCEPT
	#iptables -t filter -A FORWARD -i eth1 -o eth0 -p icmp -j ACCEPT

	#REGLE FORWARD ETH2 VERS ETH1
	iptables -t filter -A FORWARD -i eth2 -o eth1 -p tcp --dport 80 -j ACCEPT
	iptables -t filter -A FORWARD -i eth2 -o eth1 -p tcp --dport 443 -j ACCEPT

	#PING
	#iptables -t filter -A FORWARD -i eth2 -o eth1 -p icmp -j ACCEPT
	#iptables -t filter -A FORWARD -i eth2 -o eth0 -p icmp -j ACCEPT


	#DEFAULT FORWARD ROUTE (ETH1,2 VERS ETH0)
	iptables -t filter -A FORWARD -i eth2 -o eth0 -j ACCEPT
	iptables -t filter -A FORWARD -i eth1 -o eth0 -j ACCEPT

	#REGLE NAT POSTROUTING
	iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
<>
IPTABLES_NAT
camoufle tout ce qui sort par l'interface eth0 qui est l'interface connecté à internet
iptables -t nat -A POSTROUTING -o eth0 MASQUERADE

vérifier que c'est bien ajouté
iptables -t nat -L -v


##DNS
====

modifier le /etc/resolv.conf pour avoir un DNS sur chaque machine (CLIENT, SRV_WEB)
ajouter les route par defauts de chacune des machine si elle n'y sont pas.

etant donnée que la route par defaut est l'intarface 1 pour chaque machine.
route add default gw dev eth0.

##BONUS PXE
===========

#Serveur apache2
===============

<VirtualHost *:80>

        ServerName pxe
        ServerAlias *.pxe
        ServerAlias 192.168.1.44

        ServerAdmin webmaster@pxe
        UseCanonicalName Off
        ServerSignature Off

        DocumentRoot /var/www/pxe/htdocs


        ErrorLog /var/www/pxe/log/error.log
        CustomLog /var/www/pxe/log/access.log combined

        <IfModule mod_dir.c>
                Directoryindex index.html index.php index.php
        </IfModule>

        <Directory '/var/www/pxe/htdocs'>
                AllowOverride None
                Options FollowSymLinks
                Order Deny,Allow
                Deny from none
                Allow from all
        </Directory>

        <Directory '/var/www/pxe'>
                AllowOverride All
                Options Indexes FollowSymLinks MultiViews
                Order Deny,Allow
                Deny from none
                Allow from all
        </Directory>

        <IfModule mod_alias.c>
                Alias /webboot /var/www/pxe/distributions
        </IfModule>
</VirtualHost>

#Serveur tftp-hpa
================

# /etc/default/tftpd-hpa

TFTP_USERNAME="tftp"
TFTP_DIRECTORY="/var/www/pxe"
TFTP_ADDRESS="192.168.1.44:69"
TFTP_OPTIONS="--secure"
RUN_DAEMON="yes"




#Modules 
========
Pour le pxe avec un firewall faut mettre ajouter le 
`modprobe nf_conntrack_tftp`
pour que ce module reste actif apres un redemarrage il faut l'ajouter dans le fichier
`/etc/modules`

c'est obligatoir car quand le client recupere le fichier pxelinux.0 il envoi sa demande sur le port 69 mais le serveur lui repond sur un autre port > 1024.

