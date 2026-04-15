<img width="763" height="1684" alt="image" src="https://github.com/user-attachments/assets/c86a10bc-d6d2-4ff6-9927-06892aac8a44" />
# 01 - LLMNR Poisoning (Responder)

## Définition
LLMNR (Link-Local Multicast Name Resolution)
est un protocole Windows
qui permet à une machine de résoudre un nom d'hôte sur le réseau local
lorsque le DNS échoue. Quand un utilisateur tape un chemin réseau inexistant
(ex : \\SERVEUR-INEXISTANT), Windows envoie une requête LLMNR en broadcast
à tout le sous-réseau. Un attaquant qui écoute peut répondre à cette requête
en se faisant passer pour le serveur demandé, forçant la victime à lui envoyer
ses identifiants (hash NTLMv2) pour s'authentifier. C'est une attaque de type
Man-in-the-Middle passive, exploitable sans aucun accès préalable au domaine.

## Prérequis
- Être positionné sur le même segment réseau que la cible (VMnet1 — 192.168.254.0/24)
responder installé sur Kali (inclus par défaut)
- LLMNR et NBT-NS activés sur les machines Windows cibles (non patchées)
- Pare-feu désactivé sur PC01 et DC01 (configuré intentionnellement)
NBT-NS (NetBIOS Name Service) fonctionne sur le même principe (port UDP 137)
et constitue un vecteur d'attaque identique. Responder empoisonne les deux
simultanément.

## Objectif
Capturer un hash NTLMv2 de l'utilisateur johnd en empoisonnant les
requêtes LLMNR/NBT-NS sur le réseau local, afin de le transmettre à Hashcat
pour retrouver le mot de passe en clair.

## Etape 1:

Un utilisateur est connecté à PC01:
<img width="945" height="504" alt="image" src="https://github.com/user-attachments/assets/dd90912e-dd4f-4c95-b8cd-f9a070442095" />

On lance l’outils responder sur notre machine Kali linux :
<img width="728" height="456" alt="image" src="https://github.com/user-attachments/assets/e9d843f9-2c09-470a-b905-22b24d182dd5" />

Option	Signification	Rôle

-I eth0	Interface réseau	Dit à l’outil sur quelle carte réseau écouter, ici eth0.
-w	WPAD	Lance un faux serveur WPAD pour attirer les clients Windows qui cherchent un proxy automatique.
WPAD permet à Internet Explorer de détecter automatiquement un serveur capable de lui fournir les paramètres de configuration d'un serveur proxy. WPAD permet à Internet Explorer de détecter automatiquement un serveur capable de lui fournir les paramètres de configuration d'un serveur proxy
-v	Verbose	Affiche plus de détails dans la console pour voir ce que l’outil fait.
-F	Force WPAD Auth	Force l’utilisateur à s’authentifier quand il récupère wpad.dat, pour tenter de faire apparaître des identifiants NTLM.

Ce que fait Responder, étape par étape
1. Il écoute le réseau
Dès qu'il est lancé, Responder met en place une série de faux serveurs (SMB, HTTP, LDAP, FTP...) sur la machine Kali et surveille tout ce qui passe sur le réseau local.
2. Il attend une erreur de résolution
Quand une machine Windows (PC01) cherche un nom qui n'existe pas (ex: \\SERVEUR-INCONNU), elle ne trouve pas de réponse DNS. Elle passe alors à un mécanisme de secours appelé LLMNR ou NBT-NS, qui diffuse la question à tout le réseau : "quelqu'un connaît ce nom ?"
3. Il répond à la place du vrai serveur
Responder intercepte cette diffusion et répond "oui, c'est moi !" il se fait passer pour le serveur recherché.
4. Windows envoie ses identifiants
PC01, croyant parler à un serveur légitime, tente de s'y connecter automatiquement. Pour s'authentifier, Windows envoie un hash NTLMv2 une version chiffrée du mot de passe de l'utilisateur connecté.
5. Responder capture le hash
Responder récupère ce hash et l'affiche dans le terminal. Il le sauvegarde aussi dans un fichier log automatiquement.
6. Il ne casse pas le mot de passe lui-même
Responder s'arrête là il capture, il n'analyse pas. Le cassage du mot de passe à partir du hash se fait ensuite avec Hashcat.

Ainsi l’outils responder, attend une erreur de résolution de domaine, afin de capturer le hash : 
<img width="945" height="230" alt="image" src="https://github.com/user-attachments/assets/133e13e1-6fbb-4abc-9389-ef7dca2219e9" />

Dans l’explorateur de fichier, on rentre dans la barre de recherche : \\SERVEUR-INEXISTANT.

<img width="945" height="533" alt="image" src="https://github.com/user-attachments/assets/526b0a76-ef08-4e0c-9bb8-e781d02a56bc" />

Puis, on rentre nos identifiants de connexion.
<img width="945" height="520" alt="image" src="https://github.com/user-attachments/assets/45791603-3cce-4613-ae8f-5e01964bd70f" />

L’accès est refusé.
<img width="945" height="216" alt="image" src="https://github.com/user-attachments/assets/8f03520e-a743-407e-8ff4-9b8b89f649d8" />

Nous obtenons ainsi la requête du client. Nous ainsi les informations d’authentification du client, ainsi que le hash de son mot de passe : 
<img width="945" height="277" alt="image" src="https://github.com/user-attachments/assets/0c9d11ab-3de2-4d7c-b962-baade49c0705" />

On accède aux logs de Responder :

<img width="945" height="161" alt="image" src="https://github.com/user-attachments/assets/904fdea3-441f-49d0-a67a-e59c282e630a" />

Nous avons le hash du user stocker :
<img width="945" height="650" alt="image" src="https://github.com/user-attachments/assets/7a218a43-55d2-44f8-871e-2f9aa2182dcd" />

On utilise hashcat, pour pouvoir lire le hash.
Puis, on copie colle le hash, afin de pouvoir le rentrer dans hashcat :
<img width="945" height="151" alt="image" src="https://github.com/user-attachments/assets/87619ff8-ee34-46e2-9b59-b470502202e7" />
<img width="945" height="426" alt="image" src="https://github.com/user-attachments/assets/a24f6fe6-4a56-4f0e-903c-151044ef5056" />

Pourquoi /tmp ?
C'est rapide et accessible
/tmp est le dossier standard sous Linux pour stocker des fichiers de travail temporaires. Pas besoin de droits spéciaux pour y écrire, c'est accessible immédiatement.
C'est temporaire par nature
Les fichiers dans /tmp sont automatiquement supprimés au redémarrage de Kali. C'est parfait pour un lab : le hash ne traîne pas indéfiniment sur la machine.
C'est une convention pentest
Dans les tutoriels et labs de sécurité offensifs, /tmp est le dossier de facto pour stocker des fichiers de travail rapides (hashes, résultats de scans...) sans polluer le système.
•	NTLM utilise un système de défi-réponse pour vérifier l'identité de l'utilisateur chaque fois qu'ils accèdent à une ressource réseau. Kerberos, en revanche, fonctionne avec des billets. Après s'être connecté une fois, les utilisateurs reçoivent un Ticket Granting Ticket
NTLMv2 est la deuxième version du protocole d'authentification NTLM de Microsoft, utilisé dans les environnements Windows et Active Directory.
________________________________________
Ce que fait NTLM
NTLM sert à prouver ton identité sur un réseau Windows sans envoyer le mot de passe en clair. Il fonctionne sur un système de défi/réponse :
1.	Le client veut accéder à une ressource
2.	Le serveur envoie un défi (un nombre aléatoire)
3.	Le client chiffre ce défi avec le hash de son mot de passe et renvoie une réponse
4.	Le serveur vérifie la réponse → accès accordé ou refusé

NTLMv1 vs NTLMv2
	NTLMv1	NTLMv2
	NTLMv1	NTLMv2
Année	1993	1998
Chiffrement	DES (56 bits, faible)	HMAC-MD5 (128 bits)
Défi client	❌ Non	✅ Oui
Horodatage	❌ Non	✅ Oui
Résistance aux attaques	Très faible	Meilleure, mais pas totale
NTLMv2 ajoute un défi côté client + un horodatage, ce qui rend les attaques par rejeu plus difficiles.




📚 Sources consultées
Ressource	Lien
Responder — GitHub officiel (SpiderLabs)	https://github.com/SpiderLabs/Responder
Hashcat — Site officiel	https://hashcat.net/hashcat/
