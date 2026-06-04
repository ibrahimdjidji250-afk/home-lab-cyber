# 🖥️ Infrastructure Homelab - Cybersécurité & Réseau

Ce projet documente le déploiement pas à pas d'un serveur **Ubuntu 22.04.5 LTS (Jammy Jellyfish)** sur une machine physique (bare-metal). 

### 🎯 Objectifs du Projet
* **Réseau :** Maîtriser l'architecture réseau local, l'adressage statique, et le filtrage des flux.
* **Cybersécurité :** Mettre en pratique les concepts de durcissement système (*System Hardening*), de gestion des privilèges, de pare-feu, et d'analyse de logs pour se protéger contre les attaques courantes (force brute, scans de ports).
* **Supervision :** Suivre l'état de santé des ressources et monitorer les accès au système.


## 📌 Table des Matières
1. [Prérequis & Matériel](#-prérequis--matériel)
2. [Étape 1 : Préparation du Support d'Installation](#étape-1--préparation-du-support-dinstallation)
3. [Étape 2 : Installation du Système (PC Physique)](#étape-2--installation-du-système-pc-physique)
4. [Étape 3 : Configuration Réseau Post-Installation](#étape-3--configuration-réseau-post-installation)
5. [⏳ Feuille de Route (Roadmap & Sécurisation)](#-feuille-de-route-roadmap--sécurisation)
6. [📊 Commandes Utiles au Quotidien](#-commandes-utiles-au-quotidien)



## 🛠️ Prérequis & Matériel

### Matériel Requis :
* **Machine physique :** Un PC/Serveur avec processeur 64-bit, min. 2 Go de RAM (4 Go+ recommandés), et un stockage dédié (HDD/SSD).
* **Clé USB :** Capacité minimale de 8 Go.
* **Connexion Internet :** Un câble Ethernet (RJ45) branché est fortement recommandé pour l'installation initiale.

### Logiciels à Télécharger :
* Image ISO : [Ubuntu Server 22.04.5 LTS](https://releases.ubuntu.com/22.04/)
* Flashage de clé : [Rufus](https://rufus.ie/) (Windows) ou [BalenaEtcher](https://etcher.balena.io/) (Cross-platform).

---

## Étape 1 : Préparation du Support d'Installation

1. Insérez votre clé USB dans votre ordinateur principal.
2. Lancez votre logiciel de flashage (ex. Rufus).
3. Sélectionnez l'ISO d'Ubuntu Server 22.04.5 fraîchement téléchargée.
4. Conservez le schéma de partitionnement par défaut (**GPT** pour UEFI ou **MBR** selon l'âge de votre machine cible).
5. Cliquez sur **Démarrer** pour flasher l'ISO sur la clé USB.

---

## Étape 2 : Installation du Système (PC Physique)

1. Insérez la clé USB flashée dans le PC physique cible et démarrez-le.
2. Accédez au **Boot Menu** (généralement via les touches `F12`, `F11`, `F8` ou `Échap` au démarrage) et sélectionnez votre clé USB.
3. Suivez l'assistant graphique d'Ubuntu Server :
    * **Langue & Clavier :** Sélectionnez votre disposition (ex: *French*).
    * **Type d'installation :** Choisir *Ubuntu Server* (version standard).
    * **Réseau :** Laissez l'interface DHCP se configurer automatiquement (nous la fixerons à l'étape suivante).
    * **Stockage :** Sélectionnez *Use an entire disk*. Cochez la case **Set up this disk as an LVM group** (permet une gestion flexible des partitions plus tard).
    * **Profil :** Configurez votre nom d'utilisateur et un mot de passe fort.
    * **SSH Setup :** Cocher absolument **Install OpenSSH server** pour l'accès à distance.
    * **Featured Snaps :** Ne rien sélectionner pour le moment afin de garder un système minimal et léger.
4. Attendez la fin de l'installation et cliquez sur **Reboot Now**. Retirez la clé USB lorsque le système le demande.

---

## Étape 3 : Configuration Réseau Post-Installation

Sur Ubuntu Server 22.04, la configuration réseau s'effectue via **Netplan**. Pour attribuer une adresse IP statique à votre serveur afin qu'il soit toujours accessible à la même adresse :

1. Connectez-vous localement et identifiez votre interface réseau avec la commande :
```bash
   ip a
 2. Connexion SSH
```bash
ssh ibrahim@[mon @ip]
 

# Sécurisation SSH HomeLab - 03/06/2026

Procédure complète suite à détection d’intrusion. Fait hier 03/06/2026.

## 1. Contexte
Serveur compromis : clé SSH de l’attaquant trouvée dans `~/.ssh/`. 
Objectif : virer l’accès attaquant + durcir SSH + firewall.

## 2. Étapes réalisées sur le serveur Ubuntu

### A. Suppression des clés de l’attaquant
```bash
rm ~/.ssh/id_ed25519 ~/.ssh/id_ed25519.pub

![Capture suppression clés attaquant](captures/01_suppression_cles_attaquant.png)

B. Génération nouvelle clé depuis PC Windows
Sur PowerShell Windows, pas sur le serveur :

ssh-keygen -t ed25519 -f $env:USERPROFILE\.ssh\id_ed25519_homelab -C "ibrahim@homelab"

![Capture génération clé Windows](captures/02_ssh_keygen_windows.png)

Copie de la clé publique vers le serveur :

type $env:USERPROFILE\.ssh\id_ed25519_homelab.pub | ssh -p 2222 ibrahim@192.168.1.135 "cat >> ~/.ssh/authorized_keys"

C. Correction permissions - CRITIQUE POUR SSH

sudo chown ibrahim:ibrahim ~/.ssh
sudo chmod 700 ~/.ssh
sudo chown ibrahim:ibrahim ~/.ssh/authorized_keys  
sudo chmod 600 ~/.ssh/authorized_keys

![Capture correction permissions](captures/03_chmod_chown.png)

Explications :
- `700` = dossier .ssh : seul ibrahim peut lire/écrire/entrer
- `600` = authorized_keys : seul ibrahim peut lire/écrire

D. Vérification

ls -l ~/.ssh/authorized_keys

Résultat attendu : `-rw------- 1 ibrahim ibrahim ... authorized_keys`
![Capture vérification ls -l](captures/04_verification_permissions.png)

E. Test connexion sans mot de passe
Depuis Windows PowerShell :

ssh -p 2222 ibrahim@[mon @ip]

Si ça connecte direct = clé OK.
![Capture test SSH sans mot de passe](captures/05_test_ssh_ok.png)

3. Durcissement déjà appliqué hier

Outil	Statut	Commandes/Config
**Fail2Ban**	✅ Installé + Actif	`sudo apt install fail2ban`
Ban après 5 échecs sur port 2222
**UFW Firewall**	✅ Actif	`sudo ufw enable`
Seul port 2222 ouvert en SSH
**Port SSH**	✅ Changé	Port 22 → 2222 dans `/etc/ssh/sshd_config`

4. Difficultés rencontrées hier

Problème	Cause	Solution appliquée
`Permission denied` sur `authorized_keys`	Fichier appartenait à `root`	`sudo chown ibrahim:ibrahim ~/.ssh/authorized_keys`
Faute de frappe `authorised`	Anglais UK vs US	Corriger en `authorized`
`ls -l` bloqué	Dossier `.ssh` aussi root	`sudo chown ibrahim:ibrahim ~/.ssh`
Connexion demande mot de passe	Permissions mauvaises	Appliquer 700/600 strict

*Leçon clé :* SSH est parano. 1 mauvaise permission = il ignore la clé sans prévenir.

5. Prochaines étapes
1. Désactiver `PasswordAuthentication no` dans sshd_config une fois clé 100% stable
2. Redémarrer SSH : `sudo systemctl restart sshd`
3. Audit logs Fail2Ban : `sudo fail2ban-client status sshd`


---
Fait par : Ibrahim  
Date : 03/06/2026  
Serveur : homelabpc @ [mon @ip]  
Niveau : HomeLab sécurisé de base ✅


Là c’est complet. Fail2Ban + UFW sont dans section "Durcissement déjà appliqué" avec ✅




* Mon  YouTube avec la vidéo qui guide pas à  pas:https://www.youtube.com/playlist?list=PL59vNdUyu2fIf5OeoUcDf4bqLpzRBEwFs
* Mon LinkedIn pour se connecter à moi :https://www.linkedin.com/in/ibrahim-djidji-ba2ba3371/
