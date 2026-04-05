# TP3-GESTION-INTRUSION

# Rapport de Test d'Intrusion (Pentesting) - TP3

## 1. Introduction
Ce projet présente une démarche de test d'intrusion en mode **Boîte Noire** (Black Box) sur un serveur cible. L'objectif est d'identifier les vulnérabilités système et réseau, de les exploiter pour tester la robustesse du serveur, et de proposer des mesures correctives.

### Environnement de travail :
* **Machine Attaquante :** Kali Linux (IP: `10.0.2.15`)
* **Machine Cible :** Serveur de l'Entreprise X (IP: `10.0.2.3`)
* **Outils utilisés :** Nmap, Metasploit, DirBuster, Searchsploit.

---

## 2. Phase de Reconnaissance (Prise d'empreinte)
La première étape consiste à cartographier le serveur cible pour découvrir les services actifs et leurs versions.

### Commande exécutée :
```bash
nmap -sS -sV -O 10.0.2.3
```
![Import OVA](https://github.com/user-attachments/assets/dd037ba0-9a6c-4e39-a1d1-d2ecd4e4dd6a)

---

## 3. Analyse des Vulnérabilités (Analyse de vsftpd 2.3.4)
Après le scan de ports, nous avons identifié que le service **FTP** utilise la version **vsftpd 2.3.4**. 

### Recherche de faille (CVE) :
En utilisant l'outil `searchsploit` ou en consultant des bases de données comme **Exploit-DB**, nous avons découvert que cette version spécifique contient une **Backdoor** (Porte dérobée).

* **CVE associée :** CVE-2011-2523
* **Description :** Cette version de vsftpd a été compromise à la source. Si un utilisateur se connecte avec un identifiant finissant par `:)`, le service ouvre un shell (accès commande) sur le port 6200 avec les privilèges **root**.

### Pourquoi cette étape est importante ?
* **Compréhension :** On ne lance pas des attaques au hasard. On identifie d'abord une faille précise liée à une version spécifique.
* **Ciblage :** Cela nous permet de choisir le bon module dans Metasploit pour réussir l'intrusion.

 **Note :** L'accès "root" est le niveau de privilège le plus élevé. Prendre le contrôle en tant que root signifie avoir le contrôle total du serveur.

![Import OVA](https://github.com/user-attachments/assets/d8e1aed2-6b73-4bd4-930d-d889a7e3b3e6)

---

## 4. Phase d'Exploitation (Accès Root)
Après avoir configuré l'exploit `vsftpd_234_backdoor`, nous avons lancé l'attaque contre la cible.

### Exécution des commandes :
* **Commande :** `exploit`
* **Résultat :** Ouverture d'une session shell (`Command shell session 1 opened`).

### Vérification des privilèges :
Une fois connecté au serveur, nous avons vérifié notre identité avec la commande `whoami`.
* **Résultat obtenu :** `root`
* **Signification :** Le système nous reconnaît comme l'administrateur suprême du serveur. À ce stade, le serveur est totalement compromis.

### Pourquoi est-ce une preuve de réussite ?
Le fait d'obtenir `uid=0(root)` prouve que la faille de la version 2.3.4 de vsftpd permet à n'importe quel attaquant de prendre le contrôle total du serveur sans même connaître de mot de passe valide.

![Import OVA](https://github.com/user-attachments/assets/4b2c7f19-3896-4bd2-8d45-571be66d0e06)

---

## 5. Énumération Web (Discovery)
Une fois l'accès système compromis, nous avons effectué une recherche de répertoires cachés sur le serveur Web pour identifier des vecteurs d'attaque supplémentaires.

### Outil utilisé : DIRB
DIRB est un scanner de contenu web qui utilise une attaque par dictionnaire pour trouver des répertoires et fichiers non indexés.

### Résultats obtenus :
L'analyse a révélé plusieurs répertoires sensibles :
* **`/phpmyadmin/`** : Interface de gestion de base de données. C'est une découverte critique car elle permet potentiellement d'accéder aux données confidentielles (utilisateurs, commandes, etc.).
* **`/server-status`** : Fournit des informations sur l'état et la configuration du serveur Apache.
* **`/javascript/`** : Répertoire contenant des scripts pouvant être analysés pour trouver d'autres failles côté client.

### Pourquoi cette étape est-elle importante ?
Même avec un accès root, l'énumération web permet de comprendre l'architecture de l'application hébergée et de localiser rapidement les données de valeur (Crown Jewels) de l'entreprise.

![Import OVA](https://github.com/user-attachments/assets/1b02f321-a8d4-483b-aadf-097663db10f9)

---

---

## 6. Plan d'Actions et Recommandations
Suite à notre audit, nous préconisons les mesures suivantes pour sécuriser l'infrastructure :

### A. Mesures Immédiates (Critiques)
1. **Mise à jour du service FTP :** Désinstaller `vsftpd 2.3.4` et installer la version la plus récente (ex: 3.0.5) pour supprimer la porte dérobée.
2. **Désactivation des services non sécurisés :** Arrêter le service **Telnet** (Port 23) et utiliser exclusivement **SSH** pour l'administration à distance.

### B. Sécurisation Applicative
1. **Protection de phpMyAdmin :** Masquer l'URL par défaut et ajouter une authentification au niveau du serveur web (type `.htaccess`).
2. **Mise en place de HTTPS :** Installer un certificat SSL/TLS et forcer la redirection du trafic HTTP (port 80) vers HTTPS (port 443).

## Synthèse Finale
Ce TP nous a permis de simuler un cycle complet d'intrusion : de la reconnaissance à l'exploitation. La vulnérabilité critique découverte dans le service FTP montre l'importance capitale de maintenir ses systèmes à jour. Sans une politique de mise à jour rigoureuse, un serveur peut être entièrement compromis en quelques minutes.

