Depuis l’article sur le Raspberry et la radio, je fouille le net à la recherche d’information dans le but d’écrire un article sur la diffusion en DAB+ et en particulier avec l’utilisation d’un Raspberry Pi. Autant on peut transformer un RPI en micro-émetteur FM (très faible puissance), autant il est impossible de diffuser en DAB+ sans passer par un “module” supplémentaire dont voici la liste :

- EasyDAB v2 : http://tipok.org.ua/node/46
- Ettus USRP B100, B200, USRP1 … : https://www.ettus.com/
- HackRF : https://greatscottgadgets.com/hackrf/

Plus d’infos sur les modules complémentaires :
http://wiki.opendigitalradio.org/DAB_hardware

# OpenDigitalRadio

Coté software, je me suis concentré sur la solution open-source : OpenDigitalRadio. J’ai testé l’installation d’OpenDigitalRadio sur un RPI grâce au script très bien documenté sur github : https://github.com/glokhoff/RaspDAB

Pour le test, j’ai utilisé un RPI 3 avec Raspbian Jessie

## Préparation
```
$ sudo apt-get update
$ sudo apt-get upgrade
$ sudo nano /boot/config.txt
```

Et ajouter les deux lignes suivantes :

>dtoverlay=pi3-disable-bt
>dtoverlay=pi3-disable-wifi

Ajouter l’utilisateur “odr” et donner lui un mot de passe

`$ sudo adduser odr`

Puis modifier les droits de “odr”

`$ sudo visudo -f /etc/sudoers`

Ajouter la ligne suivante après “root All=(ALL:ALL) ALL”

odr ALL=(ALL:ALL) ALL

`$ sudo reboot`

Se connecter en tant qu’utilisateur “odr”

```
$ su odr
$ cd
```

Installation de ODR mmbTools d’OpenDigitalRadio

`$ sudo nano /etc/apt/sources.list`

supprimer le “#” au début de la ligne commençant par “deb-src”

`$ sudo apt-get update`

Nous allons utiliser un fork du script initial “Raspdab” disponible sur ce lien : https://github.com/glokhoff/RaspDAB . À l’instar du script initial, vous aurez la prise en charge d’ALSA et l’installation d’ODR-DabMod

```
$ git clone https://github.com/LyonelB/RaspDAB.git
$ cd RaspDAB
$ chmod +x raspdab.sh
$ ls -l
$ ./raspdab.sh
```

Appuyer sur “Enter” et allez boire un café, l’installation dure à peu près deux heures …
Tester si odr-audioenc fonctionne et ouvrir l’aide :

`$ odr-audioenc -h`

# Installation de Supervisor et création des fichiers de configuration

```
$ sudo apt-get install supervisor
$ mkdir config
$ cd config
$ mkdir supervisor
$ mkdir mot
$ touch /home/odr/config/mot/radio1.txt
$ mkfifo /home/odr/config/mot/radio1.pad
$ sudo nano /home/odr/config/supervisor/enc-radio1.conf
```

Ajoutez le texte : https://raw.githubusercontent.com/LyonelB/RaspDAB/master/enc-radio1.conf

```
$ touch /home/odr/config/mot/radio2.txt
$ mkfifo /home/odr/config/mot/radio2.pad
$ sudo nano /home/odr/config/supervisor/enc-radio2.conf
```

Ajoutez le texte : https://raw.githubusercontent.com/LyonelB/RaspDAB/master/enc-radio2.conf

`$ sudo nano /home/odr/config/conf.mux`

Ajoutez le texte : https://raw.githubusercontent.com/LyonelB/RaspDAB/master/conf.mux

```
$ sudo ln -s /home/odr/config/supervisor/enc-radio1.conf /etc/supervisor/conf.d/enc-radio1.conf
$ sudo ln -s /home/odr/config/supervisor/enc-radio2.conf /etc/supervisor/conf.d/enc-radio2.conf
$ sudo nano /etc/supervisor/supervisord.conf
```

et ajoutez les lignes suivantes :

>[inet_http_server]
>port = 9100
>username = user ; Auth username
>password = pass ; Auth password

```
$ sudo supervisorctl reread
$ sudo supervisorctl update
```

et rendez-vous sur l’ip de votre raspberry : http://xxx.xxx.x.xxx:9100

# ODR-DabMux

Tout est prêt ! Il suffit de lancer la commande suivante :

`$ odr-dabmux /home/odr/config/conf.mux`
