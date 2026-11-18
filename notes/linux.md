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
