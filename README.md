# Simulation de diffusion en DAB+

Depuis l’article sur [le Raspberry et la radio](https://technic2radio.fr/raspberry-pi-radio/), je fouille le net à la recherche d’information dans le but d’écrire un article sur la diffusion en DAB+ et en particulier avec l’utilisation d’un Raspberry Pi. Autant on peut transformer un RPI en micro-émetteur FM (très faible puissance), autant il est impossible de diffuser en DAB+ sans passer par un “module” supplémentaire dont voici la liste :

- EasyDAB v2 : http://tipok.org.ua/node/46
- Ettus USRP B100, B200, USRP1 … : https://www.ettus.com/
- HackRF : https://greatscottgadgets.com/hackrf/

Plus d’infos sur les modules complémentaires :
http://wiki.opendigitalradio.org/DAB_hardware

L'objectif de ce tutoriel est de simuler une diffusion en DAB+. 

# OpenDigitalRadio

Coté software, je me suis concentré sur la solution open-source : [OpenDigitalRadio](http://www.opendigitalradio.org/). J’ai testé l’installation d’OpenDigitalRadio sur un RPI grâce au script très bien documenté sur github : https://github.com/glokhoff/RaspDAB.

Pour le test, j’ai utilisé un RPI 3 avec [Raspbian Jessie](http://downloads.raspberrypi.org/raspbian/images/raspbian-2017-07-05/)

## Préparation

    $ sudo apt-get update
    $ sudo apt-get upgrade
    $ sudo nano /boot/config.txt

Et ajouter les deux lignes suivantes :

    dtoverlay=pi3-disable-bt
    dtoverlay=pi3-disable-wifi

Ajouter l’utilisateur “odr” et donner lui un mot de passe

    $ sudo adduser odr

Puis modifier les droits de “odr”

    $ sudo visudo -f /etc/sudoers

Ajouter la ligne suivante après “root All=(ALL:ALL) ALL”

    odr ALL=(ALL:ALL) ALL

Et rebootez votre Raspberry Pi :

    $ sudo reboot

Se connecter en tant qu’utilisateur “odr”

    $ su odr
    $ cd

Installation de ODR mmbTools d’OpenDigitalRadio

    $ sudo nano /etc/apt/sources.list

supprimer le “#” au début de la ligne commençant par “deb-src”

    $ sudo apt-get update

Nous allons utiliser un fork du script initial “Raspdab” disponible sur ce lien : https://github.com/glokhoff/RaspDAB . À l’instar du script initial, vous aurez la prise en charge d’ALSA et l’installation d’ODR-DabMod

    $ git clone https://github.com/LyonelB/RaspDAB.git
    $ cd RaspDAB
    $ chmod +x raspdab.sh
    $ ls -l
    $ ./raspdab.sh

Appuyer sur “Enter” et allez boire un café, l’installation dure à peu près deux heures …
A la fin de l'installation, si vous souhaitez vérifier que ODR-Audienc fonctionnne, vos pouvez ouvrir "l'aide" :

    $ odr-audioenc -h

# Installation de Supervisor et création des fichiers de configuration

    $ sudo apt-get install supervisor
    $ mkdir config
    $ cd config
    $ mkdir supervisor
    $ mkdir mot
    $ cd
    $ touch /home/odr/config/mot/radio1.txt
    $ mkfifo /home/odr/config/mot/radio1.pad
    $ sudo mv /home/odr/RaspDAB/enc-radio1.conf /home/odr/config/supervisor
    
Vous pouvez éditer le fichier de configuration pour modifier l'url de streaming de la "radio1" avec la commande : 

    $ sudo nano /home/odr/config/supervisor/enc-radio1.conf
    
Il faut renouveller la création des fichiers pour la seconde radio :

    $ touch /home/odr/config/mot/radio2.txt
    $ mkfifo /home/odr/config/mot/radio2.pad
    $ sudo nano /home/odr/config/supervisor/enc-radio2.conf
    $ sudo mv /home/odr/RaspDAB/enc-radio2.conf /home/odr/config/supervisor
    
On passe maintenant à la configuration du "mux" :

    $ sudo mv /home/odr/RaspDAB/conf.mux /home/odr/config
    
On ajoute des "liens" à supervisor :

    $ sudo ln -s /home/odr/config/supervisor/enc-radio1.conf /etc/supervisor/conf.d/enc-radio1.conf
    $ sudo ln -s /home/odr/config/supervisor/enc-radio2.conf /etc/supervisor/conf.d/enc-radio2.conf
    $ sudo nano /etc/supervisor/supervisord.conf

et ajoutez les lignes suivantes :

    [inet_http_server]
    port = 9100
    username = user ; Auth username
    password = pass ; Auth password

Pour que les fichiers de configuration soient pris en compte par supervisor :

    $ sudo supervisorctl reread
    $ sudo supervisorctl update

et rendez-vous sur l’ip de votre raspberry : http://xxx.xxx.x.xxx:9100

![Supervisor Statuts](https://github.com/LyonelB/RaspDAB/raw/master/Supervisor%20Status.png)

# ODR-DabMux

Tout est prêt ! Il suffit de lancer la commande suivante :

    $ odr-dabmux /home/odr/config/conf.mux
    
Votre "multiplexeur" est prêt ! Vous avez maintenant un seul flux au format [ETI](http://wiki.opendigitalradio.org/Ensemble_Transport_Interface) contenant les flux audio des radios et leurs data sur le port 18081.

# Installation de Dablin

    $ sudo apt-get install git gcc g++ cmake
    $ sudo apt-get install libmpg123-dev libfaad-dev libsdl2-dev libgtkmm-3.0-dev
    $ git clone https://github.com/Opendigitalradio/dablin.git
    $ cd dablin
    $ mkdir build
    $ cd build
    $ cmake ..
    $ make
    $ sudo make install
    
# Installation de ZMQ ETI Receiver

    $ git clone https://github.com/mpbraendli/mmbtools-aux.git
    $ cd mmbtools-aux/zmqtest/zmq-sub
    $ make
    
# Lecture du flux ETI via DABlin

    $ ./zmq-sub 127.0.0.1 18081 | dablin -s 0x8daa
    
Pour lancer DABlin avec l'interface graphique, si votre Raspberry Pi est relié à un écran

    $ ./zmq-sub 127.0.0.1 9100 | dablin_gtk -s 0x8daa #

# Notes

→ Pas besoin de DABMod : modifier le fichier raspdab.sh

https://groups.google.com/forum/#!topic/crc-mmbtools/etUFcqdZSmc

http://wiki.opendigitalradio.org/Etisnoop

https://github.com/mpbraendli/mmbtools-aux/tree/master/zmqtest/zmq-sub

