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



