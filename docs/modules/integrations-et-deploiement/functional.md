# Module : Intégrations et déploiement

## 1. Rôle du module dans Comparia

Le module **Intégrations et déploiement** permet :

- L’intégration de Comparia avec des systèmes tiers (SharePoint, Google Drive, API ouvertes).
- L’exposition d’API sécurisées pour des intégrations personnalisées.
- La gestion des différents modes de déploiement : SaaS, on-premise et marque blanche.

Il garantit que Comparia peut s’inscrire dans l’écosystème IT des entreprises tout en respectant les exigences de sécurité, de traçabilité et de configuration par tenant.

---

## 2. Fonctionnalités – Intégrations externes

### 2.1 Connecteur SharePoint

Permet aux utilisateurs autorisés de :
- Parcourir des bibliothèques SharePoint.
- Sélectionner des documents pour comparaison.
- Importer des documents dans Comparia.
- (Évolution) Exporter un rapport de comparaison vers SharePoint.

Règles de gestion :
- L’accès dépend des droits de l’utilisateur dans SharePoint et dans Comparia.
- Aucune synchronisation automatique permanente sans action explicite.
- Les actions d’import/export sont journalisées.

---

### 2.2 Connecteur Google Drive

Permet :
- Authentification via compte Google.
- Navigation dans l’arborescence Drive.
- Sélection et import de documents.
- (Évolution) Export de rapports vers Drive.

Règles de gestion :
- Consentement utilisateur requis pour l’accès.
- Respect des permissions Drive existantes.
- Traçabilité complète des opérations.

---

### 2.3 API ouvertes

Comparia expose des API permettant :

- Création d’une comparaison (upload ou référence à un fichier externe).
- Récupération du statut d’une comparaison (synchrone/asynchrone).
- Récupération des résultats (différences, score global, sous-scores).
- Téléchargement des rapports générés.
- Consultation des métadonnées de comparaison.

Règles de gestion :
- Authentification obligatoire.
- Les droits API reflètent les rôles et permissions du tenant.
- Versionnement explicite des API.
- Limitation de débit (rate limiting) par tenant.

---

## 3. Fonctionnalités – Modes de déploiement

### 3.1 Mode SaaS (prioritaire)

- Hébergement mutualisé ou isolé logiquement par tenant.
- Configuration spécifique par client (stockage, pondérations de score, branding si applicable).
- Scalabilité horizontale pour les traitements intensifs.

Règles :
- Isolation stricte des données entre tenants.
- Paramétrage par tenant sans impact sur les autres.

---

### 3.2 Mode on-premise

Permet un déploiement dans l’infrastructure du client :
- Installation contrôlée.
- Paramétrage spécifique (stockage, sécurité, branding).
- Intégration avec systèmes internes.

Règles :
- Les fonctionnalités cœur (comparaison, score, audit) doivent rester identiques au mode SaaS.
- Les mises à jour doivent être versionnées et traçables.

---

### 3.3 Marque blanche

Permet à certains clients :
- Personnalisation du logo.
- Personnalisation des couleurs principales.
- Personnalisation du nom de l’instance.

Règles :
- Aucune modification fonctionnelle.
- La marque blanche ne doit pas altérer les mécanismes de sécurité ni le moteur de scoring.

---

## 4. Contraintes fonctionnelles transverses

- Toutes les opérations d’intégration doivent être journalisées (audit).
- Les accès sont soumis aux rôles et permissions.
- Les configurations (pondérations du score, stockage, branding) sont isolées par tenant.
- Les intégrations ne doivent pas compromettre les performances globales (< 1 minute pour cas standard).

---

## 5. Dépendances

Ce module dépend :
- Du module d’authentification et gestion des rôles.
- Du module d’audit.
- Du moteur de comparaison (via API interne).
- Du module de génération de rapports.

Il est requis pour l’ouverture de Comparia à des environnements d’entreprise et pour les stratégies de distribution (SaaS, on-premise, marque blanche).