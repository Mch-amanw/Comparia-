## Spécification technique – Partage sécurisé des comparaisons

### 1. Composants impactés

- Service Collaboration – ShareLink
- API REST (routes /comparisons/{id}/share et /share/{token})
- Module Authentification & RBAC
- Module Audit
- Base de données (entité ShareLink)
- Middleware de sécurité (rate limiting, validation token)

---

### 2. Modèle de données

Entité : ShareLink

- id (UUID)
- tenantId (UUID)
- comparisonId (UUID, FK)
- token (string, stocké hashé)
- accessLevel (enum: read, comment, validate)
- passwordHash (nullable)
- allowedDomain (nullable)
- expiresAt (timestamp, nullable)
- revokedAt (timestamp, nullable)
- createdBy (UUID)
- createdAt (timestamp)

Contraintes :
- Index unique sur token (hashé).
- Index sur comparisonId.
- Vérification systématique du tenantId.

---

### 3. Génération du token

- Génération via générateur cryptographiquement sûr.
- Entropie minimale : 128 bits.
- Encodage URL-safe.
- Stockage en base sous forme hashée (SHA-256 ou supérieur).
- Le token en clair n’est jamais persisté.

---

### 4. API REST

#### 4.1 Création
POST /comparisons/{id}/share

Body :
- accessLevel
- expiresAt (nullable)
- password (nullable)
- allowedDomain (nullable)

Contrôles :
- Vérification RBAC sur comparisonId.
- Vérification que accessLevel ≤ droits du créateur.

Réponse :
- URL complète contenant le token en clair.

---

#### 4.2 Accès via lien
GET /share/{token}

Étapes serveur :
1. Hash du token reçu.
2. Recherche en base.
3. Vérification :
   - Non révoqué.
   - Non expiré.
   - Comparison existante.
   - Correspondance tenant.
4. Vérification mot de passe si configuré.
5. Vérification domaine si configuré.
6. Application du accessLevel.

Rate limiting appliqué pour éviter attaques par brute force.

---

#### 4.3 Révocation
DELETE /share/{id}

- Vérification droits (créateur ou admin).
- Mise à jour revokedAt.
- Génération événement audit.

---

### 5. Sécurité

- HTTPS obligatoire.
- Tokens haute entropie.
- Stockage hashé.
- Rate limiting sur endpoint public.
- Protection CSRF si session associée.
- Isolation stricte par tenantId.

Les accès via token ne donnent jamais accès à d’autres ressources API que la comparaison ciblée.

---

### 6. Audit

Événements générés :
- shareLinkCreated
- shareLinkAccessed
- shareLinkRevoked
- shareLinkAccessDenied

Métadonnées :
- shareLinkId
- comparisonId
- userId (si authentifié)
- timestamp
- contexte IP / userAgent si disponible

---

### 7. Performance et scalabilité

- Vérification par index unique sur token hashé (lookup O(log n)).
- Aucun impact sur moteur de comparaison.
- Conçu pour supporter 100–500 comparaisons/jour avec multiples liens.
- Cache possible pour métadonnées de lien actif (TTL court).

---

### 8. Intégration UI

- Interface de gestion des liens dans la vue comparaison.
- Indicateur visuel de statut (actif, expiré, révoqué).
- Copie sécurisée de l’URL.
- Rafraîchissement temps réel via WebSocket/SSE lors de création ou révocation.