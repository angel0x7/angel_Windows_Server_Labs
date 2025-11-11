# TP1 – Déploiement d’un Domaine Active Directory

##  Projet
**Sujet :** 2025-26_ECE_PA_ING4_WINSRV_TP1_FR.pdf  
**Livrable :** TP1 - GP01 - Velasco - Marchal - Sourdin - ADDA.pdf  

---

##  Objectif
Déployer et administrer un **domaine Active Directory (AD DS + DNS)** sous Windows Server.  
Compétences visées :
- Installation du contrôleur de domaine  
- Gestion des OU, utilisateurs, groupes  
- Application de GPO de sécurité  
- Configuration de partages réseau avec droits NTFS  
- Vérification et diagnostic via outils systèmes

---

##  Étapes principales
1. **Installation du DC** : création de la forêt `gp01.lab.ece`, test DNS et `dcdiag`  
2. **Organisation de l’annuaire** : OU, comptes et groupes (`Grp-GP01-*`)  
3. **Intégration client** : jonction au domaine, `gpresult /R`, journaux 4624/4625  
4. **Déploiement GPO** : sécurité, restrictions, fond d’écran, blocage `.exe`  
5. **Partage SMB/NTFS** : `Compta$`, gestion des droits et mapping H:

---

##  Travail individuel
- 1 question parmi 4 (sans doublon)  
- Conclusion personnelle (10 à 15 lignes)

---

##  Livrables
| Type | Contenu | Format |
|------|----------|---------|
| Groupe | Captures, réponses, tests | PDF unique |
| Individuel | Question + conclusion | PDF par étudiant |

---

##  Commandes utiles
`dcdiag /v` • `nslookup` • `gpresult /R` • `gpupdate /force` • `whoami /groups`

---

##  Synthèse
Mise en œuvre complète d’un domaine **Active Directory** : installation, sécurité, automatisation et validation fonctionnelle.
