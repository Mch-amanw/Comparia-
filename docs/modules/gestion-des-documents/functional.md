# Module : Gestion des documents

## 1. Rôle du module

Le module **Gestion des documents** est responsable de l’import, du stockage, de la préparation et de la gestion des documents à comparer dans Comparia. Il constitue le point d’entrée de tout flux de comparaison et garantit la conformité aux exigences de sécurité, de performance et de configuration client (suppression automatique ou archivage).

Il prend en charge les formats supportés (PDF, Word, évolution Excel), prépare les documents pour le moteur de comparaison (extraction et normalisation) et applique les règles de droits d’accès.

---

## 2. Fonctionnalités principales

### 2.1 Import de documents

#### 2.1.1 Import manuel (upload)
- Téléversement via interface web.
- Support des formats :
  - PDF (natifs et scannés)
  - Word (.doc, .docx)
  - Évolution : Excel
- Vérification :
  - Taille maximale (jusqu’à 50 Mo selon exigences projet)
  - Format autorisé
  - Fichier non corrompu
- Association du document à :
  - Un utilisateur
  - Un espace/projet (si applicable)

#### 2.1.2 Import via intégrations
- Import depuis :
  - SharePoint
  - Google Drive
- Respect des droits d’accès côté source.
- Récupération sécurisée du fichier.

---

### 2.2 Typologie des documents

Chaque document est caractérisé par :
- Identifiant unique
- Nom original
- Type (PDF, Word, etc.)
- Taille
- Date d’import
- Auteur (utilisateur)
- Statut :
  - Importé
  - En cours de traitement
  - Prêt pour comparaison
  - Supprimé
  - Archivé

Le module ne réalise pas la comparaison mais prépare les documents pour le moteur de comparaison.

---

### 2.3 Préparation des documents

#### 2.3.1 Détection du type
- Identification automatique du type réel du fichier.

#### 2.3.2 Extraction de contenu
- Extraction du texte des PDF natifs.
- Transmission au module OCR pour les PDF scannés.
- Extraction structurée pour Word (titres, paragraphes, tableaux).

#### 2.3.3 Normalisation
- Normalisation du texte pour permettre la comparaison inter-format.
- Conservation d’une représentation structurée compatible avec le moteur de comparaison.

---

### 2.4 Stockage des documents

Deux modes configurables par client (tenant) :

#### 2.4.1 Mode suppression automatique
- Suppression automatique des fichiers sources après traitement ou après délai configuré.
- Conservation éventuelle des résultats de comparaison selon configuration.

#### 2.4.2 Mode archivage
- Conservation des documents originaux.
- Conservation des représentations normalisées.
- Accès ultérieur pour nouvelles comparaisons.

Le choix du mode est déterminé par la configuration du client.

---

### 2.5 Gestion des droits d’accès

- Accès aux documents conditionné par les rôles utilisateurs.
- Droits par document :
  - Lecture
  - Comparaison
  - Suppression
- Les documents ne sont visibles que par les utilisateurs autorisés.

---

### 2.6 Cycle de vie du document

1. Import
2. Validation format/taille
3. Stockage initial sécurisé
4. Extraction/normalisation
5. Mise à disposition pour comparaison
6. Archivage ou suppression selon configuration

Toutes les étapes critiques doivent être journalisées (création, suppression, accès).

---

### 2.7 Contraintes fonctionnelles

- Respect des exigences RGPD.
- Support multilingue (contenu FR/EN minimum).
- Capacité à gérer des documents jusqu’à 300 pages.
- Préparation compatible avec l’objectif de traitement < 1 minute pour documents standards.

---

## 3. Dépendances

- Module OCR (pour PDF scannés).
- Moteur de comparaison (consommateur des documents normalisés).
- Module Authentification & Gestion des rôles.
- Module Audit & Journalisation.
- Module Intégrations (SharePoint, Google Drive).