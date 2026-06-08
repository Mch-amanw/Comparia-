## Ticket : Implémenter authentification et rôles

### 1. Objectif
Mettre en place le socle d’authentification et d’autorisation (RBAC) de Comparia afin de :
- Garantir que seuls les utilisateurs autorisés accèdent à la plateforme,
- Restreindre les actions selon les rôles,
- Appliquer des permissions au niveau des documents et comparaisons,
- Préparer l’intégration complète avec le journal d’audit.

Ce ticket constitue la base critique de sécurité pour l’ensemble des modules.

---

### 2. Périmètre fonctionnel

#### 2.1 Authentification des utilisateurs

Fonctionnalités couvertes :
- Connexion via email + mot de passe.
- Déconnexion manuelle.
- Expiration automatique de session après inactivité.
- Refus d’accès pour les comptes désactivés.

Règles de gestion :
- L’email est unique par tenant.
- Les mots de passe doivent respecter une politique minimale de complexité.
- Toute tentative de connexion (succès ou échec) doit être traçable.
- Une session expirée nécessite une nouvelle authentification.

Hors périmètre du ticket :
- SSO externe (évolution future).
- Authentification multi-facteurs.

---

#### 2.2 Modèle de rôles (RBAC)

Implémentation d’un modèle de rôles configurable par tenant.

Rôles initiaux par défaut :
- Administrateur
- Utilisateur standard
- Lecteur

Chaque rôle regroupe un ensemble de permissions.

Les permissions incluent au minimum :
- uploadDocument
- launchComparison
- viewComparison
- addComment
- validateDifference
- exportReport
- manageUsers
- viewAuditLogs

Règles de gestion :
- Un utilisateur peut posséder un ou plusieurs rôles.
- Les permissions sont évaluées avant toute action sensible.
- Les rôles sont isolés par tenant.

---

#### 2.3 Permissions par document

Mise en place d’un contrôle d’accès au niveau ressource (document, comparaison, rapport).

Fonctionnalités :
- Restreindre l’accès à un document à un sous-ensemble d’utilisateurs ou de rôles.
- Empêcher toute consultation, modification ou export sans permission explicite.

Règles de gestion :
- Les vérifications s’appliquent à chaque accès (lecture, modification, export).
- Un utilisateur sans permission explicite ne doit pas voir la ressource.
- Les contrôles sont systématiques et centralisés.

---

### 3. Contraintes fonctionnelles

- Le système doit fonctionner en mode multi-tenant (isolation stricte).
- Les règles d’accès doivent être homogènes pour tous les modules (Comparaison, OCR, Rapports, Intégrations).
- Les performances globales de la plateforme ne doivent pas être dégradées.

---

### 4. Dépendances

- Module Backend API (intégration middleware).
- Modèle de données User / Role / Permission.
- Journal d’audit (pour traçabilité des connexions et actions).

---

### 5. Résultat attendu

À l’issue de ce ticket :
- La plateforme impose une authentification obligatoire.
- Chaque action métier est protégée par une vérification de permission.
- Les accès aux documents sont contrôlés au niveau ressource.
- L’architecture est prête pour la journalisation complète des actions.