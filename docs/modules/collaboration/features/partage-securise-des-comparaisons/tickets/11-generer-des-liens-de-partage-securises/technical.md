## Ticket : Générer des liens de partage sécurisés – Spécification technique

### 1. Composants impactés

- API REST – endpoint de création de lien
- Service Collaboration – ShareLinkService
- Module RBAC
- Module Audit
- Base de données – entité ShareLink
- Middleware multi-tenant

---

### 2. Modèle de données

Entité : ShareLink

Champs utilisés/initialisés dans ce ticket :
- id (UUID généré serveur)
- tenantId (UUID, issu du contexte authentifié)
- comparisonId (UUID, FK)
- token (string hashé)
- accessLevel (enum: read, comment, validate)
- passwordHash (nullable)
- allowedDomain (nullable)
- expiresAt (timestamp, nullable)
- revokedAt (null à la création)
- createdBy (UUID)
- createdAt (timestamp)

Contraintes :
- Index unique sur token (hashé).
- Index sur comparisonId.
- Vérification systématique tenantId.

---

### 3. Génération du token

#### 3.1 Exigences
- Entropie minimale : 128 bits.
- Génération via générateur cryptographiquement sûr.
- Encodage URL-safe.

#### 3.2 Stockage
- Le token en clair est généré en mémoire.
- Le token est hashé (SHA-256 ou supérieur).
- Seul le hash est stocké en base.
- Le token en clair est retourné une seule fois dans la réponse API.

Flux :
1. generateSecureRandomBytes()
2. base64urlEncode()
3. hash(token)
4. persist(hash)

---

### 4. API – Création du lien

Endpoint :
POST /comparisons/{id}/share

Body JSON :
{
  "accessLevel": "read" | "comment" | "validate",
  "expiresAt": "ISO-8601 timestamp" (nullable),
  "password": "string" (nullable),
  "allowedDomain": "string" (nullable)
}

---

### 5. Contrôles serveur

#### 5.1 Vérifications d’accès
- Utilisateur authentifié.
- Vérification RBAC sur comparisonId.
- Vérification que accessLevel demandé ≤ droits réels du créateur.
- Vérification que la comparaison appartient au même tenant.

#### 5.2 Validation des paramètres
- expiresAt > now() si fourni.
- password : si fourni → hash (bcrypt ou équivalent).
- allowedDomain : validation format domaine.
- Options restreintes selon configuration tenant.

---

### 6. Réponse API

Retour :
{
  "shareLinkId": "UUID",
  "url": "https://app.comparia.io/share/{token}",
  "accessLevel": "...",
  "expiresAt": "...",
  "createdAt": "..."
}

Le token en clair n’est inclus que dans l’URL retournée.

---

### 7. Audit

À la création :
- entityType: "shareLink"
- entityId: shareLinkId
- action: "create"
- userId
- comparisonId
- metadata: { accessLevel, expiresAt }

L’événement est transmis au module Audit.

---

### 8. Sécurité

- HTTPS obligatoire.
- Aucune persistance du token en clair.
- Hash robuste pour token et mot de passe.
- Isolation stricte tenantId.
- Protection contre élévation de privilèges via contrôle accessLevel.

---

### 9. Performance

- Création O(1) avec insertion indexée.
- Impact négligeable sur moteur de comparaison.
- Compatible charge cible (100–500 comparaisons/jour, plusieurs liens par comparaison).

---

### 10. Compatibilité future

La structure doit être compatible avec :
- Endpoint GET /share/{token} (résolution via hash).
- Révocation (mise à jour revokedAt).
- Rafraîchissement temps réel via WebSocket/SSE pour UI.

Ce ticket implémente uniquement la création sécurisée et persistante des liens de partage.