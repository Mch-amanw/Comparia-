# Fonctionnalité : Déploiement SaaS, on-premise et marque blanche

## 1. Objectif métier

Cette fonctionnalité permet à Comparia d’être déployé selon trois modes distincts :

- **SaaS (prioritaire)** : hébergement centralisé géré par l’éditeur.
- **On-premise** : déploiement dans l’infrastructure du client.
- **Marque blanche** : personnalisation de l’identité visuelle pour certains clients.

Elle répond aux besoins des entreprises en matière de sécurité, conformité, intégration IT et alignement avec leur identité de marque, tout en garantissant une cohérence fonctionnelle et technique du produit.

---

## 2. Périmètre fonctionnel

### 2.1 Mode SaaS

Le mode SaaS permet :

- L’accès à Comparia via une instance hébergée.
- Une isolation stricte des données par tenant.
- Une configuration spécifique par client (pondérations du score, stockage, limites API, branding si activé).
- Une scalabilité adaptée aux volumes de traitement.

Règles de gestion :
- Chaque tenant dispose d’une configuration isolée.
- Aucune donnée d’un tenant ne doit être accessible à un autre.
- Les paramètres configurables sont auditables.
- Les performances cibles (< 1 minute pour un document standard) doivent être respectées.

---

### 2.2 Mode on-premise

Le mode on-premise permet :

- L’installation de Comparia dans l’infrastructure du client.
- L’intégration avec les systèmes internes (authentification, stockage, réseau).
- Un paramétrage spécifique (stockage, sécurité, branding).

Règles de gestion :
- Les fonctionnalités cœur (comparaison, scoring, audit, OCR, collaboration) doivent être identiques au mode SaaS.
- Les versions du moteur de scoring doivent être identiques et traçables.
- Les mises à jour doivent être versionnées et documentées.
- Les données restent sous le contrôle exclusif du client.

---

### 2.3 Marque blanche

La marque blanche permet :

- La personnalisation du logo.
- La personnalisation des couleurs principales.
- La personnalisation du nom de l’instance.

Règles de gestion :
- La personnalisation est limitée à l’identité visuelle.
- Aucune modification fonctionnelle n’est autorisée.
- La marque blanche ne doit pas altérer les mécanismes de sécurité, d’audit ou le moteur de scoring.
- Les paramètres de branding sont isolés par tenant.

---

## 3. Paramétrage par tenant

Les éléments configurables incluent :

- Pondérations du score (wT, wH, wB, wS).
- Mode de stockage (suppression automatique ou archivage).
- Limites d’utilisation API.
- Paramètres de marque blanche.

Règles :
- Toute modification de configuration est journalisée.
- Les configurations sont versionnées.
- Les changements n’impactent que le tenant concerné.

---

## 4. Critères d’acceptation

1. Il est possible de créer et configurer un tenant en mode SaaS avec isolation complète.
2. Une instance on-premise déployée offre les mêmes fonctionnalités cœur que le SaaS.
3. Les paramètres de scoring sont modifiables par tenant sans impact global.
4. Les éléments de marque blanche sont visibles uniquement pour le tenant concerné.
5. Toute modification de configuration apparaît dans le journal d’audit.
6. Les performances globales restent conformes aux exigences du projet.