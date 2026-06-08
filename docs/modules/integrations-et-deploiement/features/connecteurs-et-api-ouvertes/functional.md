# Fonctionnalité : Connecteurs SharePoint, Google Drive et API ouvertes

## 1. Objectif métier

Permettre aux entreprises d’intégrer Comparia dans leur écosystème documentaire existant (SharePoint, Google Drive) et d’automatiser les comparaisons via des API ouvertes sécurisées.

Cette fonctionnalité vise à :
- Réduire les manipulations manuelles de fichiers.
- Faciliter l’adoption en environnement professionnel.
- Permettre l’intégration avec des outils tiers (GED, ERP, portails internes).
- Garantir sécurité, traçabilité et contrôle des accès.

---

## 2. Périmètre fonctionnel

La fonctionnalité couvre :
- Le connecteur SharePoint.
- Le connecteur Google Drive.
- L’exposition d’une API REST publique versionnée.

Elle n’inclut pas la synchronisation automatique permanente (hors évolution future).

---

## 3. Connecteur SharePoint

### 3.1 Cas d’usage

Un utilisateur autorisé peut :
- Se connecter à son environnement SharePoint.
- Parcourir les bibliothèques et dossiers accessibles.
- Sélectionner un ou deux documents pour comparaison.
- Importer un document dans Comparia.
- (Évolution) Exporter un rapport de comparaison vers SharePoint.

### 3.2 Règles de gestion

- L’utilisateur doit disposer des droits suffisants côté SharePoint et côté Comparia.
- L’accès aux bibliothèques respecte strictement les permissions SharePoint.
- Aucun fichier n’est importé sans action explicite de l’utilisateur.
- Toute opération d’import ou d’export est journalisée (audit).
- Le mode de stockage appliqué (suppression ou archivage) dépend de la configuration du tenant.

### 3.3 Critères d’acceptation

- L’utilisateur peut visualiser l’arborescence SharePoint autorisée.
- L’import d’un document déclenche une comparaison standard.
- Les actions apparaissent dans le journal d’audit.
- Les droits sont correctement appliqués (aucun accès non autorisé).

---

## 4. Connecteur Google Drive

### 4.1 Cas d’usage

Un utilisateur peut :
- S’authentifier via son compte Google.
- Parcourir son Drive (selon permissions).
- Sélectionner des fichiers compatibles.
- Importer des documents pour comparaison.
- (Évolution) Exporter un rapport vers Drive.

### 4.2 Règles de gestion

- Consentement explicite requis via le mécanisme d’authentification Google.
- Les permissions Google Drive sont respectées sans élévation de privilège.
- Aucun accès aux fichiers non autorisés.
- Les tokens d’accès sont associés à l’utilisateur.
- Toutes les opérations sont journalisées.

### 4.3 Critères d’acceptation

- L’utilisateur peut se connecter et voir ses fichiers autorisés.
- Les imports déclenchent une comparaison valide.
- Les accès sont révoqués si l’utilisateur retire le consentement Google.
- Les événements sont traçables dans l’audit.

---

## 5. API ouvertes

### 5.1 Objectif

Permettre à des systèmes tiers de :
- Créer des comparaisons.
- Suivre leur état.
- Récupérer les résultats et rapports.

### 5.2 Capacités principales (v1)

- Création d’une comparaison (upload ou référence externe).
- Récupération du statut (synchrone/asynchrone).
- Récupération des résultats détaillés (score global, sous-scores, différences).
- Téléchargement d’un rapport généré.

### 5.3 Règles de gestion

- Authentification obligatoire.
- Isolation stricte par tenant.
- Les droits API reflètent les rôles du tenant.
- Rate limiting configurable par tenant.
- Versionnement explicite des endpoints.
- Toute requête est journalisée.

### 5.4 Critères d’acceptation

- Une comparaison peut être créée via API et traitée correctement.
- Les réponses respectent le format JSON documenté.
- Les traitements longs retournent un identifiant de job.
- Les limites de débit sont appliquées.
- Les données d’un tenant ne sont jamais accessibles à un autre.

---

## 6. Contraintes transverses

- Respect des exigences RGPD.
- Chiffrement des données en transit.
- Isolation stricte multi-tenant.
- Aucun impact significatif sur les performances globales (< 1 minute pour cas standard hors latence externe).
- Traçabilité complète via le module d’audit.