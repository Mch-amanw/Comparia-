## Ticket : Implémenter l’upload sécurisé de PDF et Word

### 1. Objectif
Mettre en place le mécanisme d’upload sécurisé permettant aux utilisateurs d’importer des documents PDF et Word dans Comparia, avec validation stricte du format, contrôle de la taille maximale (50 Mo) et stockage temporaire sécurisé avant traitement.

Ce ticket couvre exclusivement la phase d’upload initial et de validation. L’extraction, la normalisation et l’OCR sont déclenchés ultérieurement via le pipeline de traitement.

---

### 2. Périmètre fonctionnel

#### 2.1 Formats autorisés
Les formats autorisés sont :
- PDF (.pdf)
- Word (.doc, .docx)

Tout autre format doit être refusé explicitement.

---

### 2.2 Validation à l’import

Lors de l’upload d’un fichier :

1. Vérification de l’authentification de l’utilisateur.
2. Vérification des droits d’import sur le tenant.
3. Vérification du format réel du fichier (MIME type + extension).
4. Vérification de la taille maximale : 50 Mo.
5. Vérification que le fichier n’est pas corrompu (structure lisible).

En cas d’échec d’une validation :
- Le fichier n’est pas stocké.
- Une erreur explicite est retournée à l’utilisateur.
- L’événement est journalisé.

---

### 2.3 Stockage temporaire sécurisé

Si toutes les validations sont conformes :

- Un identifiant unique (UUID) est généré pour le document.
- Le fichier est stocké de manière sécurisée dans l’espace isolé du tenant.
- Le document est enregistré en base avec le statut `IMPORTED`.

Le stockage est considéré comme temporaire tant que le pipeline de traitement (extraction/normalisation) n’a pas été exécuté.

---

### 2.4 Statut du document

Après un upload réussi :
- Création d’une entité Document.
- Statut initial : `IMPORTED`.
- Le passage à `PROCESSING` est déclenché par le pipeline (hors périmètre direct de ce ticket).

---

### 2.5 Journalisation

Les événements suivants doivent être tracés :
- Tentative d’upload.
- Upload réussi.
- Upload refusé (avec motif).

Chaque événement inclut :
- documentId (si généré)
- tenantId
- userId
- horodatage

---

### 2.6 Contraintes fonctionnelles

- L’upload doit fonctionner en environnement SaaS et on-premise.
- Le mécanisme doit supporter des fichiers jusqu’à 50 Mo.
- Le système doit empêcher tout accès à un document d’un autre tenant.
- Aucune dépendance au branding (compatible marque blanche).