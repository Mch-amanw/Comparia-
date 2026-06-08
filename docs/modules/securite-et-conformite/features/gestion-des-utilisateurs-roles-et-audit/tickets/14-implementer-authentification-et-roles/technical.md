## Ticket : Implémenter authentification et rôles – Spécification technique

### 1. Architecture générale

Implémentation de trois composants principaux :
- Service d’authentification
- Service d’autorisation (RBAC)
- Middleware de contrôle d’accès

Ces composants sont intégrés au backend applicatif et s’appliquent à toutes les routes API sensibles.

---

### 2. Modèle de données

#### 2.1 Entités

**User**  
- id (UUID)  
- tenantId (UUID, obligatoire)  
- email (unique par tenant)  
- passwordHash  
- status (ACTIVE, DISABLED)  
- language (FR, EN)  
- createdAt  
- updatedAt

**Role**  
- id  
- tenantId  
- name

**Permission**  
- id  
- code (unique globalement)  
- description

**UserRole**  
- userId  
- roleId

**ResourcePermission**  
- id  
- tenantId  
- resourceType (DOCUMENT, COMPARISON, REPORT)  
- resourceId  
- subjectType (USER, ROLE)  
- subjectId  
- permissionCode

Contraintes :
- Index sur (tenantId, email).
- Index sur (tenantId, resourceId).
- Toutes les requêtes incluent un filtre tenantId.

---

### 3. Authentification

#### 3.1 Stockage des mots de passe
- Hash sécurisé (algorithme robuste conforme aux standards actuels).
- Aucun mot de passe en clair.

#### 3.2 Connexion
Étapes :
1. Vérification de l’existence de l’utilisateur (tenant + email).
2. Vérification du status = ACTIVE.
3. Vérification du hash.
4. Génération d’un token signé.

Le token contient au minimum :
- userId
- tenantId
- rôles
- date d’expiration

#### 3.3 Sessions
- Expiration configurable.
- Vérification systématique du token sur chaque requête protégée.

Mesures complémentaires :
- Limitation des tentatives de connexion.
- Journalisation des tentatives (succès/échec).

---

### 4. Autorisation (RBAC)

#### 4.1 Vérification des permissions globales
Processus :
1. Extraction du token.
2. Vérification de validité et expiration.
3. Récupération des rôles utilisateur.
4. Vérification que l’un des rôles possède la permission requise.

En cas d’échec : réponse d’erreur standardisée (accès refusé).

---

#### 4.2 Vérification au niveau ressource
Pour les opérations liées à un document/comparaison :

1. Vérification permission globale.
2. Vérification correspondance tenant.
3. Vérification d’une règle explicite dans ResourcePermission si restriction activée.

Ordre de priorité :
- Permission explicite USER
- Permission via ROLE
- Sinon refus par défaut

---

### 5. Middleware d’autorisation

- Interception de toutes les routes API sensibles.
- Paramétrage par route : permissionCode requis.
- Intégration centralisée pour éviter la duplication de logique.

Toutes les routes des modules Comparaison, OCR, Rapports et Intégrations devront déclarer explicitement leur permission requise.

---

### 6. Multi-tenant

- tenantId extrait du token.
- Filtrage automatique sur toutes les requêtes base de données.
- Vérification stricte que resource.tenantId == token.tenantId.

Des tests automatisés doivent vérifier l’impossibilité d’accès inter-tenant.

---

### 7. Performance et scalabilité

- Vérifications optimisées via index.
- Mise en cache possible des rôles/permissions par session.
- Impact minimal sur le temps de traitement global (< contrainte projet).

---

### 8. Intégration audit

- Chaque tentative de connexion appelle le service d’audit.
- Les refus d’accès peuvent être journalisés selon politique définie.

Le service d’audit est appelé de manière synchrone ou asynchrone selon criticité.

---

### 9. Contraintes

- Compatible SaaS et on-premise.
- Isolation stricte des tenants.
- Évolutif vers SSO et MFA sans refonte majeure.

Ce ticket établit le socle technique de sécurité pour l’ensemble de la plateforme Comparia.