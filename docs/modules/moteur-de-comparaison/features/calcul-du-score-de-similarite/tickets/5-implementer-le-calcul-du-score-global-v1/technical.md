## Implémentation technique – Calcul du score global v1

### 1. Positionnement

Ce composant est intégré au sein du Moteur de comparaison, après calcul des sous-scores.

Il s’agit d’une fonction pure (stateless), déterministe, sans effet de bord.

Signature conceptuelle :

computeGlobalScore({
  st,
  sh,
  sb,
  ss,
  weights: { wT, wH, wB, wS },
  scoringModelVersion
}) -> GlobalScoreResult

---

### 2. Validation des entrées

#### 2.1 Validation des sous-scores
- Vérifier que chaque sous-score (st, sh, sb, ss) est un nombre.
- Vérifier que chaque valeur est bornée dans [0,1].
- Sinon : lever une erreur structurée de type SCORE_INVALID_SUBSCORE.

#### 2.2 Validation des poids
- Vérifier que chaque poids est un nombre ≥ 0.
- Calculer sumWeights = wT + wH + wB + wS.
- Vérifier |sumWeights − 1| ≤ epsilon (ex : 1e-6).
- Sinon : lever une erreur SCORE_INVALID_WEIGHTS.

Aucune correction automatique silencieuse n’est autorisée.

---

### 3. Calcul du score

Formule stricte :

S = wT * st + wH * sh + wB * sb + wS * ss

Étapes :
1. Calcul en précision flottante double.
2. Bornage explicite :
   - Si S < 0 → S = 0
   - Si S > 1 → S = 1
3. Conversion en pourcentage :
   globalScorePercent = S * 100
4. Arrondi à 2 décimales (arrondi standard mathématique).

Les sous-scores retournés dans l’API sont également convertis en pourcentage et arrondis à 2 décimales.

---

### 4. Structure de sortie

GlobalScoreResult (intégré dans ComparisonResult) :

{
  globalScore: number,              // pourcentage arrondi (ex: 92.37)
  sousScores: {
    st: number,                     // pourcentage arrondi
    sh: number,
    sb: number,
    ss: number
  },
  scoringModelVersion: string,      // "v1"
  meta: {
    poidsUtilises: {
      wT: number,
      wH: number,
      wB: number,
      wS: number
    },
    tempsCalculScoreMs: number
  }
}

Contraintes :
- Aucun contenu textuel n’est inclus.
- scoringModelVersion est obligatoire.
- poidsUtilises reflète exactement les poids effectivement appliqués.

---

### 5. Performance

- Complexité : O(1).
- tempsCalculScoreMs mesuré pour observabilité.
- Impact négligeable par rapport aux étapes de diff.

---

### 6. Déterminisme et versioning

- scoringModelVersion est fixé à "v1" pour cette implémentation.
- Toute évolution ultérieure devra :
  - Créer une nouvelle fonction ou branche conditionnelle versionnée (ex : v2),
  - Garantir la reproductibilité des résultats v1.

Le calcul ne dépend d’aucun état global ni de source externe dynamique.

---

### 7. Gestion des erreurs

Erreurs structurées possibles :

{
  code: "SCORE_INVALID_SUBSCORE" | "SCORE_INVALID_WEIGHTS",
  message: string,
  scoringModelVersion: "v1"
}

Aucune donnée documentaire ne doit apparaître dans les messages d’erreur.