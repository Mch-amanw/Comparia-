## Ticket : Configurer modes suppression et archivage – Spécification technique

### 1. Extension du modèle de données

#### 1.1 Entité Tenant (ou table de configuration associée)

Ajout des champs suivants :

- `storageMode` (ENUM, non null) :
  - SUPPRESSION_AUTOMATIQUE
  - ARCHIVAGE
- `retentionDelay` (INTEGER, nullable) :
  - Représente un délai en unité standardisée (ex : heures).
- `updatedAt` (timestamp)
- `updatedBy` (UUID utilisateur)

Contraintes :
- Valeur par défaut définie lors de la création du tenant (à aligner avec la configuration produit par défaut).
- Si `storageMode = ARCHIVAGE`, `retentionDelay` peut être null.
- Si `storageMode = SUPPRESSION_AUTOMATIQUE`, `retentionDelay` peut être null (suppression immédiate) ou strictement positif.

---

### 2. API REST

#### 2.1 Consultation de la configuration

Endpoint (exemple) :
- `GET /api/tenants/{tenantId}/storage-config`

Retour :
```json
{
  "storageMode": "SUPPRESSION_AUTOMATIQUE",
  "retentionDelay": 24
}
```

Contrôle :
- Vérification que l’utilisateur appartient au tenant.
- Vérification des droits (au minimum lecture configuration tenant).

---

#### 2.2 Mise à jour de la configuration

Endpoint (exemple) :
- `PUT /api/tenants/{tenantId}/storage-config`

Payload :
```json
{
  "storageMode": "ARCHIVAGE",
  "retentionDelay": null
}
```

Traitement :
1. Vérification rôle administrateur tenant.
2. Validation cohérence des champs.
3. Mise à jour en base.
4. Mise à jour des champs `updatedAt` et `updatedBy`.
5. Émission d’un événement vers le module Audit.

---

### 3. Intégration avec DocumentService

- `DocumentService` doit disposer d’un mécanisme pour récupérer la configuration active du tenant (via repository ou service dédié).
- La configuration doit être consultable de manière performante (possibilité de cache court par tenant si nécessaire).

Aucune logique de suppression n’est implémentée dans ce ticket, uniquement l’exposition et la persistance de la configuration.

---

### 4. Journalisation

À chaque modification :

Événement : `STORAGE_MODE_UPDATED`

Contenu minimal du log :
- tenantId
- userId
- oldStorageMode
- newStorageMode
- oldRetentionDelay
- newRetentionDelay
- timestamp

Le log est transmis au module Audit via le mécanisme standard d’événements applicatifs.

---

### 5. Sécurité

- Vérification systématique du `tenantId` issu du contexte authentifié.
- Interdiction de modifier la configuration d’un autre tenant.
- Protection contre les élévations de privilèges (contrôle strict du rôle administrateur).
- Toutes les communications via HTTPS (TLS).

---

### 6. Compatibilité et contraintes

- Compatible architecture SaaS multi-tenant (isolation stricte par tenantId).
- Compatible déploiement on-premise.
- Ne doit pas introduire de dépendance forte à un fournisseur de stockage.
- Doit rester compatible avec le versionnement interne des représentations normalisées.

---

### 7. Migration

- Prévoir script de migration base de données pour ajout des colonnes.
- Initialisation des tenants existants avec une valeur par défaut cohérente.