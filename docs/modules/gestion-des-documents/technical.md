# Module : Gestion des documents – Spécification technique

## 1. Architecture du module

Le module est exposé via l’API backend et s’intègre dans l’architecture modulaire de Comparia.

Composants principaux :
- API REST "DocumentController"
- Service métier "DocumentService"
- Service d’extraction "DocumentParser"
- Connecteurs d’import (SharePoint, Google Drive)
- Couche d’accès au stockage (StorageProvider)

Il fonctionne de manière synchrone pour l’import initial et peut déclencher des traitements asynchrones pour l’extraction/OCR.

---

## 2. Modèle de données (conceptuel)

### 2.1 Entité Document
Champs principaux :
- id (UUID)
- tenantId
- ownerId
- originalFileName
- mimeType
- fileSize
- storagePath
- normalizedPath (si applicable)
- status (IMPORTED, PROCESSING, READY, DELETED, ARCHIVED)
- createdAt
- updatedAt

### 2.2 Métadonnées complémentaires
- Indicateur PDF scanné
- Langue détectée (si applicable)
- Indicateur OCR effectué

---

## 3. Stockage

### 3.1 Fichiers
- Stockage objet (SaaS) ou stockage local/privé (on-premise).
- Séparation logique par tenant.
- Chiffrement au repos.

### 3.2 Données applicatives
- Métadonnées stockées en base relationnelle.
- Indexation minimale pour recherche par utilisateur/date.

---

## 4. Extraction et normalisation

### 4.1 Pipeline
1. Détection MIME réelle.
2. Si PDF :
   - Tentative extraction texte natif.
   - Si échec ou contenu insuffisant → appel service OCR.
3. Si Word :
   - Parsing structuré (titres, paragraphes, tableaux).
4. Génération d’une représentation normalisée :
   - Texte segmenté (paragraphes, sections).
   - Identification des titres hiérarchiques.
   - Extraction du texte des cellules de tableaux.

La structure produite doit être compatible avec le moteur de scoring (ST, SH, SB, SS).

---

## 5. Sécurité

- Chiffrement TLS pour upload.
- Chiffrement des fichiers au repos.
- Isolation stricte par tenant.
- Contrôle d’accès vérifié à chaque requête API.
- Suppression physique effective en mode "suppression automatique".

---

## 6. Performance

- Upload supportant des fichiers jusqu’à 50 Mo.
- Traitement optimisé pour documents jusqu’à 300 pages.
- Extraction/OCR potentiellement asynchrone via file de tâches.
- Gestion des pics (100–500 comparaisons/jour cible).

---

## 7. Intégrations

### 7.1 SharePoint / Google Drive
- Utilisation d’API officielles.
- Authentification via OAuth si nécessaire.
- Téléchargement temporaire sécurisé avant stockage interne.

---

## 8. Journalisation

Événements à tracer :
- Upload document
- Suppression document
- Accès au document
- Déclenchement extraction/OCR

Les logs sont transmis au module Audit.

---

## 9. Contraintes spécifiques

- Compatible SaaS et on-premise.
- Compatible mode marque blanche (aucune dépendance forte à un branding spécifique).
- Versionnement interne possible des représentations normalisées pour assurer cohérence avec la version du moteur de scoring.