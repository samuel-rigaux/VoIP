# Rapport sur Asterisk pour un serveur VoIP

## Introduction et contextualisation

Dans un monde où les communications téléphoniques sont omniprésentes, les systèmes de téléphonie sur IP (VoIP) offrent des solutions flexibles et économiques pour les entreprises et les particuliers. Asterisk, une plateforme open-source, permet de créer un serveur VoIP performant et personnalisable. Ce rapport présente l'installation, la configuration et l'utilisation d'Asterisk pour mettre en place un système téléphonique complet, y compris un serveur interactif et un automate d'appel.

## Présentation fonctionnelle

Asterisk est un logiciel libre permettant la mise en place d'un serveur VoIP. Il permet de gérer des appels, d'automatiser les réponses et d'assurer une communication sécurisée via le protocole TLS. Nous avons suivi les étapes suivantes pour l'installation et la configuration :

1.  Installation des paquets nécessaires.
2.  Téléchargement et compilation d'Asterisk.
3.  Configuration des utilisateurs dans `pjsip.conf`.
4.  Sécurisation des appels par TLS.
5.  Mise en place d'un serveur vocal interactif (IVR).
6.  Automatisation des appels via un script shell.

## Avantages et inconvénients

### Avantages

-   Open-source et gratuit.
-   Hautement personnalisable.
-   Support de multiples protocoles VoIP (SIP, IAX, etc.).
-   Sécurisation via TLS et certificats SSL.
-   Intégration avec d'autres systèmes (bases de données, applications tierces).

### Inconvénients

-   Configuration complexe.
-   Nécessite des compétences en administration système et réseau.
-   Peut être gourmand en ressources pour les grandes infrastructures.

## Solutions existantes sur le marché

### Solutions open-source

-   **Asterisk** : La solution la plus populaire pour les serveurs VoIP.
-   **FreeSWITCH** : Une alternative à Asterisk avec des fonctionnalités avancées.
-   **Kamailio** : Un proxy SIP performant et modulaire.

### Solutions payantes

-   **3CX** : Solution VoIP complète avec interface graphique conviviale.
-   **Cisco Unified Communications** : Plateforme de téléphonie IP pour les grandes entreprises.
-   **Avaya IP Office** : Une solution pour les PME avec des options avancées.

## Exemples d’implémentation

### Mise en place d'un IVR (Serveur Vocal Interactif)

Nous avons configuré un menu vocal interactif dans Asterisk permettant aux utilisateurs de sélectionner différents services :

```ini
[ivr1]
exten => s,1,Answer()
exten => s,2,Set(TIMEOUT(response)=10)
exten => s,3,agi(googletts.agi,"Bonjour et bienvenue chez La Plateforme !",fr,any)
exten => s,4,agi(googletts.agi,"Pour joindre Alice, taper 1.",fr,any)
exten => s,5,agi(googletts.agi,"Pour joindre Bob, taper 2.",fr,any)
exten => s,6,agi(googletts.agi,"Pour joindre Malcom, taper 3.",fr,any)
exten => s,7,WaitExten()

exten =>1,1,Dial(PJSIP/alice,10)
exten =>2,1,Dial(PJSIP/bob,10)
exten =>3,1,Dial(PJSIP/malcom,10)

exten => [04-9#],1,agi(googletts.agi,"Entrée invalide",fr,any)
exten => _[04-9#],2,Goto(ivr_1,s,1)
exten => t,1,Goto(ivr_1,s,3)

```

### Automatisation des appels

Nous avons mis en place un script Bash permettant de générer automatiquement des appels à partir d’un fichier CSV contenant une liste de contacts.

```bash
#!/bin/bash

CSV_FILE="contact.csv"
CALLS_DIR="/var/spool/asterisk/outgoing/"
CALLER_ID="9000"

# Vérification de l'existence du fichier CSV
if [[ ! -f "$CSV_FILE" ]]; then
    echo "❌ Erreur : le fichier $CSV_FILE n'existe pas."
    exit 1
fi

# Lecture du fichier CSV
tail -n +2 "$CSV_FILE" | while IFS=, read -r NAME NUMBER; do
    if [[ -z "$NAME" || -z "$NUMBER" ]]; then
               continue
    fi

    CALL_FILE="/tmp/call_$NUMBER.call"
    
    echo "📞 Génération de l'appel pour $NAME ($NUMBER)..."

    cat <<EOF > "$CALL_FILE"
# Récupération des données du fichier CSV
Channel: PJSIP/$NUMBER
CallerID: "Prospection Automatique" <$CALLER_ID>
MaxRetries: 2
RetryTime: 60
WaitTime: 30
Context: auto_calls
Extension: s
Priority: 1
EOF

# Changement des autorisations
    chmod 777 "$CALL_FILE"
    mv "$CALL_FILE" "$CALLS_DIR/"
    echo "✅ Appel généré pour $NAME ($NUMBER)"
done

```

Dans Asterisk, nous avons configuré un contexte permettant de diffuser un message préenregistré aux contacts appelés :

```ini
[auto_calls]
exten => s,1,Answer()
same => n,Wait(1)
same => n,AGI(googletts.agi,"Bonjour, ceci est un appel automatique. Nous vous présentons l'école de la plateforme, un établissement innovant où vous pouvez développer vos compétences et acquérir de nouvelles",fr)
same => n,Hangup()

```

## Conclusion

Asterisk est une solution puissante et flexible pour mettre en place un serveur VoIP. Grâce à son architecture modulaire, il permet de répondre à divers besoins, tels que l'automatisation des appels et la mise en place d’un serveur vocal interactif. Bien que sa configuration soit technique, il offre une alternative robuste aux solutions payantes. En mettant en place un système IVR et un automate d'appel, nous avons démontré son potentiel dans un environnement professionnel.
