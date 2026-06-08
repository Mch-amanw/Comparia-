## Spécification technique – Comparaison textuelle fine

### 1. Position dans le pipeline
La comparaison textuelle fine intervient après :
1. Extraction des documents.
2. Normalisation (encodage, espaces, ponctuation, options de casse).
3. Segmentation en sections et paragraphes.

Elle opère sur des unités textuelles déjà normalisées.

---

### 2. Tokenisation

#### 2.1 Mode mot (défaut)
- Découpage basé sur séparation par espaces et ponctuation normalisée.
- Conservation de l’ordre séquentiel des tokens.
- Chaque token correspond à une unité de calcul.

#### 2.2 Mode caractère (optionnel)
- Chaîne complète normalisée découpée caractère par caractère.
- UTF-8 obligatoire.
- Les retours à la ligne sont normalisés.

Le mode utilisé est transmis via :
modeGranularite = "mot" | "caractere".

---

### 3. Algorithme de diff

#### 3.1 Algorithme principal
- Implémentation basée sur Myers diff ou LCS optimisé.
- Adapté à des volumes importants (jusqu’à 300 pages).
- Optimisation par segmentation logique (section/paragraphe) pour limiter la complexité globale.

#### 3.2 Optimisations
- Détection préalable de blocs strictement identiques via hash pour court-circuiter le diff.
- Traitement par blocs pour limiter l’usage mémoire.
- Possibilité de paralléliser par section indépendante.

Complexité cible : optimisée pour rester compatible avec l’objectif < 1 minute.

---

### 4. Calcul de D et L

- L = nombre total d’unités du document de référence.
- D = somme des opérations :
  - insertions,
  - suppressions,
  - remplacements.

Les blocs marqués comme déplacés par le module de détection ne sont pas inclus dans D.

ST = max(0, 1 − (D / L)).

Gestion des cas limites :
- Si L = 0 et document comparé vide → ST = 1.
- Si L = 0 et document comparé non vide → ST = 0.

---

### 5. Modèle de sortie (partie textuelle)

Dans l’objet ComparisonResult :

- differencesTextuelles: [
  {
    type: "ajout" | "suppression" | "modification" | "deplacement",
    sectionId,
    positionReference,
    positionCompare,
    contenuAvant,
    contenuApres
  }
]

- sousScores.st
- meta.granularite

Les positions sont exprimées en index d’unité (mot ou caractère) dans chaque document.

---

### 6. Déterminisme

- Aucune opération aléatoire.
- Ordre strictement déterminé par la séquence d’entrée.
- Même algorithme, même version de librairie et même configuration → résultat identique.

La version du modèle de scoring (ex : "v1") est incluse dans meta.scoringVersion.

---

### 7. Contraintes techniques

- Compatible multi-tenant (pondérations externes, pas de logique hardcodée spécifique client).
- Aucune persistance interne.
- Logs techniques sans contenu textuel brut.
- Gestion des erreurs :
  - Document vide,
  - Dépassement mémoire,
  - Timeout.

Retour d’erreur structuré conforme au module parent.

---

### 8. Dépendances

- Module de normalisation documentaire.
- Module de détection des blocs déplacés.
- Système de tâches asynchrones pour gros volumes.
- Module de configuration tenant (granularité, tolérance OCR).