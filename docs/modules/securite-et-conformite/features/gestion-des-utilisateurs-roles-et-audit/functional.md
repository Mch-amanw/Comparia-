## Fonctionnalité : Gestion des utilisateurs, rôles et audit

### 1. Objectif métier
La fonctionnalité **Gestion des utilisateurs, rôles et audit** permet de :
- Contrôler l’accès à la plateforme Comparia via une authentification sécurisée,
- Définir et appliquer des droits d’accès précis basés sur les rôles (RBAC),
- Restreindre les actions au niveau des documents et comparaisons,
- Garantir une traçabilité complète des actions critiques via un journal d’audit.

Elle constitue le socle de confiance garantissant la confidentialité, l’intégrité et la conformité réglementaire (notamment RGPD) de la plateforme.

---

### 2. Périmètre fonctionnel

#### 2.1 Gestion des utilisateurs
- Création d’un compte utilisateur.
- Modification des informations de profil (nom, email, langue FR/EN).
- Désactivation d’un compte.
- Suppression d’un compte (selon droits et politique de conservation).
- Réinitialisation de mot de passe.

Règles de gestion :
- Un utilisateur désactivé ne peut plus se connecter.
- Les mots de passe doivent respecter une քաղաքականité minimale de complexité.
- La langue utilisateur détermine la langue de l’interface.
- Toute création, modification ou suppression de compte est journalisée.

---

#### 2.2 Authentification
- Connexion via identifiant (email) + mot de passe.
- Gestion de session sécurisée.
- Déconnexion manuelle.
- Expiration automatique après inactivité.

Règles de gestion :
- Toute tentative de connexion (réussie ou échouée) est journalisée.
- Un compte désactivé ou supprimé ne peut pas s’authentifier.
- Les sessions expirées nécessitent une nouvelle authentification.

---

#### 2.3 Gestion des rôles (RBAC)

##### Modèle de rôles
Le système repose sur des rôles prédéfinis (configurables par tenant) tels que :
- Administrateur
- Utilisateur standard
- Lecteur

##### Permissions couvertes
Les permissions incluent notamment :
- Téléverser un document
- Lancer une comparaison
- Consulter une comparaison
- Ajouter un commentaire
- Valider/refuser des différences
- Exporter un rapport
- Gérer les utilisateurs
- Consulter les logs d’audit

Règles de gestion :
- Toute action métier est conditionnée à la vérification explicite d’une permission.
- Les permissions sont évaluées avant toute opération sensible.
- Les rôles peuvent être adaptés par tenant dans le respect du modèle RBAC.

---

#### 2.4 Droits par document et comparaison
- Attribution de droits spécifiques sur un document ou une comparaison.
- Restriction d’accès à certaines ressources à un sous-ensemble d’utilisateurs.

Règles de gestion :
- Un utilisateur ne peut consulter ou modifier qu’une ressource pour laquelle il dispose d’un droit explicite.
- Les vérifications s’appliquent à chaque accès (lecture, modification, export).
- Les droits sont isolés par tenant en mode SaaS.

---

#### 2.5 Journal d’audit

##### Événements journalisés
- Connexions et tentatives de connexion.
- Création, modification, désactivation et suppression de comptes.
- Attribution ou modification de rôles.
- Téléversement de documents.
- Lancement et consultation de comparaisons.
- Export de rapports.
- Ajout de commentaires.
- Validation ou refus de différences.
- Suppression de documents.

##### Données enregistrées
Pour chaque événement :
- Identifiant utilisateur
- Tenant associé
- Date et heure
- Type d’action
- Identifiant(s) de ressource concernée(s)
- Résultat (succès / échec)

Règles de gestion :
- Les logs sont non modifiables par les utilisateurs standards.
- Leur consultation est restreinte aux rôles autorisés.
- La durée de conservation respecte la politique définie par le client.

---

### 3. Critères d’acceptation
- Un utilisateur sans permission ne peut pas accéder à une ressource protégée.
- Toute action sensible génère une entrée dans le journal d’audit.
- Un administrateur peut créer, modifier, désactiver un utilisateur.
- Les accès inter-tenant sont techniquement impossibles.
- Les connexions réussies et échouées sont visibles dans les logs.

---

### 4. Dépendances
- Module Backend API (middleware de contrôle d’accès).
- Module Stockage (application des politiques de suppression).
- Modules Comparaison, OCR, Rapports (émission d’événements d’audit).
- Module Intégrations (contrôle d’accès et traçabilité des connexions externes).