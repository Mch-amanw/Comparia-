## Ticket : Mettre en place une file de traitement asynchrone

### 1. Objectif

Mettre en place une file de traitement asynchrone permettant d’exécuter les comparaisons de documents en arrière-plan, afin de :

- Éviter tout blocage de l’API ou de l’interface utilisateur.
- Garantir la stabilité du système lors du traitement de documents volumineux.
- Permettre la scalabilité horizontale des traitements.
- Respecter l’objectif de performance (< 1 minute pour un document standard dans des conditions normales).

Ce ticket couvre la mise en place de l’infrastructure de file et son intégration minimale avec la création d’une comparaison.

---

### 2. Périmètre fonctionnel

Le ticket inclut :

1. La création d’une file de tâches dédiée aux comparaisons.
2. La mise en file d’un job de comparaison lors de la soumission via l’API.
3. La mise à jour du statut du job selon son cycle de vie.
4. L’exécution du traitement via un worker consommant la file.

Le ticket ne couvre pas :
- L’optimisation fine des performances.
- Les métriques avancées et alertes détaillées.
- Les règles de sélection synchrone vs asynchrone avancées (seulement la base nécessaire au fonctionnement asynchrone).

---

### 3. Cycle de vie d’une comparaison (version asynchrone)

1. L’utilisateur soumet une comparaison via l’API.
2. Le système :
   - Vérifie les droits.
   - Crée une entité `ComparisonJob` avec statut `pending`.
   - Place un message dans la file de traitement.
3. Un worker consomme le message :
   - Passe le statut à `processing`.
   - Exécute le pipeline complet (extraction, OCR si requis, normalisation, diff, scoring).
   - Persiste les résultats.
   - Met à jour le statut en `completed` ou `failed`.

---

### 4. Règles de gestion

- Chaque comparaison asynchrone doit correspondre à un job unique identifié par un `jobId`.
- Un job ne peut être traité qu’une seule fois logiquement (idempotence obligatoire côté worker).
- En cas d’échec technique, le job passe en `failed`.
- Les statuts doivent être cohérents avec ceux attendus par l’Interface de visualisation (`pending`, `processing`, `completed`, `failed`).
- Les événements de cycle de vie doivent être transmis au module d’audit.

---

### 5. Dépendances

- Dépend du modèle de données `ComparisonJob`.
- Dépend du moteur de comparaison, du module OCR et du module de scoring.
- Interagit avec le module d’audit.
- Expose les statuts à l’Interface de visualisation via l’API.

---

### 6. Impact attendu

- Amélioration immédiate de la robustesse face aux documents lourds.
- Base technique indispensable à la scalabilité SaaS.
- Découplage clair entre soumission API et traitement métier lourd.