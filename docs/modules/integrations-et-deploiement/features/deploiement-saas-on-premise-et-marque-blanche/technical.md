# Spécification technique – Déploiement SaaS, on-premise et marque blanche

## 1. Architecture générale

La fonctionnalité repose sur :

- Un modèle **multi-tenant** pour le SaaS.
- Un packaging standardisé pour le on-premise.
- Un système de configuration centralisée par tenant.
- Une injection dynamique des paramètres de branding côté frontend.

---

## 2. Mode SaaS

### 2.1 Isolation multi-tenant

- Chaque requête backend inclut un identifiant de tenant obligatoire.
- Partition logique des données (clé de partition par tenant).
- Vérification systématique des permissions au niveau applicatif.

### 2.2 Configuration par tenant

Stockage des paramètres :
- Table ou collection dédiée aux configurations.
- Versionnement des configurations.
- Historisation des modifications (audit trail).

Paramètres configurables :
- Pondérations du score.
- Mode de stockage.
- Paramètres de branding.
- Limites API.

### 2.3 Scalabilité

- Services de comparaison et OCR découplés.
- File de tâches pour traitements asynchrones.
- Scalabilité horizontale des services en environnement SaaS.

---

## 3. Mode on-premise

### 3.1 Packaging

- Distribution via conteneurisation recommandée.
- Configuration par variables d’environnement ou fichier sécurisé.

### 3.2 Configuration

- Paramètres équivalents au mode SaaS.
- Stockage local ou configuré par le client.
- Compatibilité avec systèmes d’authentification internes.

### 3.3 Versioning

- Version explicite de l’application.
- Version du moteur de scoring incluse.
- Documentation des dépendances externes.

---

## 4. Marque blanche

### 4.1 Stockage

- Paramètres stockés par tenant (logo, couleurs, nom d’instance).
- Références vers fichiers stockés de manière sécurisée.

### 4.2 Injection dynamique

- Chargement des paramètres au login ou à l’initialisation de session.
- Application dynamique des styles côté frontend.
- Aucun code spécifique par client.

---

## 5. Sécurité et conformité

- HTTPS obligatoire.
- Chiffrement des données sensibles.
- Journalisation des modifications de configuration.
- Isolation stricte entre tenants en SaaS.

---

## 6. Dépendances

Cette fonctionnalité dépend :

- Du module d’authentification et gestion des rôles.
- Du module d’audit.
- Du moteur de scoring (configuration des pondérations).
- Du module de stockage.

Elle doit rester compatible avec les connecteurs externes et les API ouvertes.