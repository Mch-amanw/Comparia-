## Ticket : Mettre en place une file de traitement asynchrone

### 1. Architecture cible

Mise en place d’une architecture orientée tâches basée sur :

- API REST (création de comparaison)
- Base de données (entité `ComparisonJob`)
- Broker de messages (file de traitement)
- Worker(s) stateless de traitement

Le système doit respecter un découplage strict entre :
- Création du job
- Orchestration via la file
- Exécution du traitement

---

### 2. Modèle de données

Utilisation/complétion de l’entité `ComparisonJob` :

Champs minimum requis :
- id (UUID)
- tenantId
- documentAId
- documentBId
- status (pending | processing | completed | failed)
- createdAt
- startedAt (nullable)
- completedAt (nullable)
- durationMs (nullable)
- scoringModelVersion
- errorCode (nullable)
- errorMessage (nullable)

Index recommandés :
- (tenantId, status)
- createdAt

---

### 3. Intégration API

#### 3.1 Création d’un job

Endpoint existant ou à compléter :

POST /comparisons

Comportement :
1. Vérification des permissions utilisateur.
2. Création en base d’un `ComparisonJob` avec statut `pending`.
3. Publication d’un message dans la file contenant :
   - jobId
   - tenantId
   - priority (si applicable)
4. Retour immédiat de la réponse avec :
   - jobId
   - status = pending

Aucune logique lourde ne doit être exécutée dans le thread HTTP.

---

### 4. Broker et file de messages

#### 4.1 Exigences

- Support du mode at-least-once.
- Messages ne contenant que des identifiants (aucune donnée sensible).
- Gestion des retries basiques.

Le choix technologique (RabbitMQ, Kafka, SQS ou équivalent) doit être compatible avec :
- Déploiement SaaS.
- Option on-premise.

---

### 5. Worker de traitement

#### 5.1 Comportement

À la réception d’un message :

1. Charger le `ComparisonJob` par jobId.
2. Vérifier que le statut est `pending`.
3. Passer le statut à `processing` et renseigner `startedAt`.
4. Exécuter séquentiellement :
   - Extraction texte
   - OCR si requis
   - Normalisation
   - Diff
   - Scoring
5. Persister les résultats.
6. Mettre à jour :
   - status = completed
   - completedAt
   - durationMs

En cas d’erreur :
- status = failed
- errorCode / errorMessage renseignés

#### 5.2 Idempotence

Le worker doit :
- Vérifier l’état du job avant traitement.
- Ne pas retraiter un job déjà `processing` ou `completed`.

---

### 6. Audit

Émission d’événements :
- job_created
- job_started
- job_completed
- job_failed

Chaque événement contient :
- jobId
- tenantId
- timestamp
- userId (si applicable)

---

### 7. Sécurité

- Vérification des droits avant création du job.
- Isolation stricte par `tenantId`.
- Aucune donnée document complète dans la file.
- Respect du mode de stockage configuré (suppression ou archivage).

---

### 8. Contraintes techniques

- Workers stateless pour permettre la scalabilité horizontale.
- Timeout configurable par job.
- Gestion propre de l’arrêt des workers (graceful shutdown).
- Logs corrélés par jobId.

Cette implémentation constitue la base du traitement asynchrone, sur laquelle pourront s’appuyer les optimisations de performance, la priorisation et l’autoscaling.