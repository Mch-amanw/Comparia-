# Module : Moteur de comparaison

## 1. Rôle du module
Le Moteur de comparaison est le service central de Comparia. Il est responsable de :

- L’analyse textuelle fine de deux documents (PDF, Word, inter-format),
- La détection des différences (ajouts, suppressions, modifications),
- L’identification des déplacements de sections,
- L’analyse structurelle (titres, hiérarchie, organisation),
- L’analyse des tableaux (contenu textuel des cellules),
- Le calcul du score global de similarité ainsi que des sous-scores,
- La production d’un résultat structuré exploitable par l’interface, les exports et l’API.

Il constitue le cœur algorithmique du produit.

---

## 2. Entrées du module

Le moteur reçoit :

- Deux documents normalisés (issus des modules d’extraction PDF/Word et éventuellement OCR),
- Une configuration de comparaison :
  - Granularité (mot par défaut, caractère en option),
  - Pondérations du score (wT, wH, wB, wS),
  - Version du modèle de scoring,
  - Paramètres éventuels de tolérance (ex : OCR bruité).

Les documents fournis au moteur sont considérés comme déjà extraits et structurés (texte, titres, tableaux, sections).

---

## 3. Fonctionnalités principales

### 3.1 Comparaison textuelle

- Comparaison mot par mot (par défaut).
- Option caractère par caractère.
- Détection des :
  - Ajouts,
  - Suppressions,
  - Remplacements.
- Production d’un diff structuré exploitable pour surlignage dans l’interface.

Règles de gestion :
- L’unité de référence est le document défini comme "document de référence".
- Les modifications sont comptabilisées selon la formule définie dans la spécification du score (ST = 1 − D/L).
- Les blocs identifiés comme déplacés ne sont pas comptés comme suppression + ajout.

---

### 3.2 Détection des déplacements

- Identification de blocs textuels fortement similaires (≥ 95 % de similarité interne) mais situés à des positions différentes.
- Marquage explicite comme "bloc déplacé".

Règles de gestion :
- Un bloc déplacé :
  - N’impacte pas le score textuel ST,
  - Impacte uniquement le score structurel SS.

---

### 3.3 Analyse des titres et hiérarchie

- Comparaison du texte des titres.
- Prise en compte du niveau hiérarchique (H1, H2, etc.).
- Détection des :
  - Titres modifiés,
  - Titres ajoutés ou supprimés,
  - Changements de niveau hiérarchique.

Règles de gestion :
- Un changement majeur de titre impacte prioritairement SH.
- Une incohérence hiérarchique impacte SH et potentiellement SS.

---

### 3.4 Analyse des tableaux

- Comparaison du texte des cellules.
- Détection des ajouts/suppressions/modifications de contenu.
- Détection des lignes déplacées.

Règles de gestion :
- La mise en forme (bordures, styles) n’est pas prise en compte.
- Les lignes déplacées ne sont pas comptées comme suppression + ajout.
- Le sous-score SB est calculé selon SB = 1 − (Dtable / Ltable).

---

### 3.5 Analyse structurelle

- Analyse de la présence ou absence de sections.
- Analyse de l’ordre des sections.
- Prise en compte des blocs déplacés.

Règles de gestion :
- La similarité structurelle SS est calculée à partir de la correspondance des sections (empreintes normalisées).
- Une réorganisation sans modification textuelle majeure impacte faiblement SS.

---

### 3.6 Calcul du score global

Le moteur calcule :

- ST : similarité textuelle,
- SH : similarité des titres,
- SB : similarité des tableaux,
- SS : similarité structurelle,
- S : score global.

Formule :

S = wT × ST + wH × SH + wB × SB + wS × SS

Règles de gestion :
- Les poids par défaut sont ceux définis dans la spécification (0,60 / 0,15 / 0,15 / 0,10).
- Les poids sont configurables par tenant.
- Le score est exprimé en pourcentage (0 à 100 %).
- Le calcul est déterministe et reproductible.
- Le modèle de scoring est versionné.

---

## 4. Sorties du module

Le moteur retourne une structure de résultat contenant :

- Score global (S),
- Sous-scores (ST, SH, SB, SS),
- Version du modèle de scoring,
- Liste structurée des différences :
  - Type (ajout, suppression, modification, déplacement),
  - Localisation (section, paragraphe, tableau),
  - Contenu avant / après,
- Indicateurs exploitables pour l’affichage côte à côte,
- Métadonnées techniques (temps de traitement, identifiant de tâche).

Ce résultat est utilisé par :
- Le module Frontend (visualisation),
- Le module de génération de rapports,
- Le module d’audit,
- Les API externes.

---

## 5. Contraintes fonctionnelles

- Doit supporter des documents jusqu’à 300 pages.
- Doit fonctionner en comparaison inter-format (PDF vs Word).
- Doit respecter l’objectif de performance (< 1 minute pour un document standard).
- Doit être compatible avec traitement asynchrone.
- Ne doit pas intégrer la mise en page dans le score standard.

---

## 6. Dépendances

- Module d’extraction et normalisation (PDF/Word).
- Module OCR (en amont).
- Module de gestion des tâches asynchrones.
- Module d’audit (journalisation des résultats).

---

# technicalSpec

## 1. Positionnement architectural

Le Moteur de comparaison est un service backend dédié, exposé via API interne (REST ou équivalent).

Il peut être :
- Déployé comme service stateless scalable horizontalement (SaaS),
- Déployé en instance dédiée en environnement on-premise.

Il est conçu pour être appelé de manière synchrone (petits documents) ou asynchrone via file de tâches.

---

## 2. Pipeline de traitement

### 2.1 Étapes principales

1. Réception des deux représentations normalisées des documents.
2. Segmentation en unités logiques :
   - Sections,
   - Titres,
   - Paragraphes,
   - Tableaux.
3. Normalisation (ex : casse, espaces, caractères spéciaux selon configuration).
4. Comparaison textuelle (algorithme de diff).
5. Détection des blocs similaires déplacés.
6. Calcul des sous-scores (ST, SH, SB, SS).
7. Agrégation du score global.
8. Génération d’un objet résultat structuré.

---

## 3. Algorithmes

### 3.1 Diff textuel

- Implémentation basée sur un algorithme de type Myers diff ou LCS optimisé.
- Support de deux granularités : mot (défaut) et caractère (option).
- Optimisation mémoire pour documents volumineux.

### 3.2 Détection de déplacements

- Création d’empreintes (hash) pour blocs normalisés.
- Mesure de similarité interne (≥ 95 %).
- Correspondance entre blocs hors position initiale.

### 3.3 Analyse structurelle

- Construction d’un arbre hiérarchique des sections.
- Hash par section normalisée.
- Analyse des correspondances, absences et permutations.

### 3.4 Calcul du score

- Implémentation stricte de la formule définie en section 3.4 du projet.
- Paramétrage des poids par tenant.
- Versionnement explicite du modèle (ex : scoringModelVersion = "v1").

---

## 4. Modèle de données (conceptuel)

### 4.1 Input (simplifié)
- DocumentNormalized
  - sections[]
  - titles[]
  - paragraphs[]
  - tables[]

### 4.2 Output (simplifié)
- ComparisonResult
  - globalScore
  - textScore
  - titleScore
  - tableScore
  - structureScore
  - scoringModelVersion
  - differences[]
  - processingMetadata

---

## 5. Performance et scalabilité

- Optimisé pour documents jusqu’à 300 pages.
- Gestion mémoire maîtrisée (streaming ou traitement par blocs si nécessaire).
- Compatible avec exécution en worker isolé.
- Possibilité de paralléliser certaines analyses (ex : sections indépendantes).

Objectif :
- Temps total < 1 minute pour un document standard (selon exigences projet).

---

## 6. Sécurité et conformité

- Ne stocke pas durablement les documents (respect du mode de stockage configuré globalement).
- Traite uniquement des représentations internes sécurisées.
- Journalisation technique minimale (identifiants, métriques) sans exposer le contenu complet dans les logs.

---

## 7. Dépendances techniques

- Service d’extraction documentaire.
- Service OCR (en amont).
- Système de file de tâches pour traitement asynchrone.
- Module de configuration tenant (pondérations, options).

---