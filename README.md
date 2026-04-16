# Lab-Active-Directory

<img width="950" height="621" alt="image" src="https://github.com/user-attachments/assets/90eb6343-5355-43b8-847b-0556a88106f1" />

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


## 📁 Loot

- [`loot/domain_users.html`](loot/domain_users.html) — Export ldapdomaindump

## ⚠️ Disclaimer

Cet environnement est **strictement éducatif**. Toutes les attaques
sont réalisées sur une infrastructure personnelle, isolée, sans accès
réseau extérieur. Ne jamais reproduire ces techniques sur des systèmes
sans autorisation explicite.
