# Homelab - Laboratoire de pentest sur MacBook Pro M1

Laboratoire isolé pour m'entraîner aux tests d'intrusion et à la défense, avec une machine d'attaque et une cible volontairement vulnérable.

## Matériel et logiciels
| Élément | Détail |
| --- | --- |
| Hôte | MacBook Pro M1 Pro (Apple Silicon, ARM) |
| Hyperviseur | UTM (QEMU) |
| Attaquant | Kali Linux 2026.3 ARM64 (virtualisée, 4 Go RAM, 4 CPU) |
| Cible | Metasploitable 2 (Ubuntu 8.04 32 bits, émulée en x86, 1 Go RAM) |

## Architecture réseau
```mermaid
flowchart LR
    Mac[MacBook M1 - hôte] --- NAT[Réseau partagé UTM - 192.168.64.0/24 - DHCP + NAT]
    NAT --- Kali[Kali Linux - 192.168.64.X]
    NAT --- Meta[Metasploitable 2 - 192.168.64.4]
```

## Choix de sécurité
- **Mode « Réseau partagé » (NAT) uniquement, jamais « Bridged »** : Metasploitable est remplie de failles volontaires. En mode Bridged, elle serait visible par tous les appareils du réseau de la maison. En NAT, elle reste isolée derrière le Mac.
- Les deux machines sont dans le même sous-réseau /24 (254 hôtes possibles) et communiquent directement.

## Problèmes rencontrés et solutions
| Problème | Cause | Solution |
| --- | --- | --- |
| Écran noir « Display output is not active » à l'installation de Kali | L'installateur ARM n'affiche rien sur la carte graphique par défaut | Carte `virtio-gpu-pci` + ajout d'un **terminal série** |
| Kali redémarre sur l'installateur | ISO toujours branchée | Retrait de l'ISO dans les lecteurs |
| Metasploitable sans adresse IP (`ifconfig` : seulement `lo`) | Carte réseau émulée `ne2k_isa` non détectée par ce vieux Linux | Passage à la carte **Intel e1000** |
| Disque Metasploitable au format VMware | UTM utilise le format qcow2 | Conversion avec `qemu-img convert -f vmdk -O qcow2` |

## Vérifications
- `ping -c 4 192.168.64.4` depuis Kali : 4/4 réponses, **TTL 64** (cible Linux, aucun routeur entre les deux).

## Premier scan : `nmap 192.168.64.4`
23 ports TCP ouverts, dont :

| Port | Service | Risque |
| --- | --- | --- |
| 21 | FTP | Identifiants en clair |
| 23 | Telnet | Tout passe en clair (remplacé par SSH) |
| 139 / 445 | SMB | Cible privilégiée des rançongiciels (WannaCry) |
| 512-514 | rexec / rlogin / rsh | Accès à distance sans vraie authentification |
| 1524 | ingreslock | Backdoor (accès root) |
| 3306 / 5432 | MySQL / PostgreSQL | Bases de données exposées sur le réseau |
| 5900 / 6000 | VNC / X11 | Contrôle d'écran à distance |

**Conclusion défensive** : chaque port ouvert agrandit la surface d'attaque. Il faut fermer les services inutiles et remplacer les protocoles en clair (Telnet → SSH, FTP → SFTP).

##Captures d'écran tirées de la mise en place du homelab: 
<img width="586" height="173" alt="09-ping-kali-metasploitable" src="https://github.com/user-attachments/assets/94e2bc72-aaf9-41d2-8e15-792b64a4dc68" />
<img width="750" height="375" alt="08-metasploitable-ifconfig" src="https://github.com/user-attachments/assets/fee34c84-2f9e-4e5f-a719-6f27d5296701" />
<img width="338" height="759" alt="07-utm-cartes-reseau" src="https://github.com/user-attachments/assets/374a14b4-50e9-405f-8679-2c9ba212a95c" />
<img width="633" height="202" alt="06-metasploitable-sans-ip" src="https://github.com/user-attachments/assets/19f172c5-f202-4a29-abe4-cc36118d800a" />
<img width="1376" height="933" alt="05-kali-bureau" src="https://github.com/user-attachments/assets/bd7ceaaf-757b-4212-8375-705f154bf47c" />
<img width="1250" height="835" alt="04-kali-installation-dhcp" src="https://github.com/user-attachments/assets/93770c51-4146-4024-ba4b-3cc0a6ef1712" />
<img width="785" height="653" alt="03-kali-terminal-serie" src="https://github.com/user-attachments/assets/2f0a1bb6-e8c7-4897-b534-136830ae41bf" />
<img width="734" height="506" alt="02-kali-menu-grub" src="https://github.com/user-attachments/assets/015c0159-2c79-4799-8ba8-90a84e423e9f" />
<img width="796" height="657" alt="01-kali-ecran-noir" src="https://github.com/user-attachments/assets/7c6e2d9c-b695-4bc5-9630-da10206d3d00" />




## Prochaines étapes
- [ ] Rapport de test d'intrusion complet de Metasploitable (énumération, exploitation, recommandations)
- [ ] Ajout d'un serveur Ubuntu avec SIEM Wazuh pour détecter les attaques
