## TICKET – Spécification technique – Connecteur SharePoint et Google Drive (Import)

### 1. Architecture générale

Implémentation de deux connecteurs :
- `SharePointStorageProvider`
- `GoogleDriveStorageProvider`

Ils implémentent l’interface commune : `ExternalStorageProvider`.

Méthodes minimales requises :
- `authenticate(userContext)`
- `list(path, userContext)`
- `getFile(fileId, userContext)`

Chaque provider est isolé et interchangeable (Adapter Pattern).

---

### 2. Authentification (OAuth 2.0)

#### 2.1 Flux OAuth

- Redirection utilisateur vers le provider externe.
- Récupération d’un authorization code.
- Échange contre access token (+ refresh token si disponible).
- Stockage chiffré des tokens en base.

Association obligatoire :
- userId
- tenantId
- providerType (sharepoint | google)

Les tokens sont :
- Chiffrés au repos.
- Rafraîchis automatiquement si expirés.
- Invalidés en cas d’erreur 401 répétée ou révocation explicite.

---

### 3. Navigation (list)

#### 3.1 SharePoint

- Utilisation des API Microsoft Graph.
- Récupération des bibliothèques accessibles.
- Navigation par dossier via identifiants Graph.

#### 3.2 Google Drive

- Utilisation de l’API Google Drive.
- Requêtes filtrées par permissions utilisateur.
- Support de la pagination.

Réponse normalisée côté backend :

```
{
  id: string,
  name: string,
  type: "file" | "folder",
  mimeType?: string,
  size?: number,
  provider: "sharepoint" | "google"
}
```

Cette normalisation garantit l’indépendance du frontend.

---

### 4. Import de fichier (getFile)

Flux technique :

1. Validation user ↔ tenant.
2. Vérification permission interne (droit de créer une comparaison).
3. Appel API externe pour récupérer le flux binaire.
4. Streaming direct vers le service de stockage interne.
5. Création d’un enregistrement Document lié au tenant.
6. Journalisation de l’événement.

Contraintes :
- Streaming recommandé pour éviter stockage mémoire complet.
- Timeout configuré.
- Retry contrôlé en cas d’erreur réseau.

Formats acceptés :
- PDF
- DOC
- DOCX

Les fichiers volumineux sont compatibles avec le traitement asynchrone du moteur.

---

### 5. Sécurité

- HTTPS obligatoire pour tous les échanges.
- Vérification systématique du tenantId via middleware.
- Isolation stricte des données en base (partition par tenant).
- Aucune conservation des fichiers externes hors stockage interne.

Secrets (clientId, clientSecret) :
- Stockés via mécanisme sécurisé (variables d’environnement ou vault).

---

### 6. Journalisation

Chaque événement génère une entrée d’audit :
- eventType: EXTERNAL_IMPORT
- provider
- userId
- tenantId
- externalFileId
- timestamp
- status (success | failure)

Les erreurs sont également journalisées.

---

### 7. API backend associée

Endpoints internes nécessaires :

- `GET /api/v1/integrations/{provider}/browse`
- `POST /api/v1/integrations/{provider}/import`

Ces endpoints :
- Vérifient l’authentification.
- Injectent automatiquement le tenantId.
- Appliquent rate limiting si configuré.

---

### 8. Résilience et performance

- Gestion des timeouts réseau.
- Retry exponentiel limité.
- Gestion propre des erreurs (message standardisé côté frontend).
- Aucune dépendance bloquante : si le provider est indisponible, le reste de l’application reste opérationnel.

---

### 9. Conformité

- Respect RGPD : aucune conservation inutile de données externes.
- Suppression automatique selon politique tenant.
- Versionnement compatible avec API v1 existante.

Ce ticket livre une version initiale stable et extensible des connecteurs externes (import uniquement), prête à accueillir de futurs providers.