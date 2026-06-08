## Ticket : Développer export Word des résultats – Spécification technique

### 1. Périmètre technique

Implémentation du rendu **DOCX** dans le composant `formatRenderer`, en s’appuyant sur :
- Le `comparisonReportModel v1`.
- Le `reportBuilder` pour la construction du modèle intermédiaire filtré.
- Le `templateEngine` pour application du template standard ou marque blanche.

Aucun changement du modèle de données métier n’est introduit.

---

### 2. Flux technique spécifique DOCX

1. Réception requête POST `/comparisons/{id}/exports` avec `format = docx`.
2. Vérification des droits d’accès.
3. Récupération du `comparisonReportModel v1`.
4. Application des filtres (includeComments, includeValidatedOnly, detailLevel, language).
5. Construction du modèle intermédiaire via `reportBuilder`.
6. Passage au `formatRenderer.docxRenderer`.
7. Génération du fichier DOCX via bibliothèque OpenXML.
8. Calcul du hash SHA-256.
9. Stockage via `exportStorageHandler` selon configuration.
10. Journalisation via `exportAuditLogger`.

---

### 3. Implémentation DOCX

#### 3.1 Bibliothèque

Utilisation d’une bibliothèque compatible **OpenXML (.docx)** permettant :
- Création de documents structurés.
- Définition de styles (Heading 1, Heading 2, Paragraph).
- Gestion du surlignage.
- Gestion des tableaux.

La solution doit être compatible SaaS et on-premise.

---

#### 3.2 Structure interne du document

Le renderer doit générer :

- Page de garde :
  - Titre principal (Heading 1).
  - Bloc métadonnées.
  - Score global mis en évidence.

- Sections structurées :
  - "Synthèse" (Heading 1).
  - "Détail des différences" (Heading 1).
  - Sous-sections par différence ou par section (Heading 2/3).

Les styles doivent être cohérents et réutilisables.

---

#### 3.3 Gestion des différences

Mapping des types :
- addition → texte surligné.
- deletion → texte distinct (ex : style spécifique ou indication textuelle).
- modification → bloc avant/après clairement séparé.
- moved → mention explicite "Section déplacée".

Les extraits doivent être insérés comme texte éditable, non image.

---

#### 3.4 Internationalisation

Le renderer doit utiliser les fichiers de traduction existants.

Les éléments suivants doivent être localisés :
- Titres de sections.
- Libellés (Score global, Synthèse, Ajouts, etc.).
- Statuts (validé, refusé, en attente).

Formatage des dates conforme à la langue sélectionnée.

---

### 4. Performance

- Mode synchrone si ≤ 100 pages.
- Passage automatique en asynchrone au-delà du seuil configuré.
- Optimisation mémoire : génération en flux si possible.

Objectif : < 10 secondes en mode synchrone.

---

### 5. Sécurité

- Vérification des permissions avant génération.
- Fichier généré accessible uniquement via URL signée temporaire.
- Chiffrement au repos si archivage activé.
- Aucune donnée inter-tenant.

---

### 6. Déterminisme

À données identiques et paramètres identiques :
- Le contenu DOCX généré doit être identique (hors métadonnées temporelles).
- Le hash SHA-256 doit être reproductible.

Les timestamps doivent être isolés dans les métadonnées si nécessaires pour préserver le déterminisme du contenu principal.

---

### 7. Journalisation

L’export DOCX doit produire un log contenant :
- exportId
- comparisonId
- format = docx
- paramètres utilisés
- userId
- date/heure
- scoringModelVersion
- hash SHA-256

Transmission au module d’audit central conformément aux spécifications globales.