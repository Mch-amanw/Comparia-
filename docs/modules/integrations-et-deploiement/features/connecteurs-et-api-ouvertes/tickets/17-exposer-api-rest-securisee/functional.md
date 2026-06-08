## Ticket : Exposer API REST sécurisée

### 1. Objectif

Mettre à disposition une API REST publique, versionnée et sécurisée permettant à des systèmes tiers d’interagir avec Comparia pour :

- Créer des comparaisons de documents.
- Suivre l’état d’avancement d’une comparaison.
- Récupérer les résultats détaillés (score global, sous-scores, différences).
- Télécharger les rapports générés.

L’API doit être exploitable par des intégrateurs externes (GED, ERP, portails internes, outils métiers) tout en respectant les exigences de sécurité, d’isolation multi-tenant et de traçabilité définies au niveau projet.

---

### 2. Périmètre fonctionnel

#### 2.1 Versionnement

- L’API est exposée sous un préfixe explicite : `/api/v1/`.
- Toute évolution incompatible nécessitera la création d’une nouvelle version (`v2`, etc.).

---

#### 2.2 Endpoints exposés (v1)

1. **Création d’une comparaison**  
   `POST /api/v1/comparisons`
   - Permet de soumettre deux documents à comparer (upload ou référence externe selon capacités existantes).
   - Retourne un identifiant unique de comparaison.
   - Peut déclencher un traitement asynchrone.

2. **Consultation du statut**  
   `GET /api/v1/comparisons/{id}`
   - Retourne les métadonnées de la comparaison.
   - Inclut le statut (`pending`, `processing`, `completed`, `failed`).

3. **Récupération des résultats**  
   `GET /api/v1/comparisons/{id}/results`
   - Retourne :
     - Score global de similarité.
     - Sous-scores (ST, SH, SB, SS).
     - Métadonnées principales.
     - Données structurées des différences.

4. **Téléchargement du rapport**  
   `GET /api/v1/comparisons/{id}/report`
   - Permet de télécharger le rapport généré (PDF ou Word selon génération disponible).

---

### 3. Règles de gestion

- Authentification obligatoire pour tous les endpoints.
- Isolation stricte des données par tenant.
- Les droits API reflètent les rôles et permissions configurés pour le tenant.
- Application d’un rate limiting configurable par tenant.
- Journalisation de toutes les opérations (création, consultation, téléchargement).
- Respect des règles de stockage du tenant (suppression automatique ou archivage).
- Les traitements longs doivent être gérés de manière asynchrone avec statut consultable.

---

### 4. Contraintes fonctionnelles

- Les réponses sont au format JSON (hors téléchargement de fichier binaire).
- Les champs retournés doivent être cohérents avec le modèle de scoring défini (v1).
- L’API ne doit exposer aucune donnée appartenant à un autre tenant.
- Les messages d’erreur doivent être explicites mais ne jamais divulguer d’informations sensibles.
- L’API doit être documentée (documentation technique destinée aux intégrateurs).

---

### 5. Hors périmètre

- Synchronisation automatique avec systèmes tiers.
- Webhooks (évolution possible ultérieure).
- Gestion avancée des clés API multi-environnements (peut faire l’objet d’un ticket dédié).