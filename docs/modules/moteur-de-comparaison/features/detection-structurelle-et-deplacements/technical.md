## Spécification technique – Détection structurelle et déplacements

### 1. Positionnement dans le pipeline

Cette fonctionnalité intervient après :
1. L’extraction et la normalisation des documents,
2. La segmentation en sections hiérarchisées.

Elle alimente :
- Le calcul du sous-score SS,
- Partiellement SH (hiérarchie des titres),
- L’exclusion de certains blocs du calcul ST (cas des déplacements).

---

### 2. Représentation interne utilisée

Basée sur la structure normalisée :

DocumentNormalise = {
  sections: [
    {
      id,
      titre,
      niveau,
      positionIndex,
      contenuHash,
      paragraphes[],
      tableaux[]
    }
  ]
}

- positionIndex : position séquentielle dans le document.
- contenuHash : empreinte du contenu textuel normalisé (ex : SHA-256 tronqué).

---

### 3. Algorithme de détection des correspondances

#### 3.1 Appariement initial des sections

1. Création d’un index des sections du document de référence (A) basé sur :
   - contenuHash,
   - similarité textuelle interne.
2. Pour chaque section du document B :
   - Recherche d’une correspondance exacte par hash.
   - À défaut, calcul d’une similarité interne (basée sur ST local).

Seuil de similarité pour correspondance forte : ≥ 95 %.

---

### 4. Détection des déplacements

Une section est marquée comme déplacée si :
- Correspondance forte trouvée (≥ 95 %),
- positionIndexA ≠ positionIndexB.

Traitement :
- Ajout dans blocsDeplaces[].
- Exclusion de ces blocs du calcul D pour ST.
- Ajout d’une pénalité structurelle dans SS.

---

### 5. Détection des ajouts et suppressions

Après appariement :
- Sections de A sans correspondance → suppressionSection.
- Sections de B sans correspondance → ajoutSection.

Ces événements alimentent :
- differencesStructurelles[],
- Le calcul des penalitesStructure.

---

### 6. Analyse de l’ordre et permutation

Pour les sections appariées :
- Construction de la séquence des positions correspondantes.
- Calcul d’une mesure de désordre (ex : distance simplifiée inspirée de Kendall tau).

penalitesOrdre = nombre de permutations significatives.

SS = 1 − (penalitesStructure / totalSectionsReference)

Où penalitesStructure inclut :
- suppressions,
- ajouts,
- déplacements,
- incohérences d’ordre.

SS est borné entre 0 et 1.

---

### 7. Gestion des changements de hiérarchie

Pour chaque section appariée :
- Comparaison niveauA vs niveauB.

Si niveau différent :
- Ajout d’un événement changementNiveau.
- Impact sur SH.
- Impact secondaire possible sur SS via pénalité hiérarchique.

---

### 8. Données de sortie enrichies

Ajout dans ComparisonResult :

- differencesStructurelles: [
    {
      type,
      sectionIdA?,
      sectionIdB?,
      positionA?,
      positionB?,
      niveauAvant?,
      niveauApres?
    }
  ]
- blocsDeplaces: [
    {
      sectionIdA,
      sectionIdB,
      positionA,
      positionB,
      similariteInterne
    }
  ]

Ces données sont exploitées par :
- Le frontend (visualisation des déplacements),
- Le module de scoring,
- Le module d’export.

---

### 9. Contraintes techniques

- Complexité maîtrisée pour documents ≤ 300 pages.
- Utilisation d’empreintes pour éviter recalculs coûteux.
- Déterminisme strict : même documents + même configuration → mêmes correspondances.
- Compatible avec traitement asynchrone.
- Aucune persistance interne durable.

---

### 10. Gestion des cas limites

- Document sans titres : fallback sur segmentation par blocs textuels.
- Sections très courtes : seuil minimal de taille pour éviter faux positifs.
- OCR bruité : similarité calculée après normalisation avancée si activée.

Toute évolution des règles de pénalité impactant SS nécessite incrément de scoringVersion.