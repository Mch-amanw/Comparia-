## 3.4 Score de similarité – Définition formelle

### 3.4.1 Objectif
Le score de similarité mesure le degré de ressemblance entre deux documents sur une échelle de 0 à 100 %.

- 100 % = documents strictement identiques en contenu textuel et structure.
- 0 % = aucun contenu textuel commun significatif.

Le score est déterministe : pour une même paire de documents et une même configuration, le résultat doit être identique.

---

## 3.4.2 Périmètre des éléments pris en compte

Le score repose sur une analyse pondérée des composantes suivantes :

1. Contenu textuel principal (paragraphes)
2. Titres et hiérarchie documentaire
3. Tableaux (contenu textuel des cellules)
4. Structure (organisation et ordre des sections)

Non pris en compte dans le score standard (mais analysables séparément) :
- Mise en page (marges, polices, espacements)
- Images et éléments graphiques
- Métadonnées internes

Un mode avancé pourra intégrer la mise en page dans un score secondaire distinct.

---

## 3.4.3 Décomposition du score

Le score global S est calculé comme une moyenne pondérée :

S = wT * ST + wH * SH + wB * SB + wS * SS

Avec :
- ST = score de similarité textuelle (paragraphes)
- SH = score de similarité des titres
- SB = score de similarité des tableaux
- SS = score de similarité structurelle

Poids standards (configurables par tenant) :
- wT = 0,60
- wH = 0,15
- wB = 0,15
- wS = 0,10

Somme des poids = 1

---

## 3.4.4 Calcul des sous-scores

### A. Similarité textuelle (ST)

Basée sur une distance de type Myers diff ou LCS amélioré.

ST = 1 − (D / L)

Où :
- D = nombre total d’unités modifiées (ajouts + suppressions + remplacements)
- L = nombre total d’unités du document de référence
- Unité par défaut = mot
- Option avancée = caractère

Les déplacements de blocs textuels identiques ne sont pas comptés comme suppressions + ajouts si détectés comme "déplacés".

---

### B. Similarité des titres (SH)

Comparaison basée sur :
- Texte des titres
- Niveau hiérarchique (H1, H2, etc.)

SH combine :
- Similarité textuelle des titres
- Cohérence de leur position hiérarchique

Un changement de titre majeur impacte plus fortement SH qu’un simple changement de paragraphe.

---

### C. Similarité des tableaux (SB)

Calcul basé uniquement sur le texte des cellules.

SB = 1 − (Dtable / Ltable)

Les changements d’ordre de lignes détectés comme déplacements ne sont pas pénalisés comme suppressions.

La mise en forme du tableau (bordures, styles) n’impacte pas le score.

---

### D. Similarité structurelle (SS)

Mesure la conservation de l’organisation globale.

Basée sur :
- Présence ou absence de sections
- Ordre des sections
- Détection de blocs déplacés

Méthode :
- Création d’empreintes (hash) par section normalisée
- Analyse des correspondances et déplacements

Un bloc déplacé mais identique textuellement :
- N’impacte pas ST
- Impacte légèrement SS (réorganisation)

---

## 3.4.5 Gestion des déplacements

Si un bloc textuel est détecté comme identique (≥ 95 % de similarité interne) mais positionné ailleurs :
- Il est marqué comme "déplacé"
- Il n’est pas compté comme suppression + ajout
- Une pénalité structurelle modérée est appliquée via SS uniquement

---

## 3.4.6 Cas limites

1. Documents strictement identiques
→ S = 100 %

2. Documents identiques avec mise en page différente uniquement
→ S = 100 % (mode standard)

3. Documents identiques avec sections déplacées
→ S généralement entre 90 % et 98 % selon ampleur

4. Documents totalement différents
→ S proche de 0 %

5. Document B contient Document A intégralement + ajouts
→ Score basé sur document de référence
→ Les ajouts diminuent ST proportionnellement

6. OCR bruité
→ Les erreurs OCR impactent ST
→ Possibilité d’activation d’un seuil de tolérance (normalisation avancée)

---

## 3.4.7 Propriétés du score

- Déterministe
- Reproductible
- Indépendant de la mise en page par défaut
- Configurable par tenant (pondérations)
- Versionné (v1 du modèle de scoring)

Toute évolution de la formule entraîne une nouvelle version du modèle de scoring afin de garantir la traçabilité contractuelle.