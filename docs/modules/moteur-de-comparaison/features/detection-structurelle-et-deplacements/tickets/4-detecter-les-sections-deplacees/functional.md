## Ticket : Détecter les sections déplacées

### 1. Objectif
Mettre en œuvre le mécanisme permettant d’identifier les sections ou blocs textuels fortement similaires entre deux documents mais positionnés différemment, et de les marquer explicitement comme "déplacés".

Ce mécanisme doit :
- Distinguer un déplacement d’une suppression + ajout,
- Alimenter correctement le calcul du sous-score structurel (SS),
- Exclure les blocs déplacés du calcul des modifications textuelles (ST),
- Fournir des données structurées exploitables par le frontend et les exports.

---

### 2. Périmètre fonctionnel

Le ticket couvre :

1. L’identification des sections correspondantes entre document A (référence) et document B.
2. La détection d’un déplacement lorsque :
   - La similarité interne de la section est ≥ 95 %,
   - La position dans l’ordre des sections diffère.
3. Le marquage explicite des sections comme "déplacées".
4. La production des données nécessaires pour :
   - Alimenter la liste `blocsDeplaces`,
   - Exclure ces sections du calcul des suppressions/ajouts dans ST,
   - Appliquer une pénalité modérée dans SS.

Ne sont pas couverts par ce ticket :
- Le calcul complet du score global,
- L’implémentation du diff textuel détaillé,
- L’affichage frontend des déplacements.

---

### 3. Règles de gestion

#### 3.1 Définition d’un bloc éligible

L’unité analysée est la section logique issue de la normalisation, comprenant :
- Un identifiant unique,
- Un titre et son niveau hiérarchique,
- Un ensemble de paragraphes et éventuellement de tableaux.

Les sections trop courtes peuvent être exclues selon les règles globales définies dans la fonctionnalité parente (éviter faux positifs).

#### 3.2 Critère de similarité

Une section est considérée comme correspondante si :
- Son contenu textuel normalisé présente une similarité interne ≥ 95 %.

La similarité interne est calculée via le mécanisme textuel existant (diff local basé sur ST).

#### 3.3 Définition d’un déplacement

Une section correspondante est marquée comme déplacée si :
- Elle est appariée entre A et B,
- `positionIndexA ≠ positionIndexB`.

Effets métier :
- La section n’est pas comptée comme supprimée ni ajoutée.
- Elle est enregistrée comme événement de type `deplacementSection`.
- Elle impacte uniquement le sous-score structurel (SS), pas le score textuel (ST).

#### 3.4 Déterminisme

Pour une même paire de documents et une même configuration :
- Les correspondances doivent être identiques,
- Les blocs marqués comme déplacés doivent être strictement les mêmes.

---

### 4. Données produites

La détection des déplacements doit enrichir le résultat de comparaison avec :

1. Une liste `blocsDeplaces` contenant :
   - sectionIdA,
   - sectionIdB,
   - positionA,
   - positionB,
   - similariteInterne.

2. Une entrée correspondante dans `differencesStructurelles` avec :
   - type = "deplacementSection",
   - sectionIdA,
   - sectionIdB,
   - positionA,
   - positionB.

Ces données seront utilisées par :
- Le calcul du sous-score SS,
- Le frontend pour visualisation,
- Le module d’export.