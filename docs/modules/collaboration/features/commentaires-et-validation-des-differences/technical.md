# Spécification technique – Commentaires et validation des différences

## 1. Composants impactés

- Backend API REST (gestion des commentaires et statuts).
- Base de données (tables Comment, DiffStatus).
- Module Audit (journalisation des actions).
- Frontend de comparaison (affichage contextualisé et filtres).
- Canal temps réel (WebSocket ou SSE) pour mise à jour dynamique.

---

## 2. Modèle de données

### 2.1 Comment
Champs principaux :
- id (UUID)
- tenantId (UUID)
- comparisonId (UUID)
- diffId (UUID, nullable)
- parentCommentId (UUID, nullable)
- authorId (UUID)
- content (text)
- status (enum: active, resolved)
- createdAt (timestamp)
- updatedAt (timestamp)
- resolvedAt (timestamp, nullable)
- resolvedBy (UUID, nullable)
- deletedAt (timestamp, nullable – soft delete)

Contraintes :
- FK vers comparisonId, diffId et authorId.
- Index sur comparisonId et diffId.

### 2.2 DiffStatus
Champs principaux :
- id (UUID)
- tenantId (UUID)
- diffId (UUID)
- comparisonId (UUID)
- status (enum: pending, validated, rejected, needsReview)
- updatedBy (UUID)
- updatedAt (timestamp)

Contraintes :
- Unicité (diffId, comparisonId).
- Index sur comparisonId pour requêtes de synthèse.

---

## 3. API REST

### 3.1 Commentaires
- POST /comparisons/{id}/comments
- GET /comparisons/{id}/comments?diffId=&page=&size=
- PATCH /comments/{id}
- POST /comments/{id}/resolve
- DELETE /comments/{id} (soft delete)

### 3.2 Validation des différences
- PATCH /diffs/{id}/status
- GET /comparisons/{id}/differences/status-summary

Toutes les routes :
- Authentification obligatoire (JWT/OAuth2).
- Vérification du tenantId et des droits RBAC.

---

## 4. Logique métier

### 4.1 Gestion des statuts
- Création automatique d’un statut "pending" pour chaque différence lors de la génération de la comparaison.
- Toute mise à jour écrase le statut précédent (historique conservé via audit).

### 4.2 Résolution des commentaires
- Passage du champ status à "resolved".
- Enregistrement resolvedAt et resolvedBy.

### 4.3 Vue synthétique
Le endpoint de synthèse :
- Agrégation groupée par status.
- Calcul optimisé via requête SQL avec index.

---

## 5. Temps réel

Événements émis via WebSocket/SSE :
- commentCreated
- commentUpdated
- commentResolved
- diffStatusChanged

Payload minimal : entityId, comparisonId, type, timestamp.

Fallback : polling toutes les 5–10 secondes.

---

## 6. Sécurité

- Vérification systématique des droits sur comparisonId.
- Isolation multi-tenant via tenantId.
- Soft delete des commentaires (deletedAt).
- Journalisation automatique dans le module Audit :
  - entityType (comment, diffStatus)
  - entityId
  - action (create, update, resolve, delete, validate)
  - userId
  - timestamp

---

## 7. Performance et contraintes

- Pagination des commentaires (par défaut 50 par page).
- Index sur comparisonId et diffId pour limiter les temps de réponse.
- Les opérations de commentaires et validation ne doivent pas bloquer ni relancer le moteur de comparaison.
- Conçu pour supporter 500 comparaisons/jour avec usage collaboratif simultané.

---

## 8. Intégration avec rapports

Le module expose :
- Liste structurée des commentaires par différence.
- Statut de chaque différence.
- Synthèse globale des validations.

Ces données sont consommées par le module Rapport sans recalcul du score.

---

## 9. Versioning

- Les commentaires et statuts sont liés à un comparisonId spécifique.
- Toute nouvelle comparaison génère un nouvel ensemble indépendant de commentaires et statuts.
- Compatible avec versioning du modèle de scoring (v1 et suivants).