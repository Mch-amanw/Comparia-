# Fonctionnalité : Import multi-format

## 1. Objectif métier

Permettre aux utilisateurs d’importer des documents aux formats supportés (PDF, Word) dans Comparia, avec une extensibilité prévue vers Excel, afin de les préparer pour des comparaisons avancées, y compris inter-format (ex : PDF vs Word).

Cette fonctionnalité constitue le point d’entrée du cycle de vie d’un document et garantit que seuls des fichiers valides, sécurisés et exploitables par le moteur de comparaison sont acceptés.

---

## 2. Périmètre

### 2.1 Formats supportés

- PDF (natifs et scannés)
- Word (.doc, .docx)
- Préparation à l’extension vers Excel (.xls, .xlsx)

La fonctionnalité doit permettre :
- L’import d’un document unique.
- L’import de deux documents dans le cadre d’une comparaison immédiate.
- L’import de documents destinés à être comparés ultérieurement.

---

## 3. Règles de gestion

### 3.1 Validation à l’import

À la réception d’un fichier :

- Vérification du format réel (MIME type) et de l’extension.
- Vérification de la taille maximale (jusqu’à 50 Mo selon les exigences projet).
- Vérification de l’intégrité du fichier (non corrompu).
- Refus explicite si le format n’est pas supporté.

En cas d’erreur :
- Le document n’est pas stocké.
- Un message d’erreur explicite est affiché à l’utilisateur.
- L’événement est journalisé.

---

### 3.2 Gestion des comparaisons inter-format

La fonctionnalité doit permettre :

- L’import de deux documents de formats différents (ex : PDF et Word).
- Leur normalisation vers une représentation interne commune.
- L’assurance que la comparaison inter-format repose uniquement sur le contenu textuel et structurel normalisé.

La différence de format source ne doit pas empêcher la comparaison si les deux formats sont supportés.

---

### 3.3 Statut du document

Après import :

1. Statut initial : `IMPORTED`
2. Passage à `PROCESSING` lors de l’extraction/normalisation.
3. Passage à `READY` lorsque le document est exploitable par le moteur de comparaison.

En cas d’échec de traitement :
- Statut d’erreur explicite.
- Information visible par l’utilisateur.

---

### 3.4 Association et droits

Chaque document importé est :

- Associé à un utilisateur (owner).
- Rattaché à un tenant.
- Soumis aux règles de droits d’accès (lecture, comparaison, suppression).

Un utilisateur ne peut importer un document que s’il dispose des droits requis.

---

### 3.5 Prise en charge des PDF scannés

- Si un PDF ne contient pas de texte exploitable, le module déclenche automatiquement le traitement OCR.
- Le document n’est considéré comme `READY` qu’après finalisation de l’OCR et normalisation.

---

## 4. Cas d’usage principaux

1. Upload manuel d’un PDF natif → validation → extraction → prêt à comparer.
2. Upload d’un PDF scanné → détection absence de texte → OCR → normalisation → prêt à comparer.
3. Upload d’un fichier Word → parsing structuré → normalisation → prêt à comparer.
4. Import de deux documents de formats différents pour comparaison immédiate.

---

## 5. Critères d’acceptation

- ✅ Un utilisateur peut importer un PDF ou un Word valide jusqu’à 50 Mo.
- ✅ Un fichier non supporté est refusé avec message explicite.
- ✅ Un PDF scanné déclenche automatiquement l’OCR.
- ✅ Deux documents de formats différents peuvent être importés et préparés pour comparaison.
- ✅ Chaque document importé possède un identifiant unique et un statut cohérent.
- ✅ Les droits d’accès sont appliqués dès l’import.
- ✅ Toutes les actions critiques (upload, échec, suppression) sont journalisées.

---

## 6. Dépendances

- Module OCR (pour PDF scannés).
- Module Authentification & Gestion des rôles.
- Module Audit & Journalisation.
- Moteur de comparaison (consommateur des documents normalisés).
- Module Intégrations (pour imports externes, hors upload manuel).