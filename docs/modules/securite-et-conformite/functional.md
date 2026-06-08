# Module : Sécurité et conformité

## 1. Rôle du module dans Comparia

Le module **Sécurité et conformité** garantit la confidentialité, l’intégrité, la traçabilité et la conformité réglementaire (notamment RGPD) de l’ensemble de la plateforme Comparia.

Il encadre :
- L’authentification et la gestion des utilisateurs,
- La gestion des rôles et des droits d’accès,
- La protection des données (chiffrement, isolation),
- La traçabilité via le journal d’audit,
- Les mécanismes liés à la conformité RGPD,
- Les exigences spécifiques SaaS, on-premise et marque blanche.

Ce module est transverse et s’applique à tous les autres modules (comparaison, OCR, rapports, collaboration, intégrations).

---

## 2. Gestion des utilisateurs

### 2.1 Création et gestion des comptes
- Création de comptes utilisateurs.
- Désactivation / suppression de comptes.
- Gestion des informations de profil (nom, email, langue FR/EN).
- Réinitialisation de mot de passe.

### 2.2 Authentification
- Authentification sécurisée via identifiant + mot de passe.
- Gestion de session sécurisée.
- Déconnexion manuelle.
- Expiration automatique des sessions après inactivité.

Règles de gestion :
- Les mots de passe doivent respecter une politique minimale de complexité.
- Un utilisateur désactivé ne peut plus accéder à la plateforme.
- Toute connexion doit être tracée dans le journal d’audit.

---

## 3. Gestion des rôles et des permissions

### 3.1 Modèle de rôles
Le système repose sur un modèle de contrôle d’accès basé sur les rôles (RBAC).

Exemples de rôles (adaptables selon tenant) :
- Administrateur
- Utilisateur standard
- Lecteur

### 3.2 Permissions
Les permissions couvrent notamment :
- Téléverser des documents
- Lancer une comparaison
- Consulter une comparaison
- Ajouter des commentaires
- Valider/refuser des différences
- Exporter des rapports
- Gérer les utilisateurs (administrateur)

### 3.3 Droits par document
- Les droits peuvent être restreints à certains documents ou comparaisons.
- Un utilisateur ne peut consulter que les documents pour lesquels il dispose d’un droit explicite.

Règles de gestion :
- Toute action est autorisée uniquement si le rôle et les permissions associées le permettent.
- Les vérifications d’accès sont obligatoires pour chaque action sensible.

---

## 4. Journal d’audit

### 4.1 Objectif
Garantir la traçabilité complète des actions critiques réalisées sur la plateforme.

### 4.2 Événements journalisés
- Connexions et tentatives de connexion.
- Téléversement de documents.
- Lancement d’une comparaison.
- Consultation d’une comparaison.
- Export de rapport.
- Ajout de commentaire.
- Validation/refus de différences.
- Suppression de documents.
- Modification des droits ou des utilisateurs.

### 4.3 Données enregistrées
Pour chaque événement :
- Identifiant utilisateur
- Date et heure
- Type d’action
- Identifiants des ressources concernées
- Résultat (succès / échec)

Règles de gestion :
- Les journaux ne sont pas modifiables par les utilisateurs standards.
- Leur consultation est réservée aux rôles autorisés.
- La conservation des logs respecte la politique définie par le client.

---

## 5. Chiffrement et protection des données

### 5.1 Données en transit
- Toutes les communications doivent être chiffrées.

### 5.2 Données au repos
- Les documents stockés doivent être chiffrés.
- Les résultats de comparaison et métadonnées sensibles doivent être protégés.

### 5.3 Isolation des tenants
En mode SaaS :
- Isolation logique des données entre clients.
- Aucune donnée d’un tenant ne doit être accessible à un autre.

---

## 6. Conformité RGPD

### 6.1 Principes appliqués
- Minimisation des données collectées.
- Limitation des finalités (comparaison documentaire uniquement).
- Durée de conservation configurable selon client.

### 6.2 Droit des utilisateurs
- Possibilité de suppression de compte.
- Suppression des données associées selon politique de conservation.

### 6.3 Stockage des documents
Conformément à la configuration client :
- Mode suppression automatique après traitement.
- Mode archivage avec conservation.

---

## 7. Marque blanche et conformité spécifique

En mode marque blanche ou on-premise :
- Paramétrage des politiques de sécurité selon exigences client.
- Possibilité d’adapter les règles de conservation.

---

## 8. Dépendances

Ce module est transverse et dépend de :
- Module Backend API (contrôle d’accès systématique).
- Module Stockage (chiffrement et politiques de rétention).
- Module Comparaison et OCR (journalisation des actions).
- Module Intégrations (contrôle d’accès et audit des connexions externes).

---

# Conclusion fonctionnelle

Le module Sécurité et conformité constitue le socle de confiance de Comparia. Il garantit que toute action est authentifiée, autorisée, tracée et conforme aux exigences de confidentialité et de réglementation applicables aux documents professionnels sensibles.