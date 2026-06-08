## Ticket : Implémenter l’upload sécurisé de PDF et Word – Spécification technique

### 1. Endpoint API

#### POST /api/documents

Content-Type : multipart/form-data

Entrées :
- file : fichier binaire
- métadonnées optionnelles (si prévues par l’API existante)

Traitement :
1. Vérification du token d’authentification.
2. Vérification des permissions d’import (via module Auth).
3. Validation du fichier.
4. Génération UUID.
5. Stockage sécurisé via StorageProvider.
6. Création de l’entité Document en base.
7. Émission événement Audit.

Réponse :
- 201 Created + documentId si succès.
- 4xx en cas d’erreur de validation.

---

### 2. Validation des fichiers

#### 2.1 Contrôle du format

- Vérification extension (.pdf, .doc, .docx).
- Vérification MIME réel :
  - application/pdf
  - application/msword
  - application/vnd.openxmlformats-officedocument.wordprocessingml.document
- Refus si incohérence extension/MIME.

#### 2.2 Taille maximale

- Limite : 50 Mo.
- Rejet immédiat si dépassement (contrôle côté serveur).

#### 2.3 Vérification d’intégrité minimale

- Tentative d’ouverture via bibliothèque adaptée (PDF parser / Word parser léger).
- Si erreur critique → rejet avec statut 400.

---

### 3. Stockage sécurisé

#### 3.1 StorageProvider

Utilisation de l’abstraction existante `StorageProvider`.

Contraintes :
- Chiffrement au repos.
- Séparation logique par tenant :
  - Exemple de chemin : /{tenantId}/{documentId}/original
- Aucun accès direct public.

#### 3.2 Isolation tenant

- Le tenantId est extrait du contexte utilisateur.
- Le chemin de stockage inclut obligatoirement le tenantId.
- Toute requête ultérieure devra vérifier la correspondance tenant/document.

---

### 4. Persistance en base

Création d’un enregistrement `Document` :

Champs renseignés :
- id (UUID)
- tenantId
- ownerId
- originalFileName
- mimeType
- fileSize
- storagePath
- status = IMPORTED
- createdAt
- updatedAt

Les champs `normalizedPath`, `isScannedPdf`, `ocrPerformed` restent null ou valeur par défaut à ce stade.

---

### 5. Sécurité

- Upload uniquement via HTTPS (TLS obligatoire).
- Contrôle d’accès via middleware d’authentification.
- Validation stricte côté serveur (ne pas se fier au frontend).
- Option de scan antivirus si activée en configuration (hook extensible).

---

### 6. Gestion des erreurs

Cas à gérer :
- Non authentifié → 401
- Non autorisé → 403
- Format non supporté → 400
- Taille dépassée → 413
- Fichier corrompu → 400
- Erreur stockage → 500

Les erreurs doivent être structurées (code + message explicite) sans exposer d’informations sensibles.

---

### 7. Journalisation

Événements Audit à émettre :
- DOCUMENT_UPLOAD_ATTEMPT
- DOCUMENT_UPLOADED
- DOCUMENT_UPLOAD_FAILED

Payload minimal :
- documentId (si disponible)
- tenantId
- userId
- reason (en cas d’échec)
- timestamp

---

### 8. Performance

- Support stable jusqu’à 50 Mo.
- Gestion mémoire contrôlée (streaming recommandé pour éviter chargement complet en mémoire).
- Timeout configurable pour upload.

---

### 9. Compatibilité

- Compatible architecture SaaS (stockage objet) et on-premise (filesystem sécurisé).
- Aucun couplage avec le moteur de comparaison.
- Prépare l’intégration future avec le pipeline d’extraction via changement de statut.