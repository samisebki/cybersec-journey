# Linux - OverTheWire Bandit

## Connexion
- `ssh utilisateur@serveur -p 2220` : connexion à distance (SSH, port 2220 ici au lieu de 22)
- `exit` : se déconnecter

## Se déplacer et lister
- `ls` : lister les fichiers ; `ls -a` : afficher aussi les fichiers cachés (nom qui commence par un point)
- `cd dossier` : entrer dans un dossier ; `cd -` : revenir au dossier précédent
- `.` = dossier actuel, `..` = dossier parent, `/` = racine du système

## Lire des fichiers
- `cat fichier` : afficher le contenu
- `cat -` lit le clavier (entrée standard) : pour un fichier nommé `-`, écrire `./-`
- Nom avec des espaces : guillemets `"mon fichier"`, `\` devant chaque espace, ou **Tab** pour compléter
- `file fichier` : donne le vrai type du contenu (texte, données...), sans se fier à l'extension
- `*` (joker) : n'importe quels caractères, ex. `file ./*`

## Chercher des fichiers avec find
- Forme : `find <où> <critères>`
- `-type f` : fichiers seulement · `-size 1033c` : taille en octets (`c`) · `! -executable` : non exécutable
- `-user nom` / `-group nom` : propriétaire et groupe
- `find / ...` : chercher sur tout le système
- `2>/dev/null` : cacher les messages d'erreur (`2>` = erreurs, `/dev/null` = poubelle)

## Utile
- `man commande` : manuel (`/` pour chercher un mot, `q` pour quitter)
- `history` : commandes déjà tapées ; `!12` relance la n° 12 ; flèche du haut = commande précédente

## Lien avec la sécurité
- Fichiers cachés et dossiers système : endroits où un attaquant cache des fichiers
- `find` par propriétaire, taille ou droits : réflexe d'investigation et d'élévation de privilèges en pentest
## Filtrer et enchaîner les commandes
- `grep mot fichier` : afficher les lignes qui contiennent un mot (`-w` = mot entier)
- `sort` : trier les lignes · `uniq -u` : garder les lignes uniques (le fichier doit être trié avant)
- Pipe `|` : envoie le résultat d'une commande dans la suivante, ex. `sort data.txt | uniq -u`. On ne redonne pas le nom du fichier à la deuxième commande.
- `strings fichier` : extraire le texte lisible d'un fichier binaire (base de l'analyse de malware)
- Usage SOC : `grep "Failed password" auth.log` pour repérer les tentatives SSH ratées
- 
## Encodage, compression, réseau (Bandit 10 à 19)
- `base64 -d` : décoder du Base64. Encoder ≠ chiffrer : aucune clé, tout le monde peut décoder
- `tr 'a-zA-Z' 'n-za-mN-ZA-M'` : ROT13 (chiffre de César), cassé en une ligne
- `xxd -r` : hexdump → fichier · `file` reconnaît un fichier à sa signature (magic bytes), pas à son nom
- gzip (`gunzip`), bzip2 (`bunzip2`), tar (`tar -xf`) : `>` crée le fichier AVANT la commande
- `mktemp -d` : dossier de travail temporaire
- Clé SSH : `scp -P port user@host:fichier .` · `chmod 600 cle` · `ssh -i cle user@host -p port`
- `nc host port` : parler en clair à un port TCP · `openssl s_client -connect host:port -quiet` : en TLS
- Certificat auto-signé = chiffré mais pas authentifié → risque d'homme du milieu
- `nmap -sV -p31000-32000 host` : plage de ports + détection des services
- `diff a b` : différences entre deux fichiers
- `ssh user@host "commande"` : exécuter une commande sans ouvrir de session (contourne un `.bashrc` piégé)
- `.bashrc` modifié = technique de persistance d'attaquant
- Binaire **setuid** (`s` dans `ls -l`) : s'exécute avec les droits du propriétaire → élévation de privilèges ; recherche : `find / -perm -4000 2>/dev/null`
