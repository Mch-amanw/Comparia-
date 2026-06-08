## Spécification technique : Traitement asynchrone et gestion des performances

### 1. Architecture technique

#### 1.1 Composants impliqués

- API Backend (REST)
- Service de queue (ex : message broker)
- Workers de traitement (comparaison, OCR, scoring)
- Base de données (métadonnées, statuts)
- Service de monitoring

Architecture orientée tâches avec découplage strict entre :
- Soumission de la comparaison,
- Exécution,
- Consultation des résultats.

---

### 2. Modèle de tâche

Entité ComparisonJob :

- id
- tenantId
- documentAId
- documentBId
- status (pending, processing, completed, failed, cancelled)
- priority
- createdAt
- startedAt
- completedAt
- durationMs
- errorCode (nullable)
- errorMessage (nullable)
- scoringModelVersion

Indexation par tenantId et status pour performance.

---

### 3. File de messages

#### 3.1 Mécanisme

- Utilisation d’un broker (ex : RabbitMQ, Kafka, SQS ou équivalent).
- Message contenant : jobId, tenantId, priorité.

#### 3.2 Garanties

- Traitement au moins une fois (at-least-once).
- Idempotence obligatoire côté worker (basée sur jobId).
- Gestion des retries avec backoff exponentiel.
- Dead-letter queue pour échecs répétés.

---

### 4. Workers de traitement

Chaque worker exécute :

1. Extraction texte (PDF/Word)
2. OCR si requis
3. Normalisation
4. Calcul diff
5. Calcul scoring
6. Persistance des résultats
7. Mise à jour statut

Contraintes :
- Stateless (scalabilité horizontale).
- Timeout configurable par tâche.
- Limitation mémoire pour documents volumineux.

---

### 5. Sélection synchrone vs asynchrone

Logique côté API :

- Analyse taille fichier, nombre estimé de pages, OCR requis.
- Si sous seuil configurable et charge faible → tentative synchrone.
- Sinon → création immédiate d’un job asynchrone.

Un fallback automatique vers asynchrone est déclenché si dépassement d’un seuil de temps.

---

### 6. Scalabilité

#### 6.1 SaaS

- Autoscaling horizontal des workers.
- Métriques surveillées :
  - Longueur de la file,
  - Temps moyen de traitement,
  - CPU / mémoire workers.

#### 6.2 On-premise

- Nombre de workers configurable.
- Paramètres ajustables : concurrence maximale, taille max fichier.

---

### 7. Performance cible

- Temps moyen < 60 secondes pour documents standards.
- Capacité initiale : 500 comparaisons/jour.
- Dimensionnement pour montée en charge via scaling horizontal.

---

### 8. Monitoring et observabilité

Collecte de métriques :

- Durée moyenne par étape (OCR, diff, scoring),
- Taux d’échec,
- Temps d’attente en file,
- P95 / P99 des durées de traitement.

Logs corrélés par jobId.

Alertes configurables si :
- Temps moyen > seuil,
- Taux d’échec > seuil,
- File saturée.

---

### 9. Sécurité et isolation

- Isolation stricte des données par tenantId.
- Vérification des permissions avant création du job.
- Aucune donnée sensible dans les messages de queue (uniquement identifiants).

---

### 10. Compatibilité avec l’Interface de visualisation

- L’API expose :
  - POST /comparisons → création job
  - GET /comparisons/{id} → statut
  - GET /comparisons/{id}/result → résultats si completed

- Les statuts sont conformes à ceux attendus par le module Interface de visualisation.
- Aucun recalcul côté frontend.

La conception garantit un découplage complet entre traitement lourd backend et affichage frontend.