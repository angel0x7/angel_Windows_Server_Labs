# RAPPORT DÉTAILLÉ : DÉPLOIEMENT ET ADMINISTRATION D'UN DOMAINE ACTIVE DIRECTORY (AD)

Ce document fournit une explication complète des procédures, configurations et objectifs atteints lors des travaux pratiques (TP1 et TP2) portant sur l'établissement et l'administration avancée d'un domaine **Active Directory (AD)**.

**Domaine Établi :** `gp01.lab.ece`

---

## I. ARCHITECTURE ET DÉPLOIEMENT INITIAL (TP1)

### 1. Installation et Configuration du Contrôleur de Domaine Principal

| Serveur | Rôle(s) | Nom NetBIOS | Adresse IP Statique | Configuration DNS |
| :--- | :--- | :--- | :--- | :--- |
| **SRV-DC-GP01** | DC Principal / DNS | SRV-DC-GP01 | **192.168.10.10** | Pointeur vers lui-même (`192.168.10.10`) |

* **Préparation du serveur :** L'opération initiale a consisté à renommer le serveur avec un nom conforme à la nomenclature du groupe, puis à lui attribuer une adresse IP statique.
* **Promotion en DC :** Les rôles **AD DS (Active Directory Domain Services)** et **DNS** ont été installés pour créer une nouvelle forêt (`gp01.lab.ece`).

### 2. Validation de l'Installation du DC

L'outil de diagnostic **`DCDiag`** a été utilisé pour confirmer le bon état et le rôle du contrôleur de domaine, en se concentrant sur les tests critiques :

* **Connectivity :** Assure la joignabilité et la réponse des protocoles réseau.
* **Advertising :** Confirme que le serveur s'annonce comme **Global Catalog (GC)**, **LDAP** et **Kerberos**, ce qui est essentiel pour la découverte des services par les clients.
* **Services :** Vérifie que les services AD critiques (tels que Netlogon et le service de distribution Kerberos) sont démarrés et opérationnels.
* **DNS :** Valide l'existence de la zone de recherche directe et la présence des enregistrements **SRV** nécessaires à la localisation du contrôleur de domaine par les clients.

---

## II. ORGANISATION ET SÉCURITÉ PAR STRATÉGIES DE GROUPE (GPO) (TP1)

### 1. Structure de l'Annuaire

L'annuaire a été organisé via des **Unités d'Organisation (OU)** pour permettre une application ciblée des GPO :

* **`Utilisateurs-GP01`** : Contient tous les comptes utilisateurs standards.
* **`Ordinateurs-GP01`** : Contient les objets ordinateurs clients, permettant d'y lier les GPO de poste de travail.

Deux groupes de sécurité globaux ont été créés pour faciliter la gestion des permissions et des droits : **`Grp-GP01-Admins`** et **`Grp-GP01-Utilisateurs`**.

### 2. Politiques de Sécurité (GPO)

#### A. Politique de Domaine (Default Domain Policy)
Les exigences de sécurité des mots de passe ont été renforcées :

| Paramètre | Valeur Configurée |
| :--- | :--- |
| **Longueur minimale** | **10 caractères** |
| **Complexité** | Activée |
| **Historique** | Mémorisation des **5** derniers mots de passe |
| **Verrouillage de compte** | Seuil de **5** tentatives échouées pendant **15 minutes** |

#### B. Restrictions Client (GPO-Securite-GP01)
Cette GPO a été liée à l'OU `Ordinateurs-GP01` pour sécuriser et personnaliser l'environnement utilisateur :

* **Interface utilisateur :** Interdiction de l'accès au **Panneau de configuration**.
* **Message de connexion :** Affichage d'un message d'accueil personnalisé.
* **Fond d'écran :** Définition d'un fond d'écran commun via un chemin d'accès réseau (partage).
* **Software Restriction Policies (SRP) :** Mise en place d'une règle pour bloquer l'exécution des fichiers **`.exe`** dans le dossier **`Téléchargements`**, empêchant l'exécution de logiciels malveillants téléchargés.

---

## III. GESTION DES RESSOURCES PARTAGÉES SÉCURISÉES (TP1)

### 1. Application du Modèle AGDLP

Le partage de fichiers pour le service `Compta` a été sécurisé en utilisant le principe du moindre privilège via le modèle **AGDLP** (Accounts, Global Group, Domain Local Group, Permission) :

1.  **A (Accounts) :** Création des comptes utilisateurs individuels.
2.  **G (Global Group) :** Création du groupe global **`GG-Compta-Users-GP01`** (Contient les utilisateurs du service).
3.  **DL (Domain Local Group) :** Création du groupe local de domaine **`DL-Compta-RW-GP01`** (Contient le groupe Global `GG-Compta-Users-GP01`).
4.  **P (Permissions) :** Les droits NTFS sont appliqués **uniquement** au groupe `DL-Compta-RW-GP01`.

### 2. Partage et Droits Effectifs

* **Ressource :** Création du dossier `C:\Data\Compta` partagé sous le nom **`Compta$`** sur le DC.
* **Droit NTFS :** Le groupe `DL-Compta-RW-GP01` reçoit l'autorisation **`Modify`**.
* **Mapping par GPO :** Une GPO a été créée pour monter automatiquement le partage réseau `\\SRV-DC-GP01\Compta$` en lecteur **`H:`** pour les utilisateurs concernés.

**Règle de Priorité :** Lors d'un accès réseau, l'autorisation effective est toujours déterminée par la permission la plus **restrictive** entre les droits **NTFS** (système de fichiers local) et les droits de **partage** (accès réseau).

---

## IV. CONTINUITÉ DE SERVICE ET RÉSILIENCE (TP2)

### 1. Déploiement du Contrôleur de Domaine Secondaire

Afin d'assurer la **redondance** et la continuité de service (haute disponibilité), un second serveur, **SRV-DC2-GP01**, a été déployé :

* La machine virtuelle a été jointe au domaine existant.
* Elle a ensuite été promue en tant que **Contrôleur de Domaine Secondaire**, intégrant le conteneur `Domain Controllers` dans la console **Active Directory Users and Computers**.

### 2. Vérification de la Réplication et Test de Bascule

* **Réplication :** Après la promotion, des vérifications confirment le bon fonctionnement de la réplication des objets (utilisateurs, groupes, GPO) entre le DC principal et le DC secondaire.
* **Test de Résilience :** Le test final a consisté à simuler une panne en mettant hors ligne le contrôleur de domaine principal (`SRV-DC-GP01`). L'objectif était de vérifier que le client pouvait toujours s'authentifier correctement auprès du DC secondaire, assurant ainsi la **continuité du service d'authentification**.

---

## V. BILAN ET PERSPECTIVES

### 1. Problèmes Rencontrés

* Un problème a été rencontré lors de l'intégration du poste client (Windows 11) au domaine `gp01.lab.ece`.
* **Conséquence :** L'impossibilité de joindre le domaine a empêché la validation opérationnelle des configurations côté client. Il n'a pas été possible d'exécuter la commande **`gpresult /R`** pour générer le rapport prouvant que les GPO (notamment le montage du lecteur **H:** et les restrictions SRP) avaient été correctement appliquées à l'utilisateur et à l'ordinateur.

### 2. Leçons Stratégiques et Perspectives d'Évolution

* **Importance de l'AGDLP :** Le modèle est fondamental, car il permet une maintenance simplifiée de la sécurité à long terme : une modification des droits sur la ressource ne nécessite qu'un changement au niveau du groupe Local de Domaine (`DL-Compta-RW-GP01`).
* **Résilience à 3 ans :** Pour garantir la résilience et la maintenabilité, une stratégie sur le long terme inclut l'ajout de plusieurs DC et DNS répliqués, un durcissement continu du domaine et une surveillance active via des outils **SIEM (Security Information and Event Management)**.
* **Automatisation :** L'adoption d'une stratégie d'automatisation est essentielle pour la cohérence et la réduction des erreurs.
    * **Outils professionnels :** Des solutions comme **Microsoft Intune**, **SCCM/MECM** pour la gestion de parc, et des outils **Infrastructure as Code (IaC)** tels qu'**Ansible** ou **Terraform** sont recommandés pour automatiser les configurations et la gestion d'un environnement AD moderne.
