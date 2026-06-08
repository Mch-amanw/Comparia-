## Ticket : Développer l’algorithme de diff mot et caractère

### 1. Positionnement dans l’architecture

Ce composant est une brique interne du Moteur de comparaison.

Il est invoqué après :
1. Extraction et normalisation.
2. Segmentation en sections/paragraphe.
3. Tokenisation selon la granularité choisie.

Il ne gère ni l’OCR ni la normalisation brute.

---

### 2. Interface technique

#### 2.1 Signature conceptuelle

```
computeTextDiff(
  tokensA: string[],
  tokensB: string[],
  options: {
    granularite: "mot" | "caractere",
    deplacementRangesA?: Range[],
    deplacementRangesB?: Range[]
  }
) -> {
  operations: DiffOperation[],
  D: number,
  L: number,
  ST: number
}
```

#### 2.2 Structure DiffOperation

```
DiffOperation {
  type: "ajout" | "suppression" | "modification" | "deplacement",
  indexAStart: number,
  indexAEnd: number,
  indexBStart: number,
  indexBEnd: number
}
```

Les index sont exprimés en indices d’unités (mot ou caractère).

---

### 3. Algorithme

#### 3.1 Algorithme principal

- Implémentation basée sur Myers diff (O(ND)) ou LCS optimisé.
- Fonctionne sur tableaux d’unités (`tokensA`, `tokensB`).
- Doit produire un chemin minimal déterministe.

Contraintes :
- Aucun comportement non déterministe.
- Ordre stable des opérations.

#### 3.2 Optimisations

- Si hash global(tokensA) == hash global(tokensB) → retour immédiat sans diff.
- Segmentation préalable possible par blocs (ex : paragraphe) pour réduire complexité.
- Limitation mémoire via stockage compact des traces.

#### 3.3 Mode caractère

- Les tokens sont des caractères UTF-8.
- Attention aux caractères multi-octets.
- Comparaison stricte après normalisation.

---

### 4. Calcul de D

Après génération des opérations :

- Pour chaque opération :
  - "suppression" → D += longueur côté A
  - "ajout" → D += longueur côté B
  - "modification" → D += max(longueurA, longueurB)
  - "deplacement" → ignoré dans D

Les plages appartenant à `deplacementRanges` sont exclues du calcul.

L = tokensA.length

Cas particulier :

```
if (L == 0) {
  if (tokensB.length == 0) ST = 1
  else ST = 0
}
else {
  ST = max(0, 1 - (D / L))
}
```

---

### 5. Intégration avec détection de déplacements

- Les plages identifiées comme déplacées sont fournies en entrée.
- Les opérations recouvrant intégralement ces plages sont requalifiées en "deplacement".
- Ces opérations ne contribuent pas à D.

La cohérence avec le module de détection des blocs déplacés doit être garantie.

---

### 6. Performance

Contraintes :
- Jusqu’à 300 pages.
- Objectif global < 60 secondes.

Mesures :
- Complexité maîtrisée via segmentation.
- Possibilité d’exécution dans un worker dédié.
- Timeout configurable.

---

### 7. Déterminisme et versioning

- Aucune dépendance à des librairies non déterministes.
- Version d’algorithme alignée avec `scoringVersion` (ex: "v1").
- Toute modification future nécessite incrément de version.

---

### 8. Gestion des erreurs

Cas à gérer :
- Tableaux de tokens null ou incohérents.
- Dépassement mémoire.
- Timeout.

Retour d’erreur structuré conforme au module parent :
```
{
  code,
  message,
  etape: "diff",
  retryPossible
}
```

Aucun contenu textuel brut ne doit être loggé.