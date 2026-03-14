# Spécification de Projet : "GrabberBoxTea"
**Solution d'Infrastructure "Clé en Main" pour TPE**

## Contexte et Objectif
Pour une TPE qui a des besoins sysops spécifiques mais voulant garder un controle total de leurs données sans avoir à dépendre d'outils externes (ex. Google, AWS)

L'entreprise a des besoins spécifiques tels que :
- Accéder aux données de l'entreprise partout à l'intérieur comme à l'extérieur de l'entreprise
- Donner des accès spécifiques à des monteurs freelances, dans le cadre d'une amélioration de la communication
- Pouvoir manager le serveur via une interface simple et accessible (Pas de DSI, gestion par les employés, niveau confirmé)
- BONUS: Avoir des outils de travaux collaboratifs intégrés au cloud pour éditer des données partout (ex. Comptabilité,...)

## Architecture Technique
Le système repose sur une architecture en couches distinctes, exploitant des langages spécifiques pour chaque niveau de responsabilité :

* **Linux (Base Système) :** Système d'exploitation hôte (ex: Debian ou Alpine). Gestion avancée du partitionnement (LVM), durcissement de la sécurité (Hardening) et gestion des droits utilisateurs.
* **Bash (Provisioning & Automatisation) :** Scripts d'installation "Zero Touch". Responsable du déploiement initial, de la configuration réseau et de l'installation des dépendances.
* **C (Daemon de Monitoring) :** Développement d'un service systemd léger. Il interagit directement avec le noyau pour surveiller l'intégrité matérielle (SMART, température) et système (intégrité des fichiers critiques) avec une empreinte ressource minimale.
* **Python (Backend & Orchestration) :** API REST (Flask/FastAPI). Elle agit comme passerelle entre l'interface utilisateur et les commandes système, gérant la logique métier et l'exécution sécurisée des tâches administratives.
* **JavaScript (Frontend) :** Interface d'administration web (Dashboard). Permet à l'utilisateur final de piloter le serveur graphiquement.

## Fonctionnalités Clés

### A. Gestion du Stockage (Module Python/Bash)
* Configuration dynamique du serveur de fichiers (Samba/NFS) via l'interface web.
* Gestion des utilisateurs et attribution de quotas de stockage.
* Modification automatisée des fichiers de configuration (`/etc/samba/smb.conf`) et rechargement des services à la volée.

### B. Sauvegarde Automatisée sur Événement (Module C/Bash)
* Système de détection matériel : le daemon en **C** écoute les événements `udev` pour détecter la connexion de périphériques USB spécifiques.
* Déclenchement automatique d'un script de synchronisation (**Bash/Rsync**) lors du montage du disque.
* Remontée du statut de la sauvegarde vers l'API.

### C. Tableau de Bord de Santé (Module C/JS)
* Visualisation en temps réel des métriques vitales (CPU, RAM, I/O Disque).
* Lecture optimisée des pseudo-systèmes de fichiers `/proc` et `/sys` par le module C pour une latence minimale.

## 4. Dimension DevOps et Administration Système
Le projet intègre les meilleures pratiques d'ingénierie système :

1.  **Packaging et Déploiement :** Livraison sous forme de script d'installation global ou d'image système pré-configurée (ISO). Utilisation possible d'Ansible pour la configuration initiale.
2.  **Sécurité (SecOps) :** Configuration de pare-feu (`nftables`), isolation des processus, et mise en place de HTTPS automatisé.
3.  **Conteneurisation :** Isolation possible des briques applicatives (Web/API) via Docker, tout en maintenant les accès privilégiés nécessaires à l'administration de l'hôte.

## 5. Flux Opérationnel (Workflow)

1.  **Installation :** L'administrateur exécute le script Bash d'initialisation sur un serveur vierge. Le système est opérationnel en quelques minutes.
2.  **Configuration :** L'utilisateur se connecte au Dashboard Web local.
3.  **Action :** Une demande (ex: "Créer un partage 'Compta'") est envoyée par le Frontend (JS).
4.  **Exécution :** L'API (Python) valide la requête et orchestre les commandes système nécessaires.
5.  **Utilisation :** Les postes clients (Windows/Mac) détectent immédiatement le nouveau volume réseau sécurisé.

## 6. Récapitulatif de la Stack Technologique

| Composant | Technologie | Rôle Principal |
| :--- | :--- | :--- |
| **OS** | Debian 12 / Alpine | Base stable et sécurisée. |
| **Infrastructure** | Bash + Ansible | Provisioning et maintenance. |
| **Backend API** | Python (FastAPI) | Logique métier et interface système. |
| **Système** | **Langage C** | Monitoring bas niveau et gestion I/O. |
| **Frontend** | Vue.js / React | Interface Homme-Machine (IHM). |
| **Protocoles** | Samba (SMB) / NFS | Partage de fichiers hétérogène. |