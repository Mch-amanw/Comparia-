## Ticket : Détecter les sections déplacées – Spécification technique

### 1. Positionnement dans le pipeline

Ce mécanisme intervient après :
1. L’extraction et la normalisation des documents,
2. La segmentation en sections hiérarchisées,
3. Le calcul ou la disponibilité d’une mesure de similarité textuelle locale.

Il s’intègre dans l’étape d’appariement des sections entre document A (référence) et document B.

---

### 2. Données d’entrée

Représentation attendue :

DocumentNormalise {
  sections: [
    {
      id,
      titre,
      niveau,
      positionIndex,
      contenuNormalise,
      contenuHash
    }
  ]
}

Entrées principales :
- sectionsA[] (document de référence)
- sectionsB[] (document comparé)
- configuration (incluant seuilSimilariteDeplacement = 0.95 par défaut)

---

### 3. Algorithme de détection

#### 3.1 Étape 1 – Appariement initial

1. Construire un index des sectionsA par `contenuHash`.
2. Pour chaque sectionB :
   - Vérifier correspondance exacte par hash.
   - Si non trouvée, calculer similarité interne avec sectionsA candidates.

La similarité interne est calculée via un diff local (même granularité que configurée : mot par défaut).

Condition de correspondance forte :

similariteInterne >= seuilSimilariteDeplacement (0.95)

Chaque sectionA ne peut être appariée qu’à une seule sectionB (matching 1–1).

---

#### 3.2 Étape 2 – Détection du déplacement

Pour chaque paire appariée (sectionA, sectionB) :

Si sectionA.positionIndex != sectionB.positionIndex :
- Marquer comme bloc déplacé.

Structure interne générée :

blocsDeplaces.push({
  sectionIdA,
  sectionIdB,
  positionA,
  positionB,
  similariteInterne
})

Ajouter également dans differencesStructurelles :

{
  type: "deplacementSection",
  sectionIdA,
  sectionIdB,
  positionA,
  positionB
}

---

### 4. Impact sur le scoring

#### 4.1 Exclusion du calcul ST

Les sections marquées comme déplacées :
- Ne doivent pas être comptabilisées comme suppression + ajout dans D (formule ST).
- Doivent être exclues du calcul des unités modifiées globales liées à ces blocs.

Cela implique que le moteur de calcul ST doit recevoir la liste des blocs déplacés pour neutraliser ces segments.

#### 4.2 Contribution à SS

Chaque déplacement détecté contribue à `penalitesStructure` selon les règles définies dans la fonctionnalité parente.

La pondération exacte reste gérée par le module de calcul SS.

---

### 5. Contraintes techniques

- Complexité maîtrisée pour ≤ 300 pages.
- Priorité à l’utilisation de `contenuHash` pour limiter les calculs de similarité coûteux.
- Fallback sur similarité calculée uniquement si hash non identique.
- Matching déterministe (ordre stable de traitement, pas d’aléatoire).
- Aucun stockage persistant.
- Compatible traitement asynchrone.

---

### 6. Cas particuliers

- Si plusieurs sections candidates dépassent 95 %, choisir celle avec similarité maximale.
- En cas d’égalité stricte, choisir celle avec distance de position minimale pour garantir déterminisme.
- Sections très courtes : possibilité d’ignorer si taille < seuil minimal défini globalement.

Toute modification du seuil par défaut ou de la logique d’appariement impacte la version du modèle de scoring.