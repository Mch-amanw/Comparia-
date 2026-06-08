## Fonctionnalité : Traitement asynchrone et gestion des performances

### 1. Objectif métier

Garantir une expérience utilisateur fluide et fiable lors de la comparaison de documents, y compris pour des fichiers volumineux (jusqu’à 300 pages), tout en respectant l’objectif de performance :

- Moins d’une minute pour une comparaison standard.
- Traitement sécurisé et stable pour les fichiers plus lourds ou complexes.

La fonctionnalité vise à :
- Éviter les blocages de l’interface utilisateur.
- Assurer la scalabilité du système (100 à 500 comparaisons/jour au lancement).
- Permettre le suivi clair de l’état d’avancement d’une comparaison.

---

### 2. Périmètre fonctionnel

La fonctionnalité couvre :

- La mise en file d’attente des comparaisons.
- Le traitement différé (asynchrone) des tâches lourdes.
- Le suivi de statut d’une comparaison.
- La gestion des délais, erreurs et relances éventuelles.

Elle s’applique à :
- L’extraction de contenu (PDF, Word).
- Le traitement OCR.
- Le moteur de comparaison.
- Le calcul du score de similarité.
- La génération des résultats exploitables par l’Interface de visualisation.

---

### 3. Règles de gestion

#### 3.1 Déclenchement du mode asynchrone

Le traitement peut être :

- Synchrone pour des documents légers (selon seuil interne configurable).
- Asynchrone dès que :
  - Taille importante du fichier,
  - Nombre de pages élevé,
  - OCR requis,
  - Charge système élevée.

La décision est entièrement gérée côté backend.

---

#### 3.2 États d’une comparaison

Chaque comparaison possède un statut :

- `pending` : en attente dans la file.
- `processing` : en cours de traitement.
- `completed` : traitement terminé avec succès.
- `failed` : échec du traitement.

Règles :
- L’utilisateur peut consulter le statut à tout moment (selon droits).
- Une comparaison `completed` devient accessible dans l’Interface de visualisation.
- Une comparaison `failed` affiche un message explicite sans exposer d’informations sensibles.

---

#### 3.3 Expérience utilisateur

Depuis l’Interface de visualisation :

- Affichage d’un état "en cours de traitement".
- Possibilité de rafraîchissement automatique ou manuel.
- Accès aux résultats uniquement lorsque le statut est `completed`.

Règles :
- Aucun affichage partiel incohérent des résultats.
- Le score global ne doit être visible que lorsque le traitement est terminé.

---

#### 3.4 Gestion des erreurs

En cas d’échec (`failed`) :

- L’utilisateur est informé qu’un problème est survenu.
- L’action est journalisée dans le module d’audit.
- Aucun document supprimé (mode suppression automatique) ne doit rester accessible.

---

#### 3.5 Journalisation et traçabilité

Chaque tâche de comparaison doit enregistrer :

- Date et heure de soumission.
- Identité de l’utilisateur.
- Durée totale de traitement.
- Statut final.

Ces informations alimentent le module de journal d’audit.

---

### 4. Critères d’acceptation

1. Une comparaison volumineuse ne bloque jamais l’interface utilisateur.
2. Le statut évolue correctement de `pending` → `processing` → `completed` ou `failed`.
3. Les résultats ne sont visibles qu’après statut `completed`.
4. Le temps de traitement d’un document standard respecte l’objectif (< 1 minute) dans des conditions normales de charge.
5. Toutes les actions sont traçables via le module d’audit.
6. Aucun recalcul de score ou traitement n’est effectué côté frontend.

---

### 5. Dépendances

- Dépend du moteur de comparaison documentaire.
- Dépend du module OCR.
- Dépend du module de scoring (versionné).
- Interagit avec l’Interface de visualisation pour l’affichage des statuts.
- Alimente le module d’audit.

---

## technicalSpec

## Fonctionnalité : Traitement asynchrone et gestion des performances

### 1. Architecture technique

Mise en place d’un système de file de tâches (job queue) côté backend.

Composants impliqués :
- API Backend (soumission des tâches).
- Service de queue.
- Workers de traitement.
- Service OCR.
- Service de comparaison.
- Service de scoring.
- Module d’audit.

Flux général :
1. L’utilisateur soumet une comparaison via l’API.
2. L’API crée un enregistrement en base (statut `pending`).
3. Une tâche est placée dans la file.
4. Un worker consomme la tâche.
5. Le worker exécute :
   - Extraction
   - OCR si nécessaire
   - Normalisation
   - Diff
   - Calcul du score
6. Mise à jour du statut (`processing` → `completed` ou `failed`).

---

### 2. Modèle de données (simplifié)

Entité ComparisonJob :
- id
- documentAId
- documentBId
- status (pending | processing | completed | failed)
- scoringVersion
- createdAt
- startedAt
- completedAt
- durationMs
- errorCode (optionnel)

Indexation sur :
- status
- createdAt
- tenantId (multi-tenant SaaS)

---

### 3. Gestion de la performance

#### 3.1 Scalabilité

- Workers horizontalement scalables.
- Répartition de charge par tenant si nécessaire.
- Limitation configurable du nombre de jobs simultanés.

#### 3.2 Objectif de temps

- Monitoring du temps moyen de traitement.
- Alerting si dépassement fréquent du seuil cible (< 1 minute pour documents standards).

---

### 4. Résilience

- Retry automatique configurable en cas d’échec technique temporaire.
- Timeout maximal par tâche.
- Isolation des tâches (échec d’un job n’impacte pas les autres).

---

### 5. Sécurité

- Vérification des permissions avant création du job.
- Traitement effectué dans un environnement isolé.
- Respect du mode de stockage configuré (suppression automatique ou archivage).
- Chiffrement des données en transit entre services.

---

### 6. Intégration avec l’Interface de visualisation

- Endpoint API :
  - Création d’une comparaison.
  - Récupération du statut.
  - Récupération des résultats lorsque `completed`.

- Aucun calcul métier côté frontend.
- Le frontend doit uniquement consommer les statuts et données finales.

---

### 7. Audit et traçabilité

- Émission d’événements vers le module d’audit :
  - job_created
  - job_started
  - job_completed
  - job_failed

- Conservation des métriques pour analyse de performance.

---