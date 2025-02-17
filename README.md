# DOCUMENTATION

Pour l’installation d’asterisk, j’ai suivi ce [tutoriel](https://doc.ubuntu-fr.org/asterisk).

En premier on installe les paquets :

    sudo apt install build-essential libxml2-dev libncurses5-dev linux-headers-$(uname -r) libsqlite3-dev libssl-dev libedit-dev uuid-dev libjansson-dev

Ensuite on créer le fichier asterisk comme ceci :

    mkdir /usr/src/asterisk

Et on se rend dans ce même dossier :

    cd /usr/src/asterisk

Et on install asterisk :

    wget : [https://downloads.asterisk.org/pub/telephony/asterisk/asterisk-22-current.tar.gz](https://downloads.asterisk.org/pub/telephony/asterisk/asterisk-22-current.tar.gz)

et on décompresse le fichier :

    tar -xvzf asterisk-22-current.tar.gz

on se déplace dans le fichier :

    cd asterisk-22.2.0/

et on lance la configuration avec :

    sudo ./configure --with-jansson-bundled

Il manque un paquet, donc on l’installe avec :

    sudo apt install pkg-config

On relance la configuration avec :

    sudo ./configure --with-jansson-bundled

Finalement, on fait :

    sudo make menuselect

Dans le menu qui s'affiche, je suis allé dans **Core Sound Package** et j’ai coché à l'aide de la touche Espace **CORE-SOUNDS-FR-ULAW**. Je quitte en pressant la touche Echap. Je vais ensuite dans **Music On Hold File Packages**, Je décoche **MOH-OPSOUND-WAV** et je coche **MOH-OPSOUND-ULAW**. Enfin, je vais dans **Extras Sound Packages** et je coche **EXTRA-SOUNDS-FR-ULAW**.

Je reviens à l'écran principal et j’appuie sur Echap pour terminer et je presse S pour sauvegarder.

Enfin je tape les commandes suivantes pour terminer l’installation :

    sudo make
    
    sudo make install
    
    sudo make samples
    
    sudo make config

Enfin je lance Asterisk avec :

    /etc/init.d/asterisk start

et je lance la console d’asterisk avec :

    sudo asterisk -rvvvv

J’ai ensuite procédé à la configuration des utilisateurs. J’ai d’abord modifié le fichier `pjsip.conf` en faisant :

    cd /etc/asterisk

    sudo nano pjsip.conf

Dans ce fichier, j’ai ajouté les configurations suivantes :

    language=fr; Default language setting for all users/peers

et aussi :

    [general]
    hasvoicemail = yes
    hassip = yes
    hasiax = yes
    callwaiting = yes
    threewaycalling = yes
    callwaitingcallerid = yes
    transfer = yes
    canpark = yes
    cancallforward = yes
    callreturn = yes
    callgroup = 1
    pickupgroup = 1
    nat = yes

Ensuite j’ai rajouter les lignes de configurations sans template :

    [6001]  ; Numéro SIP  
    type=friend  ; Type d'objet SIP (friend = utilisateur)
    host=dynamic  ; Vous pouvez vous connecter à ce compte SIP à partir de n’importe quelle adresse IP
    dtmfmode=rfc2833  ; Mode du DTMF
    disallow=all  ; Désactiver tous les codecs
    allow=ulaw  ; Activer les codecs µlaw
    fullname = Samuel RIGAUX  ; Nom complet de l'utilisateur (ce qui s'affiche sur le téléphone)
    username = samuel  ; Nom d'utilisateur
    secret=root  ; Mot de passe
    context = work  ; Contexte (exploité par le fichier extensions.conf)

