## Ticket : Ajouter système de commentaires par différence – Spécification technique

### 1. Modèle de données

Utilisation de l’entité existante `Comment` avec contrainte spécifique à ce ticket :

Champs concernés :
- id (UUID)
- tenantId (UUID)
- comparisonId (UUID, FK)
- diffId (UUID, FK, NOT NULL pour ce ticket)
- parentCommentId (UUID, nullable – non utilisé dans ce ticket)
- authorId (UUID, FK)
- content (text)
- status (enum: active, resolved – par défaut active)
- createdAt (timestamp)
- updatedAt (timestamp)
- deletedAt (timestamp, nullable – soft delete)

Contraintes :
- FK vers Comparison (comparisonId).
- FK vers Difference (diffId).
- FK vers User (authorId).
- Index sur (comparisonId, diffId).
- Filtrage systématique sur deletedAt IS NULL pour affichage standard.

---

### 2. API REST

#### 2.1 Création
POST /comparisons/{id}/comments

Body JSON :
- diffId (UUID, requis)
- content (string, requis)

Contrôles :
- Vérification que la différence appartient à la comparaison {id}.
- Vérification des droits RBAC sur la comparaison.
- Injection automatique de tenantId et authorId.

Effets :
- Insertion en base.
- Émission événement temps réel : commentCreated.
- Écriture événement audit (entityType=comment, action=create).

---

#### 2.2 Modification
PATCH /comments/{id}

Body JSON :
- content (string)

Contrôles :
- Vérification existence + deletedAt IS NULL.
- Vérification droits (auteur ou rôle autorisé).
- Vérification cohérence tenantId.

Effets :
- Mise à jour content + updatedAt.
- Émission événement temps réel : commentUpdated.
- Audit (action=update).

---

#### 2.3 Suppression (soft delete)
DELETE /comments/{id}

Contrôles :
- Vérification droits.
- Vérification tenantId.

Effets :
- Mise à jour deletedAt = now().
- Non suppression physique immédiate.
- Émission événement temps réel : commentDeleted.
- Audit (action=delete).

---

### 3. Sécurité

- Authentification obligatoire (JWT/OAuth2).
- Vérification systématique du tenantId.
- Contrôle RBAC basé sur les droits sur comparisonId.
- Aucune route publique.

---

### 4. Intégration temps réel

Événements WebSocket/SSE :
- commentCreated
- commentUpdated
- commentDeleted

Payload minimal :
- commentId
- comparisonId
- diffId
- action
- timestamp

Fallback : polling côté frontend si canal temps réel indisponible.

---

### 5. Audit

Pour chaque action :
- entityType: "comment"
- entityId: commentId
- action: create | update | delete
- userId
- comparisonId
- timestamp
- metadata (diffId)

Conservation selon politique tenant.

---

### 6. Performance et contraintes

- Pagination via GET /comparisons/{id}/comments?diffId=&page=&size= (réutilisation endpoint existant).
- Index sur comparisonId et diffId pour optimiser les lectures.
- Les opérations CRUD ne doivent pas déclencher de recalcul du moteur de comparaison.
- Support cible : usage collaboratif simultané sur 500 comparaisons/jour.

---

### 7. Multi-tenant

- Toutes les requêtes filtrées par tenantId.
- Impossible d’accéder ou modifier un commentaire hors tenant.

---

### 8. Cycle de vie des données

- Soft delete immédiat.
- Suppression physique ultérieure selon politique RGPD/configuration tenant.
- Si la comparaison est supprimée (mode suppression), les commentaires associés deviennent inaccessibles (contrainte FK ou cascade logique).