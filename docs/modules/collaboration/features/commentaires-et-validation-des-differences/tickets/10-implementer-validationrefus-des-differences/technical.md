## Ticket : Implémenter validation / refus des différences – Spécification technique

### 1. Entités concernées

#### 1.1 DiffStatus
Table existante : `DiffStatus`

Champs utilisés :
- id (UUID)
- tenantId (UUID)
- diffId (UUID)
- comparisonId (UUID)
- status (enum: pending, validated, rejected, needsReview)
- updatedBy (UUID)
- updatedAt (timestamp)

Contraintes :
- Unicité sur (diffId, comparisonId)
- Index sur comparisonId

---

### 2. Initialisation des statuts

Lors de la création d’une comparaison et de la génération des différences :
- Création automatique d’une ligne `DiffStatus` par différence.
- Valeur initiale : `pending`.
- updatedBy = utilisateur système ou créateur de la comparaison.
- updatedAt = timestamp de génération.

Cette initialisation ne doit pas ralentir le traitement principal (batch insert optimisé recommandé).

---

### 3. API REST

#### 3.1 Modification du statut

Endpoint :
`PATCH /diffs/{id}/status`

Body :
```json
{
  "status": "validated" | "rejected" | "needsReview"
}
```

Comportement :
1. Vérification authentification (JWT/OAuth2).
2. Vérification du tenantId.
3. Vérification des droits RBAC sur la comparaison associée.
4. Mise à jour du champ `status`.
5. Mise à jour de `updatedBy` et `updatedAt`.
6. Émission d’un événement temps réel `diffStatusChanged`.
7. Enregistrement d’un événement d’audit.

Réponse :
- 200 OK avec représentation mise à jour du DiffStatus.
- 403 si droits insuffisants.
- 404 si différence inexistante ou hors tenant.

---

#### 3.2 Synthèse des statuts

Endpoint :
`GET /comparisons/{id}/differences/status-summary`

Comportement :
- Requête agrégée groupée par `status`.
- Retour :
```json
{
  "total": number,
  "pending": number,
  "validated": number,
  "rejected": number,
  "needsReview": number
}
```

Optimisation :
- Requête SQL avec GROUP BY.
- Index sur comparisonId.

---

### 4. Logique métier

#### 4.1 Mise à jour transactionnelle
- Mise à jour effectuée dans une transaction courte.
- Pas de recalcul du score.
- Pas de modification des entités Difference.

#### 4.2 Audit
À chaque changement de statut :
- entityType = "diffStatus"
- entityId = id du DiffStatus
- action = "validate" | "reject" | "setNeedsReview"
- userId
- timestamp
- metadata : ancien statut, nouveau statut

---

### 5. Temps réel

Émission via WebSocket ou SSE :

Événement : `diffStatusChanged`
Payload minimal :
```json
{
  "diffId": "UUID",
  "comparisonId": "UUID",
  "status": "validated",
  "updatedAt": "timestamp"
}
```

Fallback : polling 5–10 secondes si canal temps réel indisponible.

---

### 6. Sécurité

- Vérification systématique du tenantId.
- Contrôle RBAC au niveau API.
- Refus de toute modification si comparaison supprimée ou inaccessible.
- Isolation multi-tenant stricte.

---

### 7. Performance

- Mise à jour O(1) par différence.
- Requêtes de synthèse optimisées via index.
- Support cible : activité collaborative simultanée sur 500 comparaisons/jour.
- Aucune interaction bloquante avec le moteur de comparaison.

---

### 8. Intégration avec module Rapport

Les statuts doivent être exposés via un service interne ou endpoint dédié permettant :
- Récupération du statut de chaque différence.
- Inclusion dans les exports PDF/Word.
- Génération d’un résumé de validation.

Aucun recalcul du score n’est effectué lors de l’export.