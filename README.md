# Lab-Active-Directory
<img width="3235" height="2273" alt="infra_lab_ad" src="https://github.com/user-attachments/assets/96405625-fbd6-4640-8e74-7b5aab2fda06" />

Simulation d'attaque complète sur une infrastructure Active Directory 
vulnérable, de la capture de hash NTLMv2 jusqu'au Kerberoasting. 
Lab éducatif réalisé avec Kali Linux, Impacket et Hashcat. Vous retrouverz le schéma de l'infrastructure dans le fichier infra_lab_ad_excalidraw.png. 

Infrastructure

| Machine | OS                  | IP             |  Rôle                                | 
|---------|---------------------|----------------|--------------------------------------|
| DC01    | Windows Server 2022 | 192.168.254.10 | Domain Controller (AD DS, DNS, DHCP) |
| PC01    | Windows 10 Pro      | 192.168.254.11 | Workstation jointe au domaine        |
| Kali    | Kali Linux          | 192.168.254.13 | Attaquant                            |

**Domaine :** `LAB-DOMAIN.local`
**Réseau :** VMnet1 Host-Only — `192.168.254.0/24`

## ⚠️ Vulnérabilités Configurées

- SMBv1 activé sur PC01
- Pare-feu désactivé sur DC01 et PC01
- Mots de passe faibles
- LLMNR et NBT-NS actifs
- SMB Signing désactivé
- Compte `johnd` : AS-REP Roastable (no preauth)
- Compte `svcbackup` : Kerberoastable (SPN configuré)

## 🎯 Attaques Documentées

| # | Attaque | Outil | Résultat |
|---|---|---|---|
| 01 | LLMNR Poisoning | Responder | Hash NTLMv2 capturé |
| 02 | Hash Cracking | Hashcat | `abcd1234*` |
| 03 | SMB/LDAP Enumeration | NetExec | Users, groupes, partages |
| 04 | LDAP Dump | ldapdomaindump | Cartographie AD complète |
| 05 | AS-REP Roasting | Impacket GetNPUsers | Hash TGT de johnd |
| 06 | Kerberoasting | Impacket GetUserSPNs | Hash ST de svcbackup |
| 07 | AD Mapping | BloodHound CE | Chemins d'attaque vers DA |
| 08 | Pass-the-Hash | NetExec / Evil-WinRM | Mouvement latéral |
| 09 | DCSync | impacket-secretsdump | Dump complet des hashes |

## 📁 Loot

- [`loot/domain_users.html`](loot/domain_users.html) — Export ldapdomaindump

## ⚠️ Disclaimer

Cet environnement est **strictement éducatif**. Toutes les attaques
sont réalisées sur une infrastructure personnelle, isolée, sans accès
réseau extérieur. Ne jamais reproduire ces techniques sur des systèmes
sans autorisation explicite.
