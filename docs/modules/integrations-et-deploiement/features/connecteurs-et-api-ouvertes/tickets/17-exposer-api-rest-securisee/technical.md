## Ticket : Exposer API REST sécurisée – Spécification technique

### 1. Architecture

L’API REST publique est intégrée au backend principal et exposée via un contrôleur dédié sous le préfixe `/api/v1/`.

Elle s’appuie sur :
- Le module d’authentification.
- Le middleware multi-tenant.
- Le moteur de comparaison (via API interne).
- Le module de génération de rapports.
- Le module d’audit.

Architecture stateless.

---

### 2. Sécurité

#### 2.1 Authentification

- Authentification par token sécurisé (existant dans le module Auth).
- Le token est associé à :
  - un utilisateur,
  - un tenantId.

Chaque requête doit :
- Valider le token.
- Extraire et vérifier le tenantId.
- Vérifier les permissions associées au rôle.

---

#### 2.2 Isolation multi-tenant

- Toutes les requêtes doivent inclure implicitement le tenant via le token.
- Vérification systématique que la ressource demandée appartient au tenant.
- Filtrage en base par clé de partition `tenantId`.

---

#### 2.3 Rate limiting

- Implémentation d’un middleware de limitation de débit.
- Paramétrage par tenant (stocké dans configuration tenant).
- Retour d’un code HTTP approprié en cas de dépassement.

---

### 3. Endpoints

#### 3.1 POST /api/v1/comparisons

Entrée :
- Métadonnées nécessaires à la création.
- Documents via upload multipart ou références existantes.

Traitement :
- Validation des droits.
- Enregistrement en base avec statut `pending`.
- Déclenchement d’un job asynchrone si nécessaire.

Sortie :
- JSON contenant :
  - comparisonId
  - status initial
  - horodatage

---

#### 3.2 GET /api/v1/comparisons/{id}

- Vérification appartenance tenant.
- Retour des métadonnées :
  - status
  - date de création
  - informations synthétiques.

---

#### 3.3 GET /api/v1/comparisons/{id}/results

- Accessible uniquement si statut `completed`.
- Retour JSON structuré contenant :
  - score global S (v1)
  - sous-scores (ST, SH, SB, SS)
  - version du modèle de scoring
  - données structurées des différences.

Respect strict du modèle défini en section 3.4 (Score de similarité – Définition formelle).

---

#### 3.4 GET /api/v1/comparisons/{id}/report

- Vérification des droits.
- Génération à la demande ou récupération d’un rapport existant.
- Retour flux binaire (Content-Type adapté).

---

### 4. Gestion des statuts asynchrones

Statuts supportés :
- `pending`
- `processing`
- `completed`
- `failed`

Le job asynchrone est exécuté via la file de tâches existante.
Les erreurs sont capturées et le statut passe à `failed` avec message interne journalisé.

---

### 5. Journalisation

Chaque appel API journalise :
- userId
- tenantId
- endpoint appelé
- type d’opération
- comparisonId (si applicable)
- timestamp
- statut (succès/échec)

Les logs sont centralisés et exploitables.

---

### 6. Gestion des erreurs

- Codes HTTP standards (400, 401, 403, 404, 429, 500).
- Corps JSON standardisé pour erreurs.
- Aucune exposition de stacktrace ou information interne.

---

### 7. Documentation

- Documentation technique générée (ex : OpenAPI/Swagger).
- Description des endpoints, paramètres, exemples de requêtes/réponses.
- Version explicitement mentionnée (v1).

Toute modification incompatible nécessite la création d’une nouvelle version d’API.