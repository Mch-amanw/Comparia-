# Spécification technique – Connecteurs et API ouvertes

## 1. Architecture générale

La fonctionnalité repose sur trois composants principaux :

1. Service Connecteur SharePoint.
2. Service Connecteur Google Drive.
3. API REST publique versionnée (v1).

Ces composants s’intègrent au backend existant et utilisent :
- Le module d’authentification.
- Le module d’audit.
- Le moteur de comparaison (via API interne).
- Le module de génération de rapports.

---

## 2. Connecteurs externes

### 2.1 Pattern d’implémentation

- Implémentation via Adapter Pattern.
- Interface commune : `ExternalStorageProvider`.
- Méthodes typiques :
  - authenticate()
  - list(path)
  - getFile(fileId)
  - export(file, destination)

Permet l’ajout futur d’autres connecteurs sans modification du cœur.

---

### 2.2 Authentification

- OAuth 2.0 pour SharePoint et Google.
- Stockage des tokens chiffré au repos.
- Rafraîchissement automatique si token expiré.
- Association token ↔ utilisateur ↔ tenant.

Révocation gérée via invalidation côté backend.

---

### 2.3 Flux d’import

1. Sélection utilisateur via frontend.
2. Appel backend vers connecteur.
3. Récupération du flux binaire.
4. Transmission au service de stockage interne.
5. Déclenchement du moteur de comparaison.
6. Journalisation de l’événement.

Compatibilité avec file de tâches asynchrones pour fichiers volumineux.

---

## 3. API REST publique

### 3.1 Principes

- RESTful.
- JSON.
- Versionnement explicite : `/api/v1/...`.
- Stateless.

---

### 3.2 Sécurité

- Authentification par token sécurisé.
- Vérification systématique du tenantId.
- Contrôle des permissions par rôle.
- Rate limiting configurable par tenant.
- Journalisation des requêtes sensibles.

---

### 3.3 Endpoints principaux

- POST `/api/v1/comparisons`
  - Création d’une comparaison.
  - Retourne un identifiant.

- GET `/api/v1/comparisons/{id}`
  - Retourne statut + métadonnées.

- GET `/api/v1/comparisons/{id}/results`
  - Retourne score global, sous-scores, différences.

- GET `/api/v1/comparisons/{id}/report`
  - Téléchargement du rapport.

Les traitements longs génèrent un job asynchrone avec statut (`pending`, `processing`, `completed`, `failed`).

---

## 4. Multi-tenant

- Partition obligatoire par tenantId en base.
- Middleware backend vérifiant la cohérence utilisateur ↔ tenant.
- Configurations tenant (pondérations, stockage, limites API) injectées dynamiquement.

---

## 5. Performance et résilience

- Timeout configuré pour appels externes.
- Gestion des erreurs réseau (retry contrôlé).
- Dégradation propre si service externe indisponible.
- Aucune dépendance bloquante sur connecteurs pour fonctionnement cœur.

---

## 6. Journalisation et audit

Chaque action enregistre :
- Identifiant utilisateur.
- Tenant.
- Type d’opération (import, export, création API, récupération résultat).
- Horodatage.
- Identifiant de comparaison associé.

Logs centralisés et exploitables.

---

## 7. Contraintes de conformité

- HTTPS obligatoire.
- Chiffrement des tokens et secrets.
- Respect RGPD (pas de conservation inutile de données externes).
- Versionnement obligatoire en cas d’évolution des endpoints.

Toute évolution majeure implique la création d’une nouvelle version d’API (`v2`, etc.) pour préserver la compatibilité contractuelle.