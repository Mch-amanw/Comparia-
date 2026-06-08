# Module : Intégrations et déploiement – Spécification technique

## 1. Architecture du module

Le module est composé de trois sous-domaines techniques :

1. Connecteurs externes (SharePoint, Google Drive)
2. API publiques
3. Gestion du déploiement et configuration multi-tenant

Il s’intègre dans l’architecture backend via des services dédiés exposés par l’API REST principale.

---

## 2. Connecteurs externes

### 2.1 Architecture

Chaque connecteur est implémenté comme :
- Un service dédié (adapter pattern).
- Une couche d’abstraction commune (interface générique de stockage externe).

Objectif :
- Permettre l’ajout futur d’autres connecteurs sans modifier le cœur applicatif.

---

### 2.2 Authentification

- Utilisation des mécanismes standards des plateformes (OAuth 2.0).
- Stockage sécurisé des tokens (chiffrement au repos).
- Rafraîchissement automatique des tokens si applicable.

---

### 2.3 Flux d’import

1. L’utilisateur sélectionne un fichier via le connecteur.
2. Le backend récupère le flux binaire.
3. Le document est transmis au service de stockage interne ou au moteur de traitement.
4. L’événement est journalisé.

Les imports doivent être compatibles avec le traitement asynchrone si nécessaire.

---

## 3. API ouvertes

### 3.1 Principes

- API RESTful.
- Versionnement explicite (ex: /api/v1/...).
- Format JSON.

---

### 3.2 Sécurité

- Authentification par token sécurisé.
- Isolation stricte par tenant (identifiant de tenant obligatoire).
- Rate limiting configurable.

---

### 3.3 Endpoints principaux (v1)

- POST /comparisons → création d’une comparaison.
- GET /comparisons/{id} → statut et métadonnées.
- GET /comparisons/{id}/results → résultats détaillés.
- GET /comparisons/{id}/report → téléchargement du rapport.

Les traitements longs déclenchent un job asynchrone avec identifiant de suivi.

---

## 4. Multi-tenant et configuration

### 4.1 Isolation

- Séparation logique des données par tenant (clé de partition obligatoire).
- Contrôle systématique côté backend.

---

### 4.2 Configuration par tenant

Paramètres configurables :
- Pondérations du score (wT, wH, wB, wS).
- Mode de stockage (suppression ou archivage).
- Paramètres de marque blanche.
- Limites API.

Les configurations sont versionnées et auditables.

---

## 5. Déploiement SaaS

- Environnement scalable horizontalement.
- Services de traitement découplés (comparaison, OCR).
- File de tâches pour traitements lourds.

Objectif : maintenir le SLA (< 1 minute standard).

---

## 6. Déploiement on-premise

- Packaging standardisé (conteneurisation recommandée).
- Configuration via variables d’environnement ou fichier sécurisé.
- Compatibilité avec systèmes d’authentification internes.

Les dépendances externes doivent être clairement documentées.

---

## 7. Marque blanche

- Paramètres stockés par tenant (logo, couleurs, nom).
- Injection dynamique dans le frontend.
- Aucune duplication de code spécifique par client.

---

## 8. Contraintes non fonctionnelles

- Chiffrement des communications (HTTPS obligatoire).
- Journalisation centralisée des appels API.
- Surveillance des erreurs d’intégration.
- Compatibilité avec les exigences RGPD.

Toute évolution des API entraîne une nouvelle version afin de préserver la compatibilité contractuelle.