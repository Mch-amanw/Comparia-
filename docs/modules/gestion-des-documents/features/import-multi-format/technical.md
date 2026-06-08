# Spécification technique – Import multi-format

## 1. Composants impactés

- `DocumentController` (API REST)
- `DocumentService`
- `DocumentParser`
- `StorageProvider`
- Service OCR
- Module Authentification / Autorisation
- Module Audit

---

## 2. API

### 2.1 Endpoint d’upload

`POST /api/documents`

Entrées :
- Fichier binaire (multipart/form-data)
- Métadonnées éventuelles (ex : déclenchement comparaison immédiate)

Traitements :
1. Vérification authentification.
2. Vérification droits d’import.
3. Validation format et taille.
4. Génération UUID.
5. Stockage sécurisé.
6. Création en base (status = IMPORTED).
7. Déclenchement pipeline d’extraction (synchrone ou asynchrone selon taille).

---

## 3. Validation des formats

- Vérification combinée extension + MIME réel.
- Liste blanche des formats autorisés :
  - application/pdf
  - application/msword
  - application/vnd.openxmlformats-officedocument.wordprocessingml.document
  - (prévu) formats Excel

Toute extension Excel est rejetée tant que non activée.

---

## 4. Pipeline de traitement

### 4.1 Étapes

1. Détection MIME réelle.
2. Si PDF :
   - Extraction texte natif.
   - Si texte insuffisant → appel service OCR.
3. Si Word :
   - Parsing structuré (titres, paragraphes, tableaux).
4. Construction représentation normalisée :
   - Segmentation en sections.
   - Hiérarchisation titres.
   - Extraction tableaux.
5. Stockage représentation normalisée (`normalizedPath`).
6. Mise à jour statut → `READY`.

Le pipeline peut être exécuté via file de tâches pour respecter l’objectif < 1 minute et supporter la charge (100–500 comparaisons/jour).

---

## 5. Modèle de données

Extension de l’entité `Document` :

- id (UUID)
- tenantId
- ownerId
- originalFileName
- mimeType
- fileSize
- storagePath
- normalizedPath
- status (IMPORTED, PROCESSING, READY, DELETED, ARCHIVED)
- isScannedPdf (bool)
- ocrPerformed (bool)
- createdAt
- updatedAt

Index recommandés :
- (tenantId, ownerId)
- (tenantId, status)

---

## 6. Gestion inter-format

La représentation normalisée doit :

- Être indépendante du format source.
- Respecter la structure attendue par le moteur de scoring (ST, SH, SB, SS).
- Conserver suffisamment d’information hiérarchique pour le calcul de SH et SS.

Le moteur de comparaison ne doit jamais consommer directement le fichier source, uniquement la version normalisée.

---

## 7. Sécurité

- Upload via TLS.
- Scan antivirus éventuel côté serveur (si activé en configuration).
- Chiffrement au repos via `StorageProvider`.
- Isolation stricte par tenant (répertoires/logiques distincts).
- Suppression physique effective en mode suppression automatique.

---

## 8. Performance

- Support fichiers jusqu’à 50 Mo.
- Traitement optimisé pour documents jusqu’à 300 pages.
- Activation automatique du mode asynchrone pour documents volumineux.
- Timeout et gestion d’erreurs robustes pour OCR.

---

## 9. Journalisation

Événements envoyés au module Audit :
- DOCUMENT_UPLOADED
- DOCUMENT_PROCESSING_STARTED
- DOCUMENT_READY
- DOCUMENT_PROCESSING_FAILED
- DOCUMENT_DELETED

Chaque événement inclut :
- documentId
- tenantId
- userId
- timestamp

---

## 10. Contraintes

- Compatible SaaS et on-premise.
- Aucun couplage fort au branding (marque blanche).
- Versionnement possible de la représentation normalisée pour compatibilité future avec évolutions du moteur de scoring.