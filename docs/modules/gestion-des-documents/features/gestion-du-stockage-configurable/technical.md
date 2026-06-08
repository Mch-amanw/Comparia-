# Spécification technique : Gestion du stockage configurable

## 1. Extension du modèle de données

### 1.1 Configuration tenant

Ajout dans l’entité Tenant (ou configuration associée) :

- storageMode (ENUM) :
  - SUPPRESSION_AUTOMATIQUE
  - ARCHIVAGE
- retentionDelay (optionnel, en heures ou jours)
- updatedAt
- updatedBy

Les modifications doivent être historisées via le module Audit.

---

## 2. Intégration au cycle de vie du document

### 2.1 Hook post-traitement

À la fin du pipeline d’extraction/normalisation (et après comparaison si applicable), le `DocumentService` vérifie le `storageMode` du tenant.

#### Cas SUPPRESSION_AUTOMATIQUE

- Planification d’une tâche asynchrone de suppression :
  - Immédiate, ou
  - Différée selon `retentionDelay`.

- Suppression :
  - Fichier original (storagePath)
  - Représentation normalisée (normalizedPath) si configuré

- Mise à jour du statut : `DELETED`
- Envoi d’un événement au module Audit.

#### Cas ARCHIVAGE

- Aucun déclenchement automatique de suppression.
- Maintien des fichiers dans le stockage.
- Statut conservé `READY` ou passage explicite à `ARCHIVED`.

---

## 3. Suppression physique

### 3.1 StorageProvider

La suppression doit :
- Supprimer physiquement l’objet du stockage (SaaS : object storage ; on-premise : système de fichiers sécurisé).
- Gérer les erreurs (retry si échec temporaire).
- Garantir l’absence d’accès ultérieur au fichier.

En cas d’échec répété, une alerte technique doit être générée.

---

## 4. Gestion asynchrone

- Utilisation d’une file de tâches pour les suppressions différées.
- Chaque tâche contient :
  - documentId
  - tenantId
  - deadline

Un worker dédié traite les suppressions et met à jour les statuts en base.

---

## 5. Sécurité et isolation

- Vérification systématique du tenantId avant toute suppression.
- Isolation stricte des chemins de stockage par tenant.
- Chiffrement au repos jusqu’à suppression effective.

---

## 6. Journalisation

Événements à tracer :
- STORAGE_MODE_UPDATED
- DOCUMENT_AUTO_DELETED
- DOCUMENT_MANUALLY_DELETED

Chaque log contient :
- documentId (si applicable)
- tenantId
- userId (si action humaine)
- timestamp
- ancien mode / nouveau mode (si modification)

---

## 7. Compatibilité SaaS / On-Premise

- SaaS : suppression via API du fournisseur de stockage objet.
- On-premise : suppression via couche d’abstraction StorageProvider.
- Aucun comportement dépendant d’un branding (compatible marque blanche).

---

## 8. Contraintes

- Aucun impact négatif sur l’objectif de performance (< 1 minute pour traitement standard).
- La suppression automatique ne doit pas bloquer le flux principal (toujours asynchrone si différée).
- La fonctionnalité doit rester compatible avec le versionnement interne des représentations normalisées.