## Spécification technique – Gestion des utilisateurs, rôles et audit

### 1. Architecture

Composants concernés :
- Service d’authentification
- Service d’autorisation (RBAC)
- Middleware de contrôle d’accès
- Service d’audit
- Base de données principale (entités User, Role, Permission, Resource, AuditLog)

La fonctionnalité est transverse et s’intègre à toutes les API métier.

---

### 2. Modèle de données

#### Entités principales
- **User** (id, tenantId, email, passwordHash, status, language, createdAt, updatedAt)
- **Role** (id, tenantId, name)
- **Permission** (id, code, description)
- **UserRole** (userId, roleId)
- **ResourcePermission** (resourceId, userId/roleId, permissionId)
- **AuditLog** (id, tenantId, userId, actionType, resourceType, resourceId, timestamp, result)

Contraintes :
- Champ `tenantId` obligatoire pour toutes les entités multi-tenant.
- Index sur (tenantId, userId), (tenantId, timestamp) pour AuditLog.

---

### 3. Authentification

- Stockage des mots de passe sous forme de hash sécurisé.
- Vérification lors de la connexion.
- Génération d’un token d’authentification signé.
- Expiration configurable.

Mesures de sécurité :
- Limitation des tentatives de connexion.
- Aucune donnée sensible en clair.
- Journalisation des tentatives réussies et échouées.

---

### 4. Autorisation (RBAC)

#### Middleware d’autorisation
- Interception de chaque requête API sensible.
- Vérification :
  1. Validité du token.
  2. Correspondance du tenant.
  3. Présence de la permission requise.
  4. Vérification éventuelle au niveau ressource.

Toute requête non autorisée retourne une erreur standardisée (ex : accès refusé).

---

### 5. Journal d’audit

#### Écriture
- Appel systématique du service d’audit après action critique.
- Écriture possible en mode asynchrone pour ne pas dégrader les performances.

#### Intégrité
- Aucune API de modification ou suppression directe des logs.
- Accès lecture restreint via permission dédiée.

#### Performance
- Indexation optimisée pour consultation par date, utilisateur ou type d’action.
- Impact minimal sur le temps de traitement des comparaisons.

---

### 6. Isolation multi-tenant

- Filtrage automatique par `tenantId` dans toutes les requêtes.
- Vérifications supplémentaires au niveau middleware.
- Tests automatisés garantissant l’absence d’accès inter-tenant.

---

### 7. Contraintes non fonctionnelles

- Ne pas compromettre l’objectif global de comparaison < 1 minute.
- Compatible SaaS et on-premise.
- Scalabilité horizontale.
- Versionnement des modèles de rôles si évolution majeure.

---

### 8. Intégrations internes

- Tous les modules métier doivent appeler le middleware d’autorisation.
- Tous les événements critiques doivent déclencher une écriture dans AuditLog.
- Les exports de rapports et comparaisons doivent vérifier explicitement les permissions avant génération.

Cette fonctionnalité constitue le point central de contrôle d’accès et de traçabilité de Comparia.