# Module Collaboration – Spécification Technique

## 1. Architecture

Le module repose sur :
- API REST sécurisée
- Service de gestion des commentaires
- Service de gestion des statuts
- Service de partage sécurisé
- Intégration avec module d’audit
- Canal temps réel (WebSocket ou SSE)

Architecture compatible SaaS multi-tenant et on-premise.

---

## 2. Modèle de données

### 2.1 Comment
- id (UUID)
- comparisonId (UUID)
- diffId (UUID, nullable)
- parentCommentId (UUID, nullable)
- authorId (UUID)
- content (text)
- createdAt (timestamp)
- updatedAt (timestamp)
- deletedAt (timestamp, nullable)

### 2.2 DiffStatus
- id (UUID)
- diffId (UUID)
- comparisonId (UUID)
- status (enum: pending, validated, rejected, needsReview)
- updatedBy (UUID)
- updatedAt (timestamp)

### 2.3 ComparisonValidation
- id (UUID)
- comparisonId (UUID)
- globalStatus (enum: pending, validated, validatedWithReservations, rejected)
- updatedBy (UUID)
- updatedAt (timestamp)

### 2.4 ShareLink
- id (UUID)
- comparisonId (UUID)
- token (string unique, hashé en base)
- accessLevel (enum: read, comment, validate)
- passwordHash (nullable)
- allowedDomain (nullable)
- expiresAt (timestamp)
- revokedAt (timestamp, nullable)
- createdBy (UUID)
- createdAt (timestamp)

Indexation :
- comparisonId
- diffId
- token (unique)

---

## 3. API REST (exemples)

### Commentaires
- POST /comparisons/{id}/comments
- GET /comparisons/{id}/comments
- PATCH /comments/{id}
- DELETE /comments/{id}

### Statuts
- PATCH /diffs/{id}/status
- PATCH /comparisons/{id}/status

### Partage
- POST /comparisons/{id}/share
- GET /share/{token}
- DELETE /share/{id}

Toutes les routes nécessitent authentification sauf accès via token sécurisé.

---

## 4. Sécurité

### 4.1 Authentification
- JWT ou OAuth2
- Vérification du tenantId systématique

### 4.2 Autorisation
- Contrôle RBAC au niveau API
- Vérification systématique des droits sur comparisonId

### 4.3 Lien de partage
- Token aléatoire haute entropie (≥ 128 bits)
- Stockage hashé (SHA-256 ou supérieur)
- Vérification expiration + révocation
- Limitation du nombre de requêtes (rate limiting)

---

## 5. Temps réel

- WebSocket ou Server-Sent Events
- Événements :
  - commentCreated
  - commentUpdated
  - diffStatusChanged
  - comparisonValidated
- Fallback polling 5–10 secondes si WebSocket indisponible

---

## 6. Audit

Chaque action génère un événement :
- entityType (comment, diffStatus, shareLink)
- entityId
- action (create, update, delete, validate, revoke)
- userId
- timestamp
- metadata JSON

Conservation selon politique tenant.

---

## 7. Performance

- Pagination des commentaires (par défaut 50)
- Indexation des statuts par comparisonId
- Cache applicatif possible pour lecture fréquente
- Support charge cible : 500 comparaisons/jour avec activité collaborative simultanée

---

## 8. Multi-tenant

- Isolation logique par tenantId
- Clé étrangère tenantId sur toutes les entités
- Vérification systématique côté service

---

## 9. Conformité et conservation

- Suppression logique (soft delete) des commentaires
- Suppression physique selon politique RGPD configurable
- Export possible des commentaires dans le rapport PDF/Word

---

## 10. Versioning

- Compatible avec versioning du scoring
- Les commentaires restent liés à une version spécifique de comparisonId
- Toute nouvelle version de comparaison génère un nouvel identifiant