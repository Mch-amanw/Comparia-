## Spécification technique – Calcul du score de similarité

### 1. Positionnement dans le pipeline

La fonctionnalité intervient après :
1. Extraction et normalisation des documents.
2. Exécution des algorithmes de diff textuel.
3. Détection des blocs déplacés.
4. Analyse des titres, tableaux et structure.

Elle consomme les métriques intermédiaires produites par ces étapes.

---

### 2. Entrées techniques

Objet interne agrégé contenant :

- textMetrics:
  - totalUnitsReference (L)
  - totalDifferences (D)
- titleMetrics
- tableMetrics:
  - totalTableUnits (Ltable)
  - totalTableDifferences (Dtable)
- structureMetrics:
  - totalSections
  - penalitesStructure
  - blocsDeplaces
- configurationTenant:
  - wT, wH, wB, wS
  - toleranceOCR (optionnel)
  - scoringModelVersion

---

### 3. Calcul des sous-scores

#### 3.1 ST

ST = 1 − (D / L)

Contraintes :
- Si L = 0 (document vide), gestion spécifique :
  - Si les deux documents sont vides → ST = 1.
  - Sinon → ST = 0.
- ST borné entre 0 et 1.

Les différences liées aux blocs marqués comme déplacés sont exclues de D.

---

#### 3.2 SH

Calcul basé sur :
- Similarité textuelle agrégée des titres.
- Cohérence des niveaux hiérarchiques.

Normalisation finale entre 0 et 1.

---

#### 3.3 SB

SB = 1 − (Dtable / Ltable)

Contraintes :
- Si aucun tableau présent dans les deux documents → SB = 1.
- Si tableau présent uniquement dans un document → SB = 0.

Borné entre 0 et 1.

---

#### 3.4 SS

SS = 1 − (penalitesStructure / totalSections)

- Intègre pénalités liées aux absences, ajouts, permutations.
- Intègre impact modéré des blocs déplacés.
- Borné entre 0 et 1.

---

### 4. Calcul du score global

Validation préalable :
- wT + wH + wB + wS = 1 (tolérance flottante contrôlée).

Calcul :

S = wT * ST + wH * SH + wB * SB + wS * SS

- S borné entre 0 et 1.
- Conversion en pourcentage : S * 100.
- Arrondi à 2 décimales.

---

### 5. Modèle de sortie

Extension de l’objet ComparisonResult :

{
  globalScore: number,
  sousScores: {
    st: number,
    sh: number,
    sb: number,
    ss: number
  },
  scoringModelVersion: string,
  detailParSection: [
    {
      sectionId,
      scoreSection,
      stSection,
      ssSection
    }
  ],
  meta: {
    granularite,
    poidsUtilises,
    tempsCalculScoreMs
  }
}

Tous les sous-scores sont exprimés en pourcentage côté sortie API.

---

### 6. Versioning

- scoringModelVersion obligatoire (ex : "v1").
- Toute modification d’algorithme ou de pondération par défaut nécessite incrément de version.
- La version est persistée avec le résultat pour audit.

---

### 7. Performance

- Calcul du score global : complexité O(1) une fois métriques calculées.
- Le détail par section doit réutiliser les métriques existantes (pas de recalcul complet).
- Temps de calcul négligeable par rapport au diff principal.

---

### 8. Sécurité et conformité

- Aucun contenu textuel brut ne doit être inclus dans l’objet score.
- Les logs techniques incluent uniquement :
  - scoringModelVersion
  - poids utilisés
  - durée de calcul

Conforme aux exigences RGPD et aux règles du module moteur (stateless, pas de stockage persistant interne).