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

CSV_FILE="contacts.csv"
CALLS_DIR="/var/spool/asterisk/outgoing/"
CALLER_ID="LaPlateforme_"

# Vérifier si le fichier CSV existe
if [[ ! -f "$CSV_FILE" ]]; then
    echo "❌ Erreur : le fichier $CSV_FILE n'existe pas."
    exit 1
fi

# Sélectionner un contact au hasard dans le fichier CSV (sans l'entête)
SELECTED_CONTACT=$(tail -n +2 "$CSV_FILE" | shuf -n 1)

# Extraire le nom et le numéro
IFS=',' read -r NAME NUMBER <<< "$SELECTED_CONTACT"

# Vérifier que les champs ne sont pas vides
if [[ -z "$NAME" || -z "$NUMBER" ]]; then
    echo "⚠️ Contact invalide, sélection d'un autre..."
    exit 1
fi

CALL_FILE="/tmp/call_$NUMBER.call"

echo "📞 Génération de l'appel pour $NAME ($NUMBER)..."

cat <<EOF > "$CALL_FILE"
Channel: PJSIP/$NUMBER
CallerID: "LaPlateforme_" <$CALLER_ID>
MaxRetries: 2
RetryTime: 60
WaitTime: 30
Context: outgoing-calls
Extension: s
Priority: 1
EOF

# Vérifier que le fichier a été créé
if [[ ! -f "$CALL_FILE" ]]; then
    echo "❌ Erreur : Impossible de créer le fichier d'appel pour $NAME ($NUMBER)."
    exit 1
fi

# Déplacer le fichier avec les bonnes permissions
chmod 777 "$CALL_FILE"
mv "$CALL_FILE" "$CALLS_DIR/"

echo "✅ Appel généré pour $NAME ($NUMBER)"

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
