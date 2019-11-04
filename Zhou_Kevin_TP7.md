# Exercice 1. Personnalisation de GRUB
GRUB est considérablement paramétrable : résolution, langue, fond d’écran, thème, disposition du clavier….
 GRUB se configure via un fichier de paramètres (/etc/default/grub), mais aussi par des
scripts situés dans /etc/grub.d ; ces scripts commencent tous par un numéro et sont traités dans
l’ordre.
 Evidemment, seuls les scripts exécutables sont pris en compte.
 Sous Ubuntu Server, GRUB prend aussi en compte les fichiers d’extension .cfg présents
dans /etc/default/grub.d. En particulier, sur les versions récentes, le fichier de configuration
50-curtin-settings.cfg donne à la variable GRUB_TERMINAL la valeur console, ce qui désactive
tous les paramètres liés aux fonds d’écran, thèmes, certaines résolutions, etc.

**1. Commencez par changer l’extension du fichier /etc/default/grub.d/50-curtin-settings.cfg s’il
est présent dans votre environnement (vous pouvez aussi commenter son contenu).**
```bash
kz@serveur:~$ sudo mv /etc/default/grub.d/50-curtin-settings.cfg /etc/default/grub.d/50-curtin-settings.cfg.old
```
**2. Modifiez le fichier /etc/default/grub pour que le menu de GRUB s’affiche pendant 10 secondes ;
passé ce délai, le premier OS du menu doit être lancé automatiquement.**

```bash

GRUB_TIMEOUT=10
kz@serveur:~$ sudo nano /etc/default/grub
```


**3. Lancez la commande update-grub
 Cette commande fait appel au script grub-mkconfig qui construit le fichier de configuration
”final” de GRUB (/boot/grub/grub.cfg) à partir du fichier de paramètres et des scripts.**

```bash
kz@serveur:~$ sudo update-grub
```

**4. Redémarrez votre VM pour valider que les changements ont bien été pris en compte
 Pensez à lancer la commande update-grub après chaque modification de la configuration de
GRUB !**

*FAIT*

**5. On va augmenter la résolution de GRUB et de notre VM. Cherchez sur Internet le ou les paramètres
à rajouter au fichier grub.**
```bash
GRUB_GFXMODE=1024x768
GRUB_GFXPAYLOAD_LINUX=keep
update-grub
```
**6. On va à présent ajouter un fond d’écran. Il existe un paquet en proposant quelques uns : grub2-splash-images
(après installation, celles-ci sont disponibles dans /usr/share/images/grub).**

*wget pour telecharger une image*
```bash
apt-get install grub2-splashimages
GRUB_BACKGROUND=/usr/share/images/grub/Plasma-lamp.tga
sudo update-grub

```
**7. Il est également possible de configurer des thèmes. On en trouve quelques uns dans les dépôts (grub2-themes-*).
Installez-en un.**

```bash
kz@serveur:~$ sudo nano /etc/default/grub
GRUB_THEME=/boot/grub/themes/ubuntu-mate/progress_frame_c.png
kz@serveur:~$ sudo update-grub
```

**8. Ajoutez une entrée permettant d’arrêter la machine, et une autre permettant de la redémarrer.**

```bash
kz@serveur:~$ sudo nano /etc/grub.d/40_custom

#!/bin/sh
exec tail -n +3 $0
# This file provides an easy way to add custom menu entries.  Simply type the
# menu entries you want to add after this comment.  Be careful not to change
# the 'exec tail' line above.

menuentry "arret machine"{
halt
}

menuentry "reboot machine"{
reboot
}


kz@serveur:~$ sudo update-grub
```

**9. Configurer GRUB pour que le clavier soit en français**

```bash
kz@serveur:~$ sudo nano /etc/grub.d/40_custom

#!/bin/sh
exec tail -n +3 $0
# This file provides an easy way to add custom menu entries.  Simply type the
# menu entries you want to add after this comment.  Be careful not to change
# the 'exec tail' line above.
#clavier Fr
insmod keylayouts
keymap fr

kz@serveur:~$ sudo update-grub
```

# Exercice 2. Noyau
Dans cet exercice, on va créer et installer un module pour le noyau.

**1. Commencez par installer le paquet build-essential, qui contient tous les outils nécessaires (compilateurs, bibliothèques) à la compilation de programmes en C (entre autres).**

```bash
apt-get install build-essential
```
2. Créez un fichier hello.c contenant le code suivant :
1 #include <linux/module.h>
2 #include <linux/kernel.h>
3
4 MODULE_LICENSE("GPL");
5 MODULE_AUTHOR("John Doe");
6 MODULE_DESCRIPTION("Module hello world");
7 MODULE_VERSION("Version 1.00");
8
9 int init_module(void)
10 {
11 printk(KERN_INFO "[Hello world] - La fonction init_module() est appelée.\n");
12 return 0;
13 }
14
15 void cleanup_module(void)
16 {
17 printk(KERN_INFO "[Hello world] - La fonction cleanup_module() est appelée.\n");
18 }
```bash
kz@serveur:~$ touch hello.c
```
3. Créez également un fichier Makefile :
1 obj-m += hello.o
2
3 all:
4 make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules
5
6 clean:
7 make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean
8
9 install:
10 cp ./hello.ko /lib/modules/$(shell uname -r)/kernel/drivers/misc
 Les lignes 4, 7 et 10 doivent commencer par une tabulation.
```bash
kz@serveur:~$ touch Makefile
```
4. Compilez le module à l’aide de la commande make, puis installez-le à l’aide de la commande make
install.
 Le module est installé dans le dossier spécifié à la ligne 10.
```bash
kz@serveur:~$ make
make -C /lib/modules/5.0.0-31-generic/build M=/home/kz modules
make[1]: Entering directory '/usr/src/linux-headers-5.0.0-31-generic'
  Building modules, stage 2.
  MODPOST 1 modules
make[1]: Leaving directory '/usr/src/linux-headers-5.0.0-31-generic'

kz@serveur:~$ sudo make install
[sudo] password for kz:
cp ./hello.ko /lib/modules/5.0.0-31-generic/kernel/drivers/misc
```
5. Chargez le module ; vérifiez dans le journal du noyau que le message ”La fonction init_module() est
appelée” a bien été inscrit, synonyme que le module a été chargé ; confirmez avec la commande lsmod.

```bash

```
6. Utilisez la commande modinfo pour obtenir des informations sur le module hello.ko ; vous devriez
notamment voir les informations figurant dans le fichier C.
7. Déchargez le module ; vérifiez dans le journal du noyau que le message ”La fonction cleanup_module()
est appelée” a bien été inscrit, synonyme que le module a été déchargé ; confirmez avec la commande
lsmod.
8. Pour que le module soit chargé automatiquement au démarrage du système, il faut l’inscrire dans le
fichier /etc/modules. Essayez, et vérifiez avec la commande lsmod après redémarrage de la machine.
