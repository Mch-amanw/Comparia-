## Ticket : Implémentation technique du journal d’audit

### 1. Architecture générale

Mettre en place un **AuditService** centralisé utilisé par tous les modules métier.

Deux composants principaux :
- AuditService (logique applicative)
- AuditRepository (accès base de données)

L’écriture des logs peut être réalisée de manière asynchrone afin de minimiser l’impact sur les performances.

---

### 2. Modèle de données

#### Entité : AuditLog

Champs obligatoires :

- id (UUID)
- tenantId (UUID)
- userId (UUID, nullable si non authentifié)
- actionType (string normalisé)
- resourceType (string)
- resourceId (UUID ou string selon ressource)
- timestamp (datetime UTC généré côté serveur)
- result (enum : SUCCESS | FAILURE)

Contraintes :
- Index sur (tenantId, timestamp)
- Index sur (tenantId, userId)
- Index sur (tenantId, actionType)
- Champ tenantId obligatoire pour isolation multi-tenant

Aucune relation en cascade destructive ne doit supprimer les logs.

---

### 3. Intégration avec les modules métier

#### 3.1 Appel systématique
Chaque action critique doit appeler AuditService.logEvent(...)

Signature type :

logEvent({
  tenantId,
  userId,
  actionType,
  resourceType,
  resourceId,
  result
})

L’appel doit être effectué :
- Après validation des permissions.
- Après exécution de l’action (pour capturer le résultat réel).

---

### 4. Middleware et récupération du contexte

Le middleware d’authentification doit :
- Injecter userId et tenantId dans le contexte de requête.
- Rendre ces informations accessibles au AuditService.

En cas d’échec d’authentification, un log doit tout de même être créé (avec userId nullable si inconnu).

---

### 5. Performance

- Possibilité d’écriture asynchrone via file interne ou event bus.
- En cas d’échec d’écriture du log, l’erreur doit être journalisée côté système mais ne pas bloquer l’action métier principale.
- Le temps d’écriture d’un log ne doit pas impacter significativement les comparaisons.

---

### 6. Sécurité et intégrité

- Aucune API publique ne doit permettre modification ou suppression d’un AuditLog.
- L’accès en lecture doit être protégé par une permission dédiée (ex : VIEW_AUDIT_LOGS).
- Toutes les requêtes de consultation doivent filtrer strictement par tenantId.

---

### 7. API de consultation

Endpoint REST sécurisé (exemple) :
GET /api/audit-logs

Paramètres possibles :
- startDate
- endDate
- userId
- actionType
- page / size

Le backend doit :
- Vérifier la permission VIEW_AUDIT_LOGS.
- Appliquer automatiquement le filtre tenantId.
- Retourner les résultats paginés.

---

### 8. Rétention

Le service doit être compatible avec un mécanisme de purge planifiée :
- Suppression selon politique définie par tenant.
- Tâche planifiée distincte.

La suppression ne doit pas violer les contraintes légales définies par le client.

---

### 9. Compatibilité

- Compatible SaaS (multi-tenant strict).
- Compatible on-premise.
- Aucun impact sur le moteur de comparaison.

Ce ticket établit l’infrastructure technique complète du journal d’audit et son intégration transverse à toute la plateforme.